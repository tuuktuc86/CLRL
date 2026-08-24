---
title: Morphology-Aware Policy Learning
aliases:
  - Morphology-Aware Related Works
tags:
  - morphology-aware-policy-learning
  - related-works
status: draft
bibliography: reference.bib
link-citations: true
---

# Morphology-Aware Policy Learning

> [!abstract] 문서 목적
> Robot의 morphology를 policy 입력이나 architecture에 반영하는 방법을 정리하고, topology를 직접 사용하는 방법과 그렇지 않은 방법의 차이를 구분한다.

> [!info] 관련 문서
> [[related_works/related_works_overview|Related Works Overview]] · [[related_works/continual_reinforcement_learning|Continual Reinforcement Learning]] · [[related_works/cross_embodiment_learning|Cross-Embodiment Learning]]

## 연구 범위

[[wiki#Morphology|Morphology]]-aware policy learning은 robot의 body structure와 physical property를 policy의 입력이나 parameter 구성에 반영한다. 목적은 morphology가 달라도 공유할 수 있는 control rule을 학습하거나, 특정 morphology에 맞게 policy를 조절하는 것이다.

Morphology-aware라는 사실만으로 [[wiki#Cross-Embodiment|cross-embodiment]] 또는 continual learning인 것은 아니다. 여러 robot을 동시에 학습할 수도 있고, 하나의 morphology family 안에서 generalization만 평가할 수도 있다. 또한 morphology descriptor를 사용하더라도 parent–child connectivity를 직접 사용하지 않으면 topology-aware method라고 구분하기 어렵다.

## Graph-Based Structural Encoding

### Shared Modular Policies

- **Problem:** State/action dimension과 skeletal structure가 다른 agent마다 별도의 policy와 hyperparameter를 학습해야 하는 비효율을 줄이는 문제를 다룬다.
- **Method:** 각 actuator에 동일한 local policy module을 배치하고 morphology graph의 연결을 따라 module 사이에 message를 전달하여 하나의 shared policy를 구성한다 [@huang2020shared].
- **Experiments:** Custom MuJoCo planar-locomotion suite의 monopod, quadruped와 biped agent를 함께 학습하고 held-out morphology로의 generalization을 평가했다.
- **Results:** Monopod, quadruped와 biped 등 skeletal structure가 다른 planar locomotion agent를 하나의 policy로 제어했으며, training에서 보지 못한 morphology variant에도 generalization했다.

이 방법은 kinematic connectivity를 policy computation에 직접 사용하므로 명시적인 topology-aware policy에 해당한다. 다만 여러 morphology의 data를 함께 사용하는 joint training이며, 새로운 embodiment 학습 이후 이전 embodiment의 성능을 평가하는 continual setting은 아니다.

## Descriptor-Conditioned Policies

### MetaMorph

- **Problem:** 조합 가능한 modular robot design space가 매우 커서 morphology마다 controller를 개별적으로 학습하기 어려운 문제를 다룬다.
- **Method:** 각 body module의 morphology descriptor와 proprioceptive state를 token으로 만들고, Transformer가 가변 길이 token sequence에 조건화된 universal controller를 학습한다 [@gupta2022metamorph].
- **Experiments:** 다양한 modular robot을 flat terrain, variable terrain과 obstacle locomotion task에서 사전학습하고 unseen morphology와 dynamics로의 generalization을 평가했다.
- **Results:** 다양한 modular robot에서 대규모 pre-training한 뒤 unseen dynamics, kinematics와 morphology에 zero-shot generalization했으며, 새로운 morphology와 task로의 transfer에 필요한 sample을 줄였다.

Morphology를 명시적인 입력으로 사용하지만 핵심 목적은 universal control과 unseen-morphology generalization이다. Embodiment가 순차적으로 추가되는 상황의 forgetting은 평가하지 않는다.

### ModuMorph

- **Problem:** 여러 morphology에 하나의 parameter를 그대로 공유하면 robot마다 다른 optimal control strategy와 limb interaction을 충분히 표현하기 어려운 문제를 다룬다.
- **Method:** Morphology-conditioned hypernetwork로 robot별 control parameter를 생성하고, morphology만으로 결정되는 attention을 사용해 limb 사이의 information flow를 조절한다 [@xiong2023modumorph].
- **Experiments:** 여러 modular robot morphology의 locomotion joint training과 held-out morphology에 대한 zero-shot generalization에서 평가했다.
- **Results:** 다양한 training robot에서 hard-sharing 기반 universal controller보다 학습 성능이 개선되었고, unseen morphology에 대한 zero-shot generalization도 향상되었다.

ModuMorph는 multi-morphology joint training과 generalization을 다루며, morphology를 continual knowledge consolidation의 기준으로 사용하지는 않는다.

## Implicit Morphology Inference

### AnyMorph

- **Problem:** Unseen morphology로 policy를 transfer하는 기존 방법이 hand-designed morphology description과 sensor-to-limb correspondence를 요구하는 문제를 다룬다.
- **Method:** Sensor token과 action token 사이의 관계를 RL objective로 학습하여 agent morphology를 암묵적으로 추론하고, 명시적인 morphology metadata 없이 transferable policy를 구성한다 [@trabucco2022anymorph].
- **Experiments:** Agent-agnostic control benchmark의 Cheetah, Walker, Humanoid와 Hopper morphology family를 조합해 학습하고 held-out agent의 zero-shot control을 평가했다.
- **Results:** Agent-agnostic control benchmark에서 explicit morphology description 없이도 unseen agent의 zero-shot control 성능이 기존 방법보다 향상되었다.

이 방법은 morphology를 암묵적으로 추론하므로 별도의 구조 metadata가 필요하지 않지만, 명시적인 topology가 transfer와 forgetting에 주는 효과를 분리해서 분석하기는 어렵다.

## Structure-Compatible Universal Interfaces

### URMA

- **Problem:** Quadruped, humanoid와 hexapod처럼 joint/foot 수와 observation/action dimension이 다른 legged robot을 하나의 end-to-end learning framework로 제어하는 문제를 다룬다.
- **Method:** Robot observation을 joint와 foot 단위로 encode하고 component description 기반 attention으로 가변 길이 입력을 통합하며, universal morphology decoder가 active joint마다 action을 생성한다 [@bohlinger2025urma].
- **Experiments:** Quadruped, humanoid와 hexapod를 포함한 16개 legged robot의 MuJoCo locomotion을 공동 학습하고 unseen simulated·real robot으로의 zero-shot transfer를 평가했다.
- **Results:** 여러 종류의 legged robot을 하나의 locomotion policy로 학습했고, training에서 보지 못한 robot platform으로 simulation 및 real world에서 policy를 transfer할 수 있음을 보였다.

URMA는 joint와 foot 수가 다른 robot을 하나의 interface로 처리할 수 있다. 그러나 joint와 foot의 parent–child edge를 message passing에 직접 사용하지 않으므로, 이 문서에서는 topology-aware graph method가 아니라 component-wise morphology-conditioned interface로 구분한다.

## 방법 비교

| Method | Morphology 사용 방식 | Explicit topology | 주된 학습 설정 |
| --- | --- | ---: | --- |
| Shared Modular Policies | Graph에 따른 actuator message passing | 사용 | Multi-embodiment joint training |
| MetaMorph | Joint-level morphology token | 직접 사용하지 않음 | Pre-training과 generalization |
| ModuMorph | Morphology-conditioned parameter modulation | 직접 사용하지 않음 | Multi-morphology joint training |
| AnyMorph | Sensor/action 관계에서 morphology 추론 | 사용하지 않음 | Cross-morphology transfer |
| URMA | Component description 기반 encoder/decoder | 사용하지 않음 | Multi-embodiment locomotion |

## 정리

Morphology-aware policy learning은 서로 다른 robot 사이에 공유 가능한 control representation을 만들 수 있음을 보여준다. 그러나 대부분은 모든 training embodiment에 함께 접근하거나 새로운 embodiment에 대한 generalization을 평가한다.

따라서 이 문헌은 **robot 구조를 policy에 어떻게 표현할 것인가**에 대한 근거를 제공하지만, 순차적으로 등장하는 embodiment 사이에서 어떤 지식을 보존하고 재사용할 것인지는 별도의 [[related_works/continual_reinforcement_learning|Continual Reinforcement Learning]] 문제로 남는다.
