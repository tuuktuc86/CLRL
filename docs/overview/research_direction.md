---
title: Research Direction
aliases:
  - 연구 방향
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - research-direction
status: draft
---

# Research Direction

이 문서는 프로젝트의 연구 문제, 가설, 범위와 실험 논리를 연결하는 최상위 방향 문서다. 구체적인 dataset contract는 [Dataset](../experiments/dataset.md), 전체 실험 흐름은 [Experiment Guideline](../experiments/guideline.md), Round별 실행 설정은 각 Round 문서에서 관리한다.

## 한 문장 방향

> 서로 다른 robot embodiment가 순차적으로 도착하는 continual offline reinforcement learning에서, morphology를 구성하는 kinematic topology를 structural prior로 사용하면 새로운 embodiment의 control capability를 획득하면서 이전 embodiment의 capability를 더 잘 유지할 수 있는지 검증한다.

## 해결하려는 문제

기존 continual reinforcement learning은 주로 동일한 observation/action interface에서 task가 순차적으로 변하는 상황을 다룬다. 반면 실제 robot은 joint 수, action dimension, kinematic structure와 physical property가 서로 다르다. 새로운 embodiment마다 policy를 처음부터 학습하면 과거 control knowledge를 재사용하지 못하고, 하나의 model을 계속 fine-tuning하면 이전 embodiment의 capability가 손상될 수 있다.

이 프로젝트는 동일한 forward-locomotion objective를 수행하지만 embodiment가 다른 robot dataset이 순차적으로 주어지는 [Embodiment-Incremental Continual Reinforcement Learning](./glossary.md#embodiment-incremental-continual-reinforcement-learning) 문제를 다룬다. 핵심은 heterogeneous input/output dimension을 수용하는 것만이 아니라, embodiment 사이에서 무엇을 재사용하고 무엇을 보존할지 학습하는 것이다.

## 선행 연구 이후에 남는 연구 공백

선행 연구는 이미 다음 가능성을 보여주었다.

- Cross-embodiment learning과 morphology-aware policy는 여러 robot 사이에서 representation이나 policy를 공유할 수 있다.
- Continual RL의 replay, regularization과 parameter isolation은 catastrophic forgetting을 완화할 수 있다.
- Heterogeneous continual RL은 서로 다른 state/action dimension을 alignment하거나 shared latent space로 처리할 수 있다.

따라서 이 연구는 cross-embodiment continual RL을 처음 가능하게 만든다고 주장하지 않는다. 남는 질문은 **robot morphology의 구조적 하위 요소인 topology를 continual learning 과정의 inductive bias로 명시적으로 사용했을 때 stability–plasticity trade-off가 실제로 개선되는가**이다. 세부 문헌 근거와 주장 경계는 [Related Works](./related_works.md)와 [Research Positioning Memo](../related_works_research/research_positioning_morphology_clrl.md)에 정리한다.

## Research Question

> **Can structural knowledge of robot embodiments enable continual reinforcement learning to acquire control capabilities for new embodiments more efficiently while retaining previously learned capabilities?**

이 질문은 다음 두 평가 축으로 분리한다.

1. **Acquisition and forward transfer:** 과거 embodiment에서 학습한 knowledge가 target embodiment의 학습에 어떤 영향을 주는가?
2. **Retention and forgetting:** Target embodiment를 학습한 뒤 source embodiment의 control capability가 얼마나 유지되는가?

두 축을 하나의 임의 가중합으로 축약하지 않는다. 새 embodiment를 거의 학습하지 않아 retention만 높은 방법과, target을 잘 학습하지만 source를 잊는 방법을 구분해 해석한다.

## 중심 가설

현재 가설은 다음과 같다.

> Robot topology는 parameter 또는 representation 중 서로 관련된 부분을 선택적으로 재사용하고 보호할 수 있는 구조적 단서를 제공하며, topology를 사용하지 않는 continual learning baseline보다 acquisition–retention 균형을 개선할 수 있다.

이는 아직 검증된 결론이 아니다. Morphology가 비슷한 pair에서만 도움이 되거나, explicit topology가 기존 component-wise representation보다 추가 이점을 주지 못할 가능성도 실험 결과로 받아들인다.

## Method 방향

현재 baseline backbone은 variable joint/foot dimension을 처리하기 위해 URMA-style shared component encoder와 universal joint decoder를 사용한다. 이 backbone은 observation에 포함된 joint/foot descriptor를 사용하지만 explicit graph adjacency와 parent–child message passing은 사용하지 않는다.

향후 제안 method는 동일한 heterogeneous I/O 처리 능력을 유지하면서, torso–joint–foot graph 또는 kinematic connectivity가 continual learning 과정에 실제로 영향을 주도록 설계한다. Topology가 기여할 수 있는 후보 역할은 다음과 같다.

- 연결된 component 사이의 representation 학습
- Source와 target 사이에서 재사용할 parameter 또는 substructure 선택
- 이전 embodiment에 중요한 knowledge의 선택적 consolidation
- 새로운 embodiment를 위한 update가 기존 knowledge에 미치는 interference 조절

정확한 mechanism은 아직 확정하지 않는다. 단순히 graph encoder를 추가해 model capacity를 늘리는 것만으로는 연구 가설을 검증할 수 없으므로, topology 제거·교란 또는 capacity-matched comparison으로 구조 정보의 효과를 분리해야 한다.

## 실험 논리

### Round 1: 비교 범위와 protocol 확정

Round 1은 제안 method의 우수성을 주장하는 단계가 아니다. [Cross-Embodiment Offline RL Dataset](../experiments/dataset.md)의 `all_robots_replay_forward_1m`을 사용하여 다음 pair를 독립적으로 평가한다.

```text
unitree_a1 -> unitree_go1
unitree_a1 -> unitree_h1
unitree_a1 -> hexapod
```

비교군은 다음 두 층으로 나눈다.

- **Backbone-matched:** Sequential IQL with URMA-style I/O, URMA-I/O IQL + actor-only EWC
- **Cross-architecture reference:** L2M (A1-pretrained adaptation), VQ-CD

Target-only IQL은 각 robot의 독립 학습 curve와 IQL forward-transfer 기준을 제공한다. Round 1에서는 source와 target 성능을 모든 evaluation checkpoint에서 함께 측정하여, 어떤 pair와 baseline이 이후 topology-aware method를 평가하기에 진단적인지 확인한다. 구체적인 budget과 구현 contract는 [Round 1 Experiments Guideline](../experiments/experiments_1/experiments1_guideline.md)을 따른다.

### 이후 단계: Topology-aware method 검증

제안 method가 준비되면 Round 1에서 확정한 dataset, pair, evaluation과 resource-reporting contract를 유지한다. 이후 실험은 최소한 다음 질문에 답해야 한다.

1. Sequential IQL보다 target acquisition 또는 forward transfer를 유지하거나 개선하는가?
2. EWC와 다른 retention baseline보다 source forgetting을 줄이는가?
3. Parameter 수, training-sample exposure와 계산량 차이를 고려해도 이득이 남는가?
4. Explicit topology를 제거하거나 교란했을 때 이득이 감소하는가?
5. 가까운 morphology pair와 먼 pair에서 관찰되는 효과가 어떻게 다른가?

Pairwise 결과가 진단적일 때만 더 긴 embodiment sequence, 순서 민감도 또는 replay 허용 setting으로 범위를 확장한다.

## 측정과 해석 원칙

- Robot별 raw episodic return과 source/target learning curve가 주 결과다.
- Target-only reference와 동일한 architecture·dataset·budget을 비교할 수 있을 때 forward transfer를 계산한다.
- Single-task-relative normalized score는 robot 사이의 scale을 비교하는 보조 자료다. IQL reference 성능에 종속되므로 raw result를 대체하지 않는다.
- Canonical morphology distance는 pair의 구조적 관계를 설명하는 보조 자료다. Round 1의 세 pair만으로 일반적인 correlation을 주장하지 않는다.
- Dataset quality, termination rate와 behavior distribution 차이를 morphology 효과와 분리해 기록한다.
- Training-sample exposure, parameter 수, auxiliary memory와 wall-clock time을 서로 다른 비용으로 보고한다.

## 연구가 성립하기 위한 최소 증거

제안 method의 장점을 주장하려면 다음 조건을 함께 만족해야 한다.

1. Target acquisition을 심각하게 희생하지 않으면서 source retention을 개선한다.
2. Backbone-matched baseline과 비교해 topology-aware mechanism의 이점이 나타난다.
3. Cross-architecture reference와 비교할 때 architecture, pretraining과 resource 차이를 공개한다.
4. Topology ablation 이후 성능 변화가 나타나 구조 정보가 실제 원인이라는 근거를 제공한다.
5. 적어도 하나의 가까운 pair와 하나의 구조적으로 먼 pair에서 결과를 해석할 수 있다.

반대로 성능 차이가 model capacity, 추가 target exposure, dataset quality 또는 tuning budget으로 설명되면 topology의 이점으로 주장하지 않는다. Explicit topology가 이점을 주지 않는 결과도 연구 가설에 대한 유효한 검증 결과로 기록한다.

## 현재 범위에서 주장하지 않는 내용

- 기존 cross-embodiment learning 또는 heterogeneous continual RL이 문제를 전혀 풀지 못했다는 주장
- Morphology나 topology를 입력하면 자동으로 continual learning이 해결된다는 주장
- 모든 robot pair와 학습 순서에서 동일한 효과가 나타난다는 주장
- Round 1 baseline 결과만으로 제안 method가 우수하다는 주장
- Pairwise offline 결과만으로 online continual robot learning 전체에 일반화된다는 주장

## 문서 체계

| 문서 | 역할 |
| --- | --- |
| [Research Direction](./research_direction.md) | 연구 목적, 가설, 범위와 증명 논리의 canonical source |
| [Motivation](./motivation.md) | 연구 필요성을 독자에게 설명하는 서사 |
| [Glossary](./glossary.md) | 프로젝트 canonical term과 의미 경계 |
| [Related Works](./related_works.md) | 출판용 선행 연구 분류와 본 연구의 위치 |
| [Dataset](../experiments/dataset.md) | Dataset의 factual contract와 preprocessing 정보 |
| [Experiment Guideline](../experiments/guideline.md) | 연구 전체의 실험 방향과 비교 원칙 |
| [Round 1 Experiments Guideline](../experiments/experiments_1/experiments1_guideline.md) | Round 1 구현·실행이 가능한 상세 명세 |
| [`related_works_research/`](../related_works_research/) | 조사 근거, 구현 자료와 포지셔닝 메모 |

방향성 결정이 바뀌면 이 문서를 먼저 갱신하고, Motivation·Related Works·Experiment Guideline 중 영향을 받는 문서를 함께 정렬한다. Round별 수치 설정만 바뀌는 경우에는 이 문서를 수정하지 않는다.

## Open Questions

다음 항목은 아직 연구 결정으로 고정하지 않는다.

1. Topology가 encoder, parameter selection, consolidation 또는 이들의 조합 중 어디에 직접 작용할 것인가?
2. Kinematic graph의 node/edge feature를 어떤 수준까지 사용할 것인가?
3. Canonical morphology distance가 실제 transfer 또는 interference를 예측하는가?
4. Pairwise setting 이후 어떤 길이와 순서의 continual sequence를 사용할 것인가?
5. Replay를 허용하는 별도 setting을 언제 비교할 것인가?
