---
title: Related Works Overview
aliases:
  - Related Works
  - 관련 연구 개요
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - related-works
status: draft
bibliography: reference.bib
link-citations: true
---

# Related Works Overview

> [!abstract] 문서 목적
> 본 연구와 관련된 문헌을 세 가지 연구 축으로 나누고, 각 축이 해결한 문제와 아직 남아 있는 문제를 연결한다.

> [!info] 관련 문서
> [[motivation/motivation|Motivation]] · [[related_works/continual_reinforcement_learning|Continual Reinforcement Learning]] · [[related_works/morphology_aware_policy_learning|Morphology-Aware Policy Learning]] · [[related_works/cross_embodiment_learning|Cross-Embodiment Learning]]


## Continual Reinforcement Learning

[[related_works/continual_reinforcement_learning|Continual reinforcement learning]]은 task가 순차적으로 주어질 때 새로운 task를 학습하면서 이전 task의 성능과 지식을 유지하는 문제를 다룬다. 기존 방법은 주로 replay, regularization, architecture 또는 parameter isolation으로 구분된다.

### Replay-Based Approaches

Replay-based method는 제한된 memory에 과거 experience를 저장하고 새로운 task의 data와 함께 다시 학습한다. CLEAR는 on-policy learning과 off-policy replay를 결합하고, replay sample에 behavioral cloning을 적용해 이전 policy를 보존한다 [@rolnick2019clear]. RECALL은 task별 reward scale과 replay distribution의 차이를 adaptive normalization과 policy distillation으로 보정한다 [@zhang2023recall]. Replay는 이전 experience를 직접 활용할 수 있지만, 과거 data 저장이 허용되어야 하며 memory 크기에 따라 조건이 달라진다.

### Regularization-Based Approaches

Regularization-based method는 이전 task에서 중요했던 parameter나 policy의 변화를 제한한다. EWC는 Fisher information으로 parameter importance를 추정하고 중요한 weight의 변화를 억제한다 [@kirkpatrick2017ewc]. Policy Consolidation은 여러 시간 규모의 과거 policy를 유지하고 현재 policy를 자신의 history에 regularize한다 [@kaplanis2019policy]. Progress & Compress는 새 task를 학습하는 active column과 지식을 축적하는 knowledge base를 분리하고, distillation과 EWC로 지식을 통합한다 [@schwarz2018progress]. 이러한 방법은 과거 transition을 직접 저장하지 않을 수 있지만, 제약이 강하면 새로운 task 학습을 방해할 수 있다.

### Architecture and Parameter Isolation

Architecture 또는 parameter-isolation method는 task별 module, prompt나 mask를 이용해 서로 다른 task가 사용하는 parameter를 분리한다. P2DT는 offline Decision Transformer에 progressive prompt를 추가하여 task-specific context를 보존한다 [@wang2024p2dt]. VQ-CD는 heterogeneous observation/action space를 vector-quantized latent space에 정렬하고, sparse task mask로 diffusion policy의 일부 weight를 선택한다 [@hu2025vqcd]. VQ-CD는 서로 다른 interface를 직접 다룬다는 점에서 cross-embodiment continual RL과 가깝지만, parameter 선택에 robot morphology나 kinematic connectivity를 사용하지 않는다.

### Benchmarks and Evaluation

Continual World는 Meta-World task sequence를 이용해 catastrophic forgetting뿐 아니라 이전 학습이 새로운 task의 학습에 미치는 forward transfer를 함께 평가한다 [@wolczyk2021continualworld]. 다만 하나의 robot embodiment와 고정된 observation/action interface를 사용하므로 embodiment 변화에 따른 transfer와 forgetting은 포함하지 않는다.

## Morphology-Aware Policy Learning

[[related_works/morphology_aware_policy_learning|Morphology-aware policy learning]]은 joint, link, actuator와 body connectivity 같은 robot 구조를 policy의 입력이나 architecture에 반영한다. 기존 방법은 graph-based encoding, morphology descriptor conditioning, implicit morphology inference와 universal interface로 구분할 수 있다.

### Graph-Based Structural Encoding

Shared Modular Policies는 actuator마다 동일한 policy module을 사용하고 morphology graph를 따라 message를 전달한다. Kinematic connectivity를 policy computation에 직접 사용하여 state/action dimension과 skeletal structure가 다른 agent를 하나의 shared policy로 제어한다 [@huang2020shared]. 이 방법은 topology를 명시적으로 사용하지만, 여러 embodiment에 동시에 접근하는 joint training을 전제로 한다.

### Morphology Conditioning and Inference

MetaMorph는 morphology descriptor와 proprioceptive state를 joint-level token으로 표현해 Transformer policy를 학습한다 [@gupta2022metamorph]. ModuMorph는 morphology-conditioned hypernetwork와 attention modulation으로 shared controller를 robot별로 조절한다 [@xiong2023modumorph]. 반면 AnyMorph는 hand-designed morphology descriptor를 제공하지 않고 sensor와 action token의 관계에서 morphology representation을 학습한다 [@trabucco2022anymorph]. 이 방법들은 unseen morphology에 대한 generalization과 transfer를 다루지만 순차 학습 이후의 이전 성능 유지는 평가하지 않는다.

### Structure-Compatible Universal Interfaces

URMA는 robot observation을 joint, foot과 global state로 분리하고 component description 기반 attention encoder와 universal decoder를 사용한다. 이를 통해 joint와 foot 수가 다른 quadruped, biped와 hexapod를 하나의 locomotion policy로 처리한다 [@bohlinger2025urma]. 다만 parent–child connectivity를 message passing에 직접 사용하지 않으므로, explicit topology-aware method보다는 component-wise universal interface에 가깝다.

## Cross-Embodiment Learning

[[related_works/cross_embodiment_learning|Cross-embodiment learning]]은 서로 다른 robot 사이에서 representation, reward, model 또는 policy를 공유하고 이전하는 연구 범위다. 주요 설정은 multi-embodiment pre-training, source-to-target adaptation과 pooled offline learning이다.

### Representation and Reward Transfer

XIRL은 서로 다른 embodiment의 video demonstration에서 task progression을 나타내는 공통 representation을 학습하고 이를 reward로 사용한다 [@zakka2022xirl]. PEAC은 reward-free interaction에서 embodiment-aware하고 task-agnostic한 representation을 사전학습한 뒤 downstream task와 새로운 embodiment에 활용한다 [@ying2024peac]. 두 방법은 embodiment 차이를 넘어 재사용할 수 있는 representation을 학습하지만, policy가 embodiment를 순차적으로 학습하면서 이전 성능을 유지해야 하는 setting은 아니다.

### Policy Transfer and Adaptation

Efficient Morphology-Aware Policy Transfer는 pre-trained morphology-aware policy를 새로운 target embodiment로 이전할 때 adapter, prefix tuning과 일부 layer fine-tuning을 비교한다 [@przystupa2025efficient]. 이는 target adaptation의 parameter와 sample efficiency를 다루는 `pre-training → single target` 설정이며, target 학습 이후 source embodiment의 성능 유지는 요구하지 않는다.

### Offline and Pooled-Data Learning

Cross-Embodiment Offline Reinforcement Learning은 16개 legged robot의 offline dataset을 함께 사용해 shared control prior를 학습하고, morphology similarity 기반 grouping으로 robot 사이의 gradient conflict를 줄인다 [@abe2026cross]. Morphology가 learning dynamics와 관련될 수 있음을 보여주지만 모든 robot dataset에 동시에 접근하므로, dataset이 embodiment별로 순차적으로 주어지는 continual setting과는 다르다.

## Research Gap

기존 continual RL은 sequential task에서 knowledge retention과 transfer를 개선하지만 대체로 고정된 robot interface를 가정한다. Morphology-aware policy learning은 robot 구조를 이용해 여러 embodiment 사이에 parameter를 공유할 수 있음을 보여주지만 주로 joint training과 unseen-morphology generalization을 다룬다. Cross-embodiment learning은 embodiment 사이의 representation과 policy transfer 가능성을 보여주지만 대부분 pre-training, pooled training 또는 single-target adaptation에 초점을 둔다.

그러나 다음 질문은 아직 충분히 검증되지 않았다.

> 서로 다른 embodiment가 순차적으로 주어질 때, morphology의 구조적 정보를 continual learning mechanism에 사용하면 새로운 embodiment의 학습과 이전 embodiment의 성능 유지를 함께 개선할 수 있는가?

본 연구는 cross-embodiment learning 자체를 처음 제안하는 것이 아니다. Embodiment별 task와 dataset이 순차적으로 주어지는 상황에서, 기존 continual RL의 knowledge retention과 cross-embodiment learning의 knowledge transfer를 morphology-aware mechanism으로 연결했을 때 실제 이점이 있는지를 검증한다.
