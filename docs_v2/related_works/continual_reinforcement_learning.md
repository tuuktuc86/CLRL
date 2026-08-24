---
title: Continual Reinforcement Learning
aliases:
  - Continual RL Related Works
tags:
  - continual-reinforcement-learning
  - related-works
status: draft
bibliography: reference.bib
link-citations: true
---

# Continual Reinforcement Learning

> [!abstract] 문서 목적
> Continual reinforcement learning의 주요 문제와 해결 방법을 정리하고, 서로 다른 observation/action interface를 다루는 방법이 어디까지 발전했는지 확인한다.

> [!info] 관련 문서
> [[related_works/related_works_overview|Related Works Overview]] · [[related_works/morphology_aware_policy_learning|Morphology-Aware Policy Learning]] · [[related_works/cross_embodiment_learning|Cross-Embodiment Learning]]

## 연구 범위

[[wiki#Continual Reinforcement Learning|Continual reinforcement learning]]은 RL task가 순차적으로 주어질 때 새로운 task를 학습하면서 이전 task의 지식과 성능을 유지하거나 재사용하는 문제를 다룬다. 주요 관심은 [[wiki#Catastrophic Forgetting|catastrophic forgetting]], [[wiki#Forward Transfer|forward transfer]], [[wiki#Backward Transfer|backward transfer]]다.

기존 방법은 주된 mechanism에 따라 replay, regularization, architecture 또는 parameter isolation으로 나눌 수 있다. 이 분류는 상호 배타적이지 않으며 실제 방법은 여러 mechanism을 함께 사용할 수 있다 [@khetarpal2022towards; @pan2025survey].

## Replay-Based Approaches

Replay-based method는 과거 experience의 일부를 저장하고 새로운 task의 data와 함께 다시 학습한다. 과거 data 접근을 제한하는 continual learning에서는 replay buffer의 허용 여부와 크기를 실험 조건으로 명시해야 한다.

### CLEAR

- **Problem:** 새로운 experience가 과거 지식을 덮어쓰는 catastrophic forgetting을 줄이면서도 현재 task를 계속 학습해야 하는 문제를 다룬다.
- **Method:** 현재 experience의 on-policy learning과 과거 experience의 off-policy replay를 결합하고, replay sample에는 behavioral cloning을 적용해 이전 policy의 변화를 제한한다 [@rolnick2019clear].
- **Experiments:** DeepMind Lab의 multi-task stream과 Atari 2600 game sequence에서 continual learning 성능을 평가했다.
- **Results:** 여러 multi-task RL 실험에서 task identity를 사용하지 않고도 기존 continual learning 방법보다 forgetting을 줄이면서 새로운 task의 학습 성능을 유지했다.

### RECALL

- **Problem:** 과거 task의 높은 reward scale이 현재 task 학습을 압도하고, replay된 offline data와 현재 policy 사이의 distribution shift가 다시 forgetting을 일으키는 문제를 다룬다.
- **Method:** Approximate target을 adaptive normalization하고 과거 task의 policy를 distillation하여 replay의 plasticity와 stability를 각각 보완한다 [@zhang2023recall].
- **Experiments:** Meta-World manipulation task로 구성된 Continual World benchmark에서 평가했다.
- **Results:** Continual World 실험에서 perfect-memory replay보다 새로운 task 학습과 이전 task 유지가 모두 개선되었고, 기존 continual learning 방법과 비슷하거나 더 높은 전체 성능을 보였다.

## Regularization-Based Approaches

Regularization-based method는 이전 task에서 중요했던 parameter나 policy가 새로운 task 학습 중 급격히 변하지 않도록 제한한다. 과거 transition을 직접 저장하지 않아도 사용할 수 있지만, 강한 제약은 새로운 task 학습을 방해할 수 있다.

### EWC

- **Problem:** 하나의 neural network가 task를 순차적으로 학습할 때 새 task의 parameter update가 이전 task에 중요한 parameter를 훼손하는 문제를 다룬다.
- **Method:** 이전 task에서 각 parameter의 중요도를 Fisher information으로 근사하고, 중요한 parameter가 기준값에서 멀어질수록 큰 quadratic penalty를 부과한다 [@kirkpatrick2017ewc].
- **Experiments:** Sequential MNIST 계열 classification과 Atari 2600 game sequence에서 평가했다.
- **Results:** Sequential MNIST 계열 classification과 Atari 2600 game sequence에서 naive fine-tuning보다 이전 task의 성능 손실을 줄이면서 새 task를 학습할 수 있음을 보였다.

EWC는 morphology나 heterogeneous interface를 직접 다루지 않는다. RL에 적용할 때는 actor, critic 또는 전체 model 중 어떤 parameter를 보호하는지 별도로 정해야 한다.

### Policy Consolidation

- **Problem:** Task boundary와 환경 변화의 시간 규모를 미리 알 수 없는 continual RL에서 policy가 과거 행동을 급격히 잊는 문제를 다룬다.
- **Method:** 현재 policy와 서로 다른 시간 규모의 과거 policy를 기억하는 hidden network cascade를 두고, 현재 policy를 자신의 장·단기 history에 대해 regularization한다 [@kaplanis2019policy].
- **Experiments:** MuJoCo continuous-control task와 두 task가 교대하는 sequence, RoboSumo의 Ant-vs-Ant competitive self-play에서 평가했다.
- **Results:** Single-task continuous control, 두 task가 교대하는 설정과 multi-agent competitive self-play에서 baseline보다 학습 중 성능 유지가 개선되었다.

### Progress & Compress

- **Problem:** Task가 늘어날 때 architecture나 task-specific parameter를 계속 확장하지 않으면서 이전 지식을 보존하고 새 task 학습을 가속하는 문제를 다룬다.
- **Method:** 현재 task를 학습하는 active column과 장기 지식을 저장하는 knowledge base를 분리하고, task 종료 후 distillation과 EWC를 이용해 새 지식을 knowledge base에 압축한다 [@schwarz2018progress].
- **Experiments:** Sequential handwritten-alphabet classification, Atari game sequence와 DeepMind Lab의 3D maze navigation에서 평가했다.
- **Results:** Sequential handwritten-alphabet classification, Atari와 3D maze navigation에서 과거 data를 저장하거나 model을 확장하지 않고도 이전 task 유지와 이후 task 학습을 함께 개선했다.

Policy Consolidation과 Progress & Compress는 policy 또는 parameter의 변화를 제한하지만, 서로 다른 robot의 observation/action interface나 morphology correspondence는 고려하지 않는다.

## Architecture and Parameter Isolation

이 접근은 task마다 사용하는 parameter를 분리하거나 필요한 module만 선택한다. Interference를 줄이기 쉽지만 task가 늘어날수록 parameter와 routing state가 증가할 수 있다.

### L2M

- **Problem:** 여러 RL task로 pre-training한 model을 새 task에 fine-tuning할 때 pre-training task의 성능이 크게 떨어지는 문제를 다룬다.
- **Method:** Pre-trained Decision Transformer backbone을 동결하고, learnable modulation pool로 backbone 내부의 information flow만 task에 맞게 조절한다 [@schmied2023l2m].
- **Experiments:** Meta-World와 DMControl dataset으로 backbone을 사전학습한 뒤 Continual World task sequence에서 adaptation을 평가했다.
- **Results:** Meta-World와 DMControl로 pre-training한 뒤 Continual World에서 adaptation했을 때 pre-training task의 성능을 유지하면서 benchmark의 기존 방법보다 높은 continual adaptation 성능을 보였다.

L2M은 사전학습된 backbone을 전제로 하므로 직접적인 비교에는 pre-training 조건과 비용을 함께 고려해야 한다.

### P2DT

- **Problem:** Offline trajectory를 순차적으로 학습하는 Decision Transformer가 새 task를 학습하면서 이전 task의 policy를 잊는 문제를 다룬다.
- **Method:** 새로운 task가 등장할 때 task-specific decision token을 progressive prompt로 추가하여 shared backbone 안에서 task별 context를 분리한다 [@wang2024p2dt].
- **Experiments:** D4RL `medium` dataset의 Gym MuJoCo HalfCheetah, Hopper와 Walker2D를 서로 다른 순서로 학습하는 continual offline RL sequence에서 평가했다.
- **Results:** 여러 continual offline RL task sequence에서 task 수가 증가해도 naive sequential Decision Transformer보다 forgetting을 완화하고 전체 성능의 확장성을 높였다.

Prompt는 task-specific parameter isolation을 제공하지만 robot의 joint나 link 관계를 transfer signal로 사용하지 않는다.

### VQ-CD

- **Problem:** 기존 continual offline RL이 동일한 observation/action space를 가정하여 interface dimension이 다른 task sequence를 직접 처리하기 어려운 문제를 다룬다.
- **Method:** Vector quantization으로 서로 다른 state/action space를 공통 latent space에 정렬하고, task-related sparse mask로 unified diffusion model의 weight를 선택하며 inverse dynamics model로 action을 복원한다 [@hu2025vqcd].
- **Experiments:** 동일한 interface에서는 MuJoCo Ant-dir와 Continual World CW10을, heterogeneous interface에서는 D4RL Hopper·Walker2d·HalfCheetah의 여러 dataset-quality sequence를 평가했다.
- **Results:** 동일한 space와 서로 다른 space를 포함한 여러 continual learning sequence에서 다수의 replay·regularization·architecture baseline보다 높은 전체 성능을 보였다.

VQ-CD는 heterogeneous interface를 직접 다룬다는 점에서 cross-embodiment continual RL과 가깝다. 다만 정렬과 parameter 선택은 명시적인 morphology나 kinematic connectivity가 아니라 학습된 latent representation과 task-related mask에 기반한다.

## Benchmark and Evaluation

### Continual World

- **Problem:** Continual RL 평가가 catastrophic forgetting에 집중하고, 이전 지식이 새 task 학습을 돕는 forward transfer와 현실적인 robotic task diversity를 충분히 측정하지 못하는 문제를 다룬다.
- **Method:** Meta-World의 manipulation task를 순차적으로 구성하고 average performance, forgetting과 forward transfer를 함께 측정하는 benchmark와 evaluation protocol을 제안한다 [@wolczyk2021continualworld].
- **Experiments:** Simulated Sawyer robot의 Meta-World manipulation task로 구성한 CW10, CW20과 짧은 task triplet에서 여러 continual learning baseline을 평가했다.
- **Results:** 여러 continual learning baseline을 비교해 forgetting을 줄이는 것만으로는 높은 continual performance가 보장되지 않으며, 새 task의 acquisition과 forward transfer를 함께 평가해야 함을 보였다.

Continual World는 하나의 robot embodiment와 고정된 observation/action interface를 사용하지만, 이전 task 유지 성능과 새로운 task 학습 성능을 동시에 측정해야 한다는 평가 원칙을 제공한다.

## 정리

기존 continual RL은 replay, regularization과 parameter isolation을 통해 forgetting을 완화하고 transfer를 개선한다. VQ-CD는 이 범위를 heterogeneous state/action space까지 넓혔다. 그러나 대부분의 방법은 robot의 morphology와 topology를 지식 보존이나 parameter 선택의 기준으로 직접 사용하지 않는다.

따라서 continual RL 문헌은 **순차 학습에서 무엇을 유지하고 어떻게 interference를 줄일 것인가**를 설명하고, [[related_works/morphology_aware_policy_learning|Morphology-Aware Policy Learning]]은 **robot 구조를 policy에 어떻게 반영할 것인가**를 설명하는 별도의 축으로 구분한다.
