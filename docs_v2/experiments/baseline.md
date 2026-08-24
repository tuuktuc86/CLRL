---
title: Baseline
aliases:
  - 비교 방법
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - offline-reinforcement-learning
  - baseline
status: draft
bibliography: ../related_works/reference.bib
link-citations: true
---

# Baseline

> [!abstract] 문서 목적
> cross-embodiment continual offline RL에서 비교할 baseline과 reference의 역할을 정리한다.
> 구체적인 model 크기, 학습 budget과 hyperparameter는 각 Round 문서에서 결정한다.

> [!info] 관련 문서
> [[experiments/dataset|Dataset]] · [[experiments/metric|Metric]] · [[related_works/continual_reinforcement_learning|Continual Reinforcement Learning]] · [[related_works/cross_embodiment_learning|Cross-Embodiment Learning]]

## 1. overview
baseline은 새로운 embodiment를 학습하면서 이전 embodiment의 성능을 유지하고, 이전에 학습한 지식을 새로운 embodiment에 활용할 수 있는지를 비교하기 위해 사용한다.

이 문서에서는 어떤 baseline을 사용하는지만 작성하고 구현에 관련된 내용은 experiments의 각 round 문서 별로 기록하겠다.


## 2. 비교 방법

| 구분 | Category | Method | 확인할 질문 |
|---|---|---|---|
| Reference | Single-task | **Single-task IQL** | 각 task를 독립적으로 학습했을 때의 기준 성능과 학습 속도는 얼마인가? |
| Main CL | Naive | **Sequential IQL** | 별도의 continual mechanism 없이 순차 학습하면 얼마나 잊는가? |
| Main CL | Regularization | **IQL + EWC** | 중요한 parameter를 보호하는 것만으로 forgetting을 완화할 수 있는가? |
| Main CL | Replay | **IQL + ER** | 소량의 과거 transition을 저장하면 이전 성능 유지와 새로운 task 학습이 얼마나 개선되는가? |
| Main CL | Parameter isolation | **IQL + PackNet** | task별 parameter를 분리하면 forgetting을 막으면서 새로운 task를 학습할 수 있는가? |
| Joint reference | Morphology-aware pooled training | **IQL + EG** | full-data-access morphology-aware 방법은 sequential continual learning 방법과 어떤 성능 차이를 보이는가? |
| Main CL | Heterogeneous-space CORL | **VQ-CD** | heterogeneous observation/action space를 위한 기존 continual offline RL 방법이 cross-embodiment locomotion에서도 작동하는가? |
| Proposed | Morphology-aware continual learning | **Ours** | morphology-aware continual mechanism이 기존 방법보다 stability와 forward transfer를 개선하는가? |

## 3. 공통 기반 알고리즘: IQL

본 연구는 offline 환경에서의 continual RL을 다루기 때문에 공통 기반 알고리즘으로 IQL을 사용한다. IQL은 대표적인 offline RL 알고리즘이며, 별도의 environment interaction 없이 고정된 dataset으로 policy를 학습할 수 있다.

또한 같은 IQL objective에 regularization, replay와 parameter isolation을 각각 적용할 수 있다. 이를 통해 offline RL 알고리즘의 차이보다 각 continual learning mechanism이 forgetting과 새로운 task 학습에 미치는 영향을 비교한다.

다만 VQ-VC 처럼 모든 알고리즘이 IQL 기반으로 동작하지는 않는다.

## 4. Baselines

IQL 기반 baseline은 각 원 논문에서 제안한 continual learning mechanism을 가져와 공통 IQL에 적용한다. 구체적인 architecture, parameter 적용 범위와 재현 방법은 각 Round 실험 문서에서 정의한다.

### 4.1 Single-task IQL [@kostrikov2022iql]

Single-task IQL은 각 embodiment를 처음부터 독립적으로 학습한다. Continual learning method가 아니며, 다음 두 기준을 제공한다.

- robot별 독립 학습 성능
- sequential learning curve와 비교하는 [[experiments/metric#3.3 Forward Transfer|forward transfer]] AUC reference

Single-task model은 다른 embodiment에서 학습한 parameter나 data를 사용하지 않는다.

### 4.2 Sequential IQL [@kostrikov2022iql]

Sequential IQL은 IQL을 순차 학습에 적용한 별도의 forgetting 완화 방법이 없는 기준선이다. task $i$의 IQL 학습이 끝나면 model parameter를 다음 task로 전달하고, 다음 stage에서는 현재 task dataset만 사용한다.

이전 task의 raw transition이나 별도의 task-specific parameter는 저장하지 않는다. 따라서 Sequential IQL의 forgetting은 continual mechanism이 없을 때 발생하는 기본적인 interference를 보여준다.

### 4.3 IQL + EWC [@kirkpatrick2017ewc]

EWC에서 제안한 parameter importance 기반 regularization을 가져와 IQL에 적용한다. 이전 task에서 중요했던 parameter가 새로운 task 학습 중 크게 변하지 않도록 제한하며, 이전 parameter와 parameter importance를 저장하지만 과거 transition은 저장하지 않는다.

EWC는 Sequential IQL과 같은 IQL backbone을 사용한다. 두 방법의 차이는 parameter 보호가 stability와 새로운 task의 학습에 미치는 영향이다.

### 4.4 IQL + ER [@rolnick2019clear]

Experience Replay에서 제안한 과거 experience 재사용 방식을 가져와 IQL에 적용한다. 이전 task transition의 일부를 replay buffer에 저장하고 현재 task data와 함께 학습하며, 과거 data를 전혀 사용하지 않는 방법과 full-data joint training 사이의 제한된 data-access 조건을 나타낸다.

replay buffer 크기, task별 memory 할당과 current/replay sampling 비율은 결과에 직접 영향을 준다. 따라서 정확한 memory budget과 sampling rule은 Round별 실험 문서에서 고정하고, 저장한 transition 수와 memory 사용량을 함께 보고한다.

### 4.5 IQL + PackNet [@mallya2018packnet]

PackNet은 image classification의 continual learning을 위해 제안된 parameter-isolation 방법이다. PackNet에서 제안한 iterative pruning과 task별 parameter mask를 가져와 IQL에 적용한다.

과거 transition은 사용하지 않지만 task별 parameter mask와 evaluation 대상 task identity가 필요하다. Sequential IQL과 비교하여 parameter 분리가 forgetting을 줄이는지, 그리고 사용할 수 있는 parameter가 감소하면서 새로운 task의 학습이 제한되는지를 확인한다.

### 4.6 IQL + EG [@abe2026cross]

Embodiment Grouping은 morphology graph 사이의 거리를 이용해 robot을 group으로 나누고, group별 actor update를 수행한다. Critic은 여러 robot이 섞인 global minibatch로 업데이트하고 actor는 morphology group별 minibatch로 나누어 업데이트한다.

IQL + EG도 모든 embodiment dataset에 동시에 접근하는 pooled offline learning 방법이다. 따라서 upper bound나 continual learning baseline으로 해석하지 않고, static offline RL과 continual offline RL의 관계를 확인하는 **full-access morphology-aware reference**로 사용한다.

IQL + EG와 Ours의 차이는 모든 과거 dataset에 접근하는 joint training과 순차적인 dataset 접근 조건의 차이를 보여준다.

### 4.7 VQ-CD [@hu2025vqcd]

VQ-CD는 서로 다른 state/action space를 vector quantization으로 공통 latent space에 정렬하고, task-related sparse mask로 unified policy의 weight를 선택하는 continual offline RL 방법이다. 따라서 observation/action dimension이 다른 cross-embodiment sequence를 직접 다루는 기존 방법으로 사용한다.

VQ-CD는 IQL variant가 아니므로 parameter 수, representation 학습에 사용한 data, persistent state와 계산량을 별도로 기록한다. morphology나 kinematic connectivity를 직접 사용하지 않는다는 점에서 proposed method와 구분한다.

## 5. 과거 dataset 접근 수준

offline RL은 environment와 추가로 상호작용하지 않는 학습 조건이고, continual learning은 task와 dataset이 순차적으로 주어지는 접근 조건이다. dataset이 존재한다는 사실과 현재 stage에서 학습 algorithm이 그 dataset에 접근할 수 있다는 것은 구분한다.

| 접근 수준 | 사용 가능한 data | 해당 방법 |
|---|---|---|
| No past-data access | 현재 task dataset만 사용 | Sequential IQL, EWC, PackNet, VQ-CD, Ours |
| Limited past-data access | 현재 task dataset과 제한된 replay buffer | IQL + ER |
| Full joint access | 실험에 포함된 모든 embodiment dataset | IQL + EG |

현재 비교군을 continual learning으로만 잡아야 할지 multi task learning으로 잡아야 할지 명확하게 정해지지 않았기 때문에 모든 가능성을 열어두고 함께 비교한다.
