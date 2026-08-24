---
title: Wiki
aliases:
  - 위키
  - 용어 정리
tags:
  - terminology
  - continual-reinforcement-learning
status: living
---

# Wiki

> [!abstract] 목적
> 문서를 작성하거나 연구를 진행할 때 단어의 정의가 명확하지 않거나 같은 단어가 서로 다른 의미로 사용될 수 있다. 이 문서는 이러한 혼동을 예방하기 위해 용어의 의미를 정리한다.

> [!important] 규칙
> 1. 사용하는 단어를 정의한다.
> 2. 비슷한 단어와 반대되는 단어 사이의 관계를 함께 정리한다.


## Continual Learning

### 정의

Continual learning은 학습 task나 data distribution이 시간에 따라 순차적으로 주어지고, 모델이 이전 단계에서 획득한 지식을 유지하거나 활용하면서 하나의 학습 과정을 계속 이어가는 문제 설정이다.

각 단계를 독립된 model로 처음부터 학습하는 것이 아니라, 이전 단계의 model parameter 또는 허용된 persistent state를 다음 단계로 전달한다. 새로운 task를 순서대로 학습한다는 절차만으로는 충분하지 않으며, 새로운 학습 이후에도 이전 task의 성능을 평가할 수 있어야 한다.

### 조건


   학습 task는 한 번에 모두 주어지지 않고 단계별로 주어진다. 과거 단계의 task나 미래 단계의 task의 dataset은 사용할 수 없다.(그러나 rehearsal 방법같이 일부 replay buffer를 공유하는 방식은 조건부 허용됨)


### 유사 단어 사이의 정리

- **Fine-tuning**: 학습된 model을 새로운 task의 data로 추가 학습하는 절차다. 이전 task의 성능 유지까지 요구하지는 않는다.
- **Active learning**: 제한된 비용 안에서 어떤 data를 수집하거나 학습에 사용할지 선택하는 방법이다. 순차적인 지식 유지가 핵심은 아니다.
- **Transfer learning**: 이전 task에서 얻은 지식을 새로운 task의 학습에 활용한다. 새로운 task를 학습한 뒤 이전 task의 성능을 유지할 필요는 없다.
- **Meta-learning**: 여러 task를 통해 새로운 task에 빠르게 적응하는 능력을 학습한다. Task의 순차적 도착과 이전 성능의 유지를 반드시 요구하지는 않는다.


## Continual Reinforcement Learning

Continual reinforcement learning은 [[#Continual Learning]]을 reinforcement learning에 적용한 문제 설정이다. Environment, task, dynamics, reward 또는 dataset이 순차적으로 주어지며, agent는 이전에 학습한 policy와 value knowledge를 유지하거나 활용하면서 계속 학습한다.

Offline dataset의 quality가 혼합되어 있다면 reward와 return을 이용해 더 좋은 behavior를 구분하고 policy 성능을 개선할 수 있어야 한다.



## Stability

새로운 task를 학습한 이후에도 이전 task에서 획득한 성능을 유지할 수 있는 능력이다.

## Catastrophic Forgetting

새로운 task를 학습한 이후 이전 task에서 획득한 지식이나 성능이 크게 감소하는 현상이다. 이전 성능을 유지하는 능력인 [[#Stability|stability]]가 부족할 때 나타날 수 있다.

## Forgetting

새로운 task를 학습한 뒤 이전 task의 성능이 감소하는 현상이다. metric으로 사용할 때는 이전 task를 학습한 직후의 성능과 이후 시점의 성능 차이로 나타낸다. 이후 학습으로 이전 task의 성능이 개선되면 forgetting 값은 음수가 될 수 있다.

[[#Catastrophic Forgetting|Catastrophic forgetting]]은 forgetting이 커서 이전에 획득한 지식이나 성능이 현저하게 손실된 경우를 뜻한다.

## Forward Transfer

이전 학습이 이후 학습의 성능이나 학습 속도에 미치는 영향을 뜻한다. 이전 task를 먼저 학습한 경우와 새로운 task를 처음부터 학습한 경우를 비교하여 측정한다.

## Backward Transfer

이후 학습이 이전에 학습한 task의 성능에 미치는 영향을 뜻한다. 새로운 task를 학습하기 전과 후의 이전 task 성능을 비교하여 측정한다.

## Normalized Score

서로 다른 task나 environment의 return scale을 비교하기 위해 raw return을 기준 성능에 상대적인 값으로 변환한 score다. 어떤 기준을 사용하는지는 평가 protocol에서 명시해야 한다.

예를 들어 random policy와 expert policy의 성능을 각각 `0`과 `100`의 기준으로 사용할 수 있다. 두 값은 절대적인 하한과 상한이 아니므로 normalized score는 `0`보다 작거나 `100`보다 클 수 있으며, raw return과 함께 해석해야 한다.

## Area Under the Learning Curve (AUC)

학습 진행량에 따른 성능 곡선 아래의 면적이다. 같은 학습 budget에서 최종 성능뿐 아니라 학습 과정에서 얼마나 빠르게 성능을 획득했는지를 하나의 값으로 요약한다.

AUC가 높더라도 최종 성능이 반드시 높은 것은 아니므로 최종 성능과 함께 해석해야 한다. 실제 값은 학습 중 여러 checkpoint에서 측정한 성능으로 learning curve를 구성한 뒤 수치적으로 근사한다.

## Embodiment

Embodiment는 agent가 특정한 몸을 통해 환경을 감지하고 행동하는 방식을 포함한 구체적인 구성이다. 몸의 [[#Morphology|morphology]]뿐 아니라 sensor, actuator와 observation/action interface를 포함한다.

쉽게 말하면 서로 다른 robot은 일반적으로 서로 다른 embodiment다. 같은 robot도 sensor나 제어 방식이 달라지면 다른 embodiment로 볼 수 있으므로, 구체적인 구분 기준은 실험 조건에서 밝혀야 한다.

## Cross-Embodiment

서로 다른 [[#Embodiment|embodiment]] 사이에서 학습하거나 지식을 공유·이전하는 범위를 뜻한다.

## Morphology

Robot의 몸을 구성하는 구조적·물리적 속성을 뜻한다. [[#Topology|topology]], 다리 길이, 몸통 크기, 질량, 관절 위치, link 길이와 재료의 탄성 등이 포함된다.

Morphology는 [[#Embodiment|embodiment]]의 일부이며 sensor나 observation/action interface 자체를 뜻하지 않는다.

## Morphology Descriptor

morphology descriptor는 [[#Morphology|morphology]]를 구성하는 구조적·물리적 속성을 model이 사용할 수 있는 수치로 표현한 것이다. relative position, joint axis와 range, mass, actuator limit 또는 control parameter 등이 포함될 수 있다.

morphology descriptor가 반드시 [[#Topology|topology]]를 포함하는 것은 아니다. component 사이의 adjacency나 parent–child relation이 없다면 morphology의 속성은 나타낼 수 있지만 component의 연결 관계까지 알 수는 없다.

## Topology

Robot을 구성하는 body component와 그 연결 관계를 뜻한다. 쉽게 말하면 어떤 부품이 어떤 부품과 연결되어 있는가를 나타낸다. [[#Morphology|Morphology]]의 구조적 요소이므로, topology가 같아도 link 길이나 질량이 다르면 morphology는 다를 수 있다.

## 세 용어의 관계

- **Topology**: 어떤 부품이 어떻게 연결되어 있는가
- **Morphology**: 연결된 몸이 어떤 형태와 물리적 특성을 가지는가
- **Embodiment**: 그 몸으로 환경을 어떻게 감지하고 행동하는가
