---
title: Related Works
aliases:
  - 관련 연구
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - related-works
status: draft
bibliography: ../reference.bib
link-citations: true
---

# Related Works

## 분류 원칙

본 연구는 서로 다른 robot [[overview/glossary#Embodiment|embodiment]]가 순차적으로 주어지는 [[overview/glossary#Cross-Embodiment Continual Reinforcement Learning|cross-embodiment continual reinforcement learning]]을 다룬다. 관련 연구는 주된 연구 질문을 기준으로 다음 세 축에 배치한다.

1. **[[overview/glossary#Continual Learning|Continual Reinforcement Learning]]:** [[overview/glossary#Sequential Training|sequential training]]에서 지식을 보존하고 재사용하는 방법
2. **[[overview/glossary#Morphology-Aware Policy Learning|Morphology-Aware Policy Learning]]:** [[overview/glossary#Morphology|morphology]]를 policy의 입력이나 architecture에 반영하는 방법
3. **[[overview/glossary#Cross-Embodiment Learning|Cross-Embodiment Learning]]:** 여러 embodiment 사이에서 experience, representation, model 또는 policy를 공유하거나 이전하는 방법

한 논문이 여러 축에 걸칠 수 있으므로 주된 기여를 기준으로 한 곳에만 배치한다. 직접적인 비교 대상은 위 세 축에 두고, 문제 설정이나 평가 목적이 다른 논문은 마지막의 `Adjacent Work`에 분리한다.

### Continual RL 내부 분류에 대한 권장사항

`replay`, `regularization`, `architecture/parameter isolation`의 세 분류는 [[overview/glossary#Catastrophic Forgetting|catastrophic forgetting]]을 완화하는 **주된 메커니즘**을 설명하기에 적합하다. 다만 이는 유일하거나 상호 배타적인 taxonomy가 아니다. Khetarpal et al.은 explicit knowledge retention, leveraging shared structure, learning to learn으로 구분하고 [@khetarpal2022towards], Pan et al.은 저장·이전되는 지식을 policy, experience, dynamics, reward로 구분한다 [@pan2025survey].

따라서 본 문서에서는 다음 원칙을 사용한다.

- RL 문헌에서 더 일반적인 명칭인 **Replay-Based Approaches**를 사용한다.
- **Regularization-Based Approaches**에는 parameter 또는 policy 변화를 제한하는 방법을 둔다.
- `structure-based`는 robot의 물리적 structure와 혼동되므로 **Architecture/Parameter-Isolation-Based Approaches**로 바꾼다.
- 복합 방법은 주된 메커니즘에 배치하고 hybrid임을 명시한다.
- benchmark와 evaluation 연구는 학습 방법이 아니므로 별도 subsection에 둔다.

## 1. Continual Reinforcement Learning

### 1.1 Replay-Based Approaches

**Experience Replay for Continual Learning** (Rolnick et al., NeurIPS 2019) [@rolnick2019clear]

- **문제:** task identity 없이 변화하는 RL stream에서 과거 policy가 소실되는 문제를 다룬다.
- **해결:** CLEAR는 새로운 experience의 on-policy 학습과 과거 experience의 off-policy replay 및 behavioral cloning을 결합한다.
- **차이:** 동일한 agent interface를 전제로 하며 embodiment별 observation/action space와 morphology 변화는 다루지 않는다.

**Replay-enhanced Continual Reinforcement Learning** (Zhang et al., TMLR 2023) [@zhang2023recall]

- **문제:** 단순 replay에서 task별 reward scale과 offline distribution shift가 새로운 task 학습과 과거 성능 보존을 방해하는 문제를 다룬다.
- **해결:** RECALL은 target normalization과 과거 task의 policy distillation을 이용해 replay의 stability와 plasticity를 개선한다.
- **차이:** Continual World의 고정된 robot interface를 사용하며 embodiment 구조를 replay 선택이나 지식 공유에 활용하지 않는다.

### 1.2 Regularization-Based Approaches

**Policy Consolidation for Continual Reinforcement Learning** (Kaplanis, Shanahan, and Clopath, ICML 2019) [@kaplanis2019policy]

- **문제:** 명시적인 task boundary 없이 여러 시간 규모로 변하는 experience distribution에서 policy가 과거 행동을 잊는 문제를 다룬다.
- **해결:** 현재 policy와 여러 시간 규모의 hidden policy 사이를 지속적으로 regularize하여 과거 policy의 급격한 변화를 제한한다.
- **차이:** 고정된 observation/action interface에서 policy history를 보존하며 embodiment 사이의 구조적 대응은 사용하지 않는다.

**Progress & Compress: A scalable framework for continual learning** (Schwarz et al., ICML 2018) [@schwarz2018progress]

- **문제:** parameter 수를 계속 늘리지 않으면서 새 task를 학습하고 과거 능력을 보존하는 문제를 다룬다.
- **해결:** active column에서 새 task를 학습한 뒤 distillation과 EWC를 이용해 knowledge base로 통합한다.
- **차이:** regularization과 architecture를 결합한 hybrid이며, 서로 다른 robot interface나 morphology 기반 선택은 고려하지 않는다.

### 1.3 Architecture/Parameter-Isolation-Based Approaches

**Self-Composing Policies for Scalable Continual Reinforcement Learning** (Malagon, Ceberio, and Lozano, ICML 2024) [@malagon2024self]

- **문제:** task 수가 늘어날 때 catastrophic forgetting을 막으면서 이전 policy를 새 task에 재사용하는 문제를 다룬다.
- **해결:** task별 module을 확장하고 이전 policy들을 선택적으로 조합하는 growable architecture를 제안한다.
- **차이:** task마다 capacity가 증가하며, module 선택에 robot morphology나 kinematic correspondence를 사용하지 않는다.

**P2DT: Mitigating Forgetting in Task-Incremental Learning with Progressive Prompt Decision Transformer** (Wang et al., ICASSP 2024) [@wang2024p2dt]

- **문제:** offline trajectory를 task 순서대로 학습하는 Decision Transformer의 catastrophic forgetting을 다룬다.
- **해결:** task가 추가될 때 progressive prompt를 확장하여 task-specific context를 분리한다.
- **차이:** prompt 기반 parameter isolation을 사용하지만 서로 다른 embodiment의 joint/link 관계를 transfer signal로 사용하지 않는다.

**Tackling Continual Offline RL through Selective Weights Activation on Aligned Spaces** (Hu et al., NeurIPS 2025) [@hu2025vqcd]

- **문제:** observation/action dimension이 서로 다른 continual offline RL task를 하나의 model에서 처리하는 문제를 다룬다.
- **해결:** VQ-CD는 heterogeneous space를 vector-quantized latent space에 정렬하고 sparse task mask로 선택된 weight를 활성화한다.
- **차이:** 본 연구와 가장 가까운 heterogeneous-space 방법이지만, 정렬 기준이 명시적인 morphology나 kinematic correspondence가 아니다.

### 1.4 Benchmarks and Evaluation

**Continual World: A Robotic Benchmark For Continual Reinforcement Learning** (Wołczyk et al., NeurIPS 2021) [@wolczyk2021continualworld]

- **문제:** continual RL 평가가 과거 성능의 손실에 치우치고 새 task 학습에 대한 transfer를 충분히 측정하지 않는 문제를 다룬다.
- **해결:** Meta-World 기반 task sequence와 [[overview/glossary#Forward Transfer|forward transfer]] 및 catastrophic forgetting 평가 기준을 제공한다.
- **차이:** 하나의 고정된 robot embodiment에서 task가 바뀌며, 본 연구는 locomotion 목적을 가능한 한 유지한 채 embodiment가 바뀐다.

## 2. Morphology-Aware Policy Learning

이 절은 morphology를 어떻게 표현하고 variable observation/action interface를 policy architecture에 어떻게 수용하는지에 초점을 둔다. 세부 갈래는 morphology 정보를 policy에 제공하는 방식에 따라 나눈다.

### 2.1 Explicit Morphology Conditioning

이 갈래는 morphology graph, descriptor 또는 morphology-conditioned parameter처럼 구조 정보를 policy에 명시적으로 제공한다.

**One Policy to Control Them All: Shared Modular Policies for Agent-Agnostic Control** (Huang, Mordatch, and Pathak, ICML 2020) [@huang2020shared]

- **문제:** state/action dimension과 skeletal structure가 다른 agent들을 하나의 policy로 제어하는 문제를 다룬다.
- **해결:** SMP는 actuator별 shared module과 morphology graph의 message passing으로 variable action space를 처리한다.
- **차이:** 여러 embodiment를 동시에 사용하는 joint training이며 sequential embodiment stream과 catastrophic forgetting을 평가하지 않는다.

**MetaMorph: Learning Universal Controllers with Transformers** (Gupta et al., ICLR 2022) [@gupta2022metamorph]

- **문제:** 다양한 modular robot을 하나의 universal controller로 학습하고 unseen embodiment에 일반화하는 문제를 다룬다.
- **해결:** morphology descriptor와 proprioceptive state를 joint-level token으로 표현하여 Transformer policy의 pre-training을 수행한다.
- **차이:** 모든 training embodiment에 함께 접근하며 새 embodiment 학습 이후 과거 embodiment의 성능 변화를 측정하지 않는다.

**Universal Morphology Control via Contextual Modulation** (Xiong, Beck, and Whiteson, ICML 2023) [@xiong2023modumorph]

- **문제:** hard parameter sharing만으로는 morphology별로 달라지는 적절한 control strategy를 표현하기 어려운 문제를 다룬다.
- **해결:** ModuMorph는 morphology-conditioned hypernetwork와 attention modulation으로 controller를 조절한다.
- **차이:** multi-task joint training과 unseen-morphology generalization이 중심이며 sequential learning과 과거 성능 보존은 다루지 않는다.

### 2.2 Implicit Morphology Inference

이 갈래는 hand-designed morphology descriptor를 직접 제공하는 대신 observation과 action의 관계에서 embodiment 구조를 학습한다.

**AnyMorph: Learning Transferable Polices By Inferring Agent Morphology** (Trabucco, Phielipp, and Berseth, ICML 2022) [@trabucco2022anymorph]

- **문제:** hand-designed morphology graph나 sensor-to-limb alignment 없이 새로운 embodiment로 policy를 이전하는 문제를 다룬다.
- **해결:** sensor token과 action token 사이의 관계에서 morphology representation을 암묵적으로 학습한다.
- **차이:** cross-morphology generalization이 목적이며, 본 연구처럼 명시적인 morphology를 continual transfer prior로 사용하는 설정과 다르다.

### 2.3 Structure-Compatible Universal Interfaces

이 갈래는 특정 morphology descriptor에 의존하기보다 body component별 가변 길이 입력과 출력을 구성하여 서로 다른 robot interface를 수용한다.

**One Policy to Run Them All: an End-to-end Learning Approach to Multi-Embodiment Locomotion** (Bohlinger et al., CoRL 2024) [@bohlinger2025urma]

- **문제:** quadruped, biped, hexapod 등 서로 다른 legged robot을 하나의 locomotion policy로 제어하는 문제를 다룬다.
- **해결:** URMA는 joint, foot, global observation을 가변 길이로 처리하고 unseen simulated/real robot으로 transfer한다.
- **차이:** multi-embodiment joint training과 배포 시 transfer가 중심이며 sequential update 이후 과거 embodiment를 재평가하지 않는다.

## 3. Cross-Embodiment Learning

[[overview/glossary#Cross-Embodiment Learning|Cross-Embodiment Learning]]은 여러 embodiment 사이에서 지식을 공유하거나 이전하는 넓은 연구 범위다. 본 문서에서는 논문의 주된 학습 설정에 따라 reinforcement learning, policy transfer/adaptation, embodiment–controller co-design으로 나눈다. 한 방법이 여러 갈래에 걸칠 수 있으므로 실제 배치는 논문의 핵심 연구 질문을 따른다.

### 3.1 Cross-Embodiment Reinforcement Learning

이 갈래는 여러 embodiment에서 RL objective를 통해 공유 representation, value 또는 policy를 학습한다. 현재 문헌은 다시 unsupervised pre-training과 offline/pooled-data RL로 나눌 수 있다.

#### 3.1.1 Unsupervised Pre-training

**PEAC: Unsupervised Pre-training for Cross-Embodiment Reinforcement Learning** (Ying et al., NeurIPS 2024) [@ying2024peac]

- **문제:** reward-free environment에서 여러 embodiment와 downstream task에 재사용할 수 있는 지식을 학습하는 문제를 다룬다.
- **해결:** PEAC는 embodiment-aware하고 task-agnostic한 intrinsic objective를 이용해 cross-embodiment pre-training을 수행한다.
- **차이:** pre-training 후 downstream adaptation을 평가하며 embodiment가 계속 추가되는 stream이나 과거 성능 보존은 다루지 않는다.

#### 3.1.2 Offline and Pooled-Data Reinforcement Learning

**Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets** (Abe et al., ICLR 2026) [@abe2026cross]

- **문제:** suboptimal trajectory가 포함된 heterogeneous robot dataset에서 공유 가능한 control prior를 학습하는 문제를 다룬다.
- **해결:** 16개 legged robot dataset을 구축하고 offline RL과 morphology-distance 기반 grouping으로 inter-robot gradient conflict를 줄인다.
- **차이:** 모든 robot dataset을 pooled하여 사용하므로 순차적 데이터 도착, 과거 dataset 접근 제한, catastrophic forgetting을 다루지 않는다.

### 3.2 Policy Transfer and Adaptation

이 갈래는 여러 source embodiment에서 학습한 policy를 새로운 target embodiment에 얼마나 효율적으로 이전할 수 있는지를 다룬다. 기반 policy가 RL로 학습되었더라도 핵심 질문이 새로운 embodiment에 대한 adaptation이면 이 갈래에 배치한다.

**Efficient Morphology-Aware Policy Transfer to New Embodiments** (Przystupa et al., RLC 2025) [@przystupa2025efficient]

- **문제:** pre-trained morphology-aware policy를 새로운 target embodiment에 적은 parameter와 interaction으로 이전하는 문제를 다룬다.
- **해결:** adapter, prefix tuning, 일부 layer tuning 등 parameter-efficient fine-tuning 방법을 비교한다.
- **차이:** `pre-training → single target` 설정이며 여러 target이 순차적으로 추가될 때의 [[overview/glossary#Backward Transfer|backward transfer]]를 평가하지 않는다.

### 3.3 Embodiment–Controller Co-Design (Adjacent)

이 갈래는 morphology와 controller를 함께 탐색하거나 설계하면서 embodiment 사이의 지식 이전을 활용한다. 본 연구의 직접적인 비교 대상은 아니지만, 구조 변화에 따라 어떤 parameter를 공유하고 분리할지 보여준다는 점에서 연관된다.

**TE-RoboNet: Transfer Enhanced RoboNet for Sample-Efficient Generation of Robot Co-Designs** (Nagiredla et al., EWRL 2025) [@nagiredla2025terobonet]

- **문제:** robot co-design 과정에서 morphology가 변할 때마다 controller를 처음부터 학습해야 하는 sample cost를 다룬다.
- **해결:** shared core와 morphology-specific adapter를 이용해 서로 다른 DoF 사이에서 controller 지식을 이전한다.
- **차이:** 주된 목적이 robot co-design이며, workshop 논문으로서 표준 continual RL sequence와 과거 embodiment 성능 평가를 제공하지 않는다.

## 본 연구의 위치

기존 연구는 replay·regularization·parameter isolation, morphology-aware universal policy, cross-embodiment transfer를 각각 다룬다. 그러나 다음 조건의 교집합은 충분히 검증되지 않았다.

> 서로 다른 embodiment가 순차적으로 등장하고 과거 dataset에 항상 접근할 수 없을 때, morphology의 구조적 정보를 이용해 새 embodiment의 학습을 가속하면서 과거 embodiment의 제어 능력을 유지할 수 있는가?

따라서 본 연구는 다음을 함께 평가해야 한다.

1. 이전 embodiment의 지식이 이후 embodiment 학습에 미치는 forward transfer
2. 새 embodiment 학습이 이전 embodiment 성능에 미치는 backward transfer와 catastrophic forgetting
3. generic latent alignment 또는 task ID 대비 morphology 정보의 추가 가치
4. replay memory, task-specific parameter, adapter 등 방법별 resource 증가량

연구 동기는 [[overview/motivation|Motivation]], 데이터 구성은 [[experiments/dataset|Dataset]], 초기 검증 계획은 [[experiments/experiments_1/Round1_experiments|Round 1 Experiments]]에 정리한다.

## 조사 메모

- [[related_works_research/continual_rl_taxonomy|Continual RL 분류 조사 메모]]
- [[related_works_research/morphology_cross_embodiment_taxonomy|Morphology-Aware / Cross-Embodiment 분류 조사 메모]]
