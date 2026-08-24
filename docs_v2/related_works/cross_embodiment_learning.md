---
title: Cross-Embodiment Learning
aliases:
  - Cross-Embodiment Related Works
tags:
  - cross-embodiment
  - related-works
status: draft
bibliography: reference.bib
link-citations: true
---

# Cross-Embodiment Learning

> [!abstract] 문서 목적
> 서로 다른 robot embodiment 사이에서 representation, reward, model 또는 policy를 공유하고 이전하는 연구를 학습 설정에 따라 정리한다.

> [!info] 관련 문서
> [[related_works/related_works_overview|Related Works Overview]] · [[related_works/continual_reinforcement_learning|Continual Reinforcement Learning]] · [[related_works/morphology_aware_policy_learning|Morphology-Aware Policy Learning]]

## 연구 범위

[[wiki#Cross-Embodiment|cross-embodiment learning]]은 서로 다른 [[wiki#Embodiment|embodiment]] 사이에서 지식을 공유하거나 이전하는 넓은 연구 범위다. Observation/action dimension, sensor와 actuator 구성이 달라도 공통된 representation이나 control knowledge를 학습할 수 있는지를 다룬다.

이 범위는 특정 학습 protocol을 전제하지 않는다. 여러 robot의 data를 동시에 사용하는 joint training, source에서 target으로 한 번 이전하는 adaptation, 여러 robot dataset을 합치는 pooled offline learning이 모두 포함될 수 있다. 따라서 cross-embodiment learning이 곧 continual learning을 의미하지는 않는다.

## Representation and Reward Transfer

### XIRL

- **Problem:** Shape, action과 end-effector dynamics가 다른 embodiment의 video demonstration 사이에서도 같은 task의 진행 상태를 인식하고 reward를 만들어야 하는 문제를 다룬다.
- **Method:** 여러 expert의 offline video에 temporal cycle-consistency를 적용하여 task progression을 담는 visual embedding을 self-supervised하게 학습하고, 현재 state와 goal state의 embedding distance를 reward로 사용한다 [@zakka2022xirl].
- **Experiments:** X-MAGICAL의 seen·unseen embodiment와 real-world human demonstration에서 simulated robot으로의 transfer에서 reward generalization과 downstream policy learning을 평가했다.
- **Results:** X-MAGICAL과 real-world human demonstration에서 simulated robot으로 transfer하는 실험에서 seen·unseen embodiment 모두에 reward가 generalization했으며, 기존 방법보다 적은 interaction으로 target policy를 학습했다.

XIRL은 embodiment가 달라도 task knowledge를 이전할 수 있음을 보여주지만, policy가 embodiment를 순차적으로 학습하거나 이전 embodiment의 control performance를 유지해야 하는 setting은 아니다.

### PEAC

- **Problem:** 기존 cross-embodiment RL의 지식이 특정 task에 결합되어 embodiment 고유의 특성과 task-agnostic skill을 충분히 학습하지 못하는 문제를 다룬다.
- **Method:** Reward-free environment에서 embodiment-aware하고 task-agnostic한 지식을 학습하는 CEURL을 정의하고, cross-embodiment exploration과 skill discovery를 위한 intrinsic reward 기반 PEAC을 제안한다 [@ying2024peac].
- **Experiments:** DeepMind Control Suite, Robosuite와 real-world legged locomotion에서 pre-training 후 downstream adaptation과 cross-embodiment generalization을 평가했다.
- **Results:** DMC, Robosuite와 real-world legged locomotion에서 downstream adaptation 성능과 cross-embodiment generalization이 기존 unsupervised RL 방법보다 개선되었다.

PEAC의 중심 질문은 reusable representation의 pre-training과 downstream transfer다. 새 embodiment를 학습한 뒤 기존 embodiment의 성능이 어떻게 변하는지는 평가 대상이 아니다.

## Policy Transfer and Adaptation

### Efficient Morphology-Aware Policy Transfer

- **Problem:** Morphology-aware pre-trained policy의 unseen embodiment zero-shot 성능이 end-to-end fine-tuning보다 낮지만, 전체 model과 새 data를 이용한 adaptation은 비용이 큰 문제를 다룬다.
- **Method:** Target embodiment에 대해 parameter subset tuning, learnable adapter와 prefix tuning을 비교하여 적은 parameter만 갱신하는 parameter-efficient fine-tuning을 적용한다 [@przystupa2025efficient].
- **Experiments:** Morphology-aware base policy를 여러 unseen target embodiment에 online adaptation하며 full fine-tuning, scratch training과 parameter-efficient tuning을 비교했다.
- **Results:** 새로운 embodiment의 online adaptation에서 scratch training보다 필요한 sample을 줄였고, model parameter의 일부만 조정해도 base policy의 zero-shot 성능을 개선했다.

이 연구는 `pre-training → single target adaptation` 구조다. Target adaptation의 효율성은 평가하지만 이후에도 source embodiment의 성능을 유지해야 하는 continual objective는 두지 않는다.

## Offline and Pooled-Data Learning

### Cross-Embodiment Offline Reinforcement Learning

- **Problem:** 여러 robot의 heterogeneous offline dataset을 합치면 morphology와 data quality 차이로 actor gradient가 충돌하여 shared policy 학습이 저해되는 문제를 다룬다.
- **Method:** 16개 legged robot에 IQL 기반 pooled offline learning을 적용하고, morphological similarity로 robot을 묶은 뒤 group gradient로 actor update의 충돌을 줄인다 [@abe2026cross].
- **Experiments:** 16개 legged robot의 MuJoCo forward-locomotion replay dataset에서 pooled IQL, behavior cloning과 여러 gradient-conflict resolution method를 비교했다.
- **Results:** Suboptimal trajectory가 섞인 locomotion dataset에서 pure behavior cloning보다 높은 pre-training 성능을 보였으며, morphology grouping은 기존 gradient-conflict resolution보다 inter-robot conflict와 pooled policy 성능을 더 효과적으로 개선했다.

이 연구는 morphology와 IQL gradient의 관계를 분석한다는 점에서 구조 정보의 학습상 가치를 보여준다. 다만 모든 robot dataset에 동시에 접근하는 pooled training이므로, embodiment별 dataset이 순차적으로 주어지는 상황이나 과거 성능의 forgetting은 다루지 않는다.

## Adjacent Work: Embodiment–Controller Co-Design

### TE-RoboNet

- **Problem:** Robot morphology와 controller를 함께 탐색하는 co-design의 공간이 크고, design마다 controller를 독립적으로 학습하면 sample efficiency와 controller quality가 낮아지는 문제를 다룬다.
- **Method:** 모든 morphology가 공유하는 core network와 morphology-specific adapter를 결합하여 DoF와 topology가 다른 robot 사이에서 control knowledge를 transfer한다 [@nagiredla2025terobonet].
- **Experiments:** 네 가지 robot co-design environment에서 동일한 memory·computation budget의 co-design baseline과 비교하고 design 사이의 controller transfer를 평가했다.
- **Results:** 여러 robot co-design environment에서 동일한 memory와 computation budget을 사용한 co-design baseline보다 더 높은 design-control 성능을 보였다.

Embodiment 사이의 parameter 공유 방식을 보여준다는 점에서는 관련되지만, 주된 평가 대상은 continual retention이 아니라 sample-efficient robot design generation이다.

## 학습 설정 비교

| 연구 방향 | Data 접근 방식 | 주로 측정하는 것 | 이전 embodiment 재평가 |
| --- | --- | --- | ---: |
| XIRL / PEAC | Multi-embodiment pre-training | Representation transfer와 downstream adaptation | 요구하지 않음 |
| Morphology-aware policy transfer | Source pre-training 후 single target adaptation | Target adaptation 효율 | 요구하지 않음 |
| Cross-embodiment offline RL | 모든 robot dataset의 pooled training | Shared policy 성능과 gradient conflict | Continual 관점에서는 측정하지 않음 |
| Cross-embodiment continual RL | Embodiment별 dataset의 sequential access | 새로운 embodiment 학습과 이전 성능 유지 | 필요 |

## 정리

기존 cross-embodiment learning은 서로 다른 robot 사이에서도 representation, reward와 policy를 공유하거나 이전할 수 있음을 보여준다. 그러나 대부분은 joint pre-training, pooled training 또는 하나의 target adaptation을 사용한다.

따라서 남는 질문은 cross-embodiment transfer의 가능성 자체가 아니라, **embodiment가 순차적으로 주어질 때 이전 control knowledge를 유지하면서 새로운 embodiment에 재사용할 수 있는가**이다. 이 지점에서 cross-embodiment learning은 [[related_works/continual_reinforcement_learning|Continual Reinforcement Learning]]과 연결된다.
