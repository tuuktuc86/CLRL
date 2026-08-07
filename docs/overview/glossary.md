---
title: Glossary
aliases:
  - 용어집
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - glossary
status: living
---

# Glossary

이 문서는 프로젝트에서 작성하는 글의 canonical terminology를 정의한다. 논문 고유의 표현, 논문 제목, 방법 이름, dataset의 실제 field name은 원문을 유지하고, 그 밖의 프로젝트 서술은 아래 용어에 맞춘다.

정의는 연구가 구체화되는 과정에서 추가하거나 수정한다. 일반적인 RL 용어는 제외하고, 이 연구에서 의미 경계를 명확히 해야 하는 핵심 용어만 기록한다.

## Active Learning

제한된 labeling budget에서 학습자가 어떤 unlabeled sample의 label을 요청할지 선택하는 학습 설정이다. RL experience를 지속적으로 수집하는 것이나 학습 대상이 순차적으로 등장하는 것만으로는 Active Learning이라 하지 않는다.

## Adaptation

학습된 model이나 policy가 target task, domain 또는 embodiment에 관한 정보를 이용해 해당 target에 맞는 행동이나 성능을 획득하는 과정이다. Parameter를 갱신하는 [[#Fine-Tuning|Fine-Tuning]]은 Adaptation의 한 방법이며, conditioning이나 context inference처럼 parameter 업데이트가 없는 방법도 포함한다.

## Backward Transfer

이후 embodiment에서의 학습이 이전 embodiment에서 학습한 지식 또는 성능에 미치는 시간 방향의 transfer를 말한다. 영향이 유익한지 해로운지는 이 용어 자체에 포함하지 않는다.

## Catastrophic Forgetting

새로운 embodiment를 학습하는 과정에서 이전 embodiment에서 획득한 지식 또는 성능이 현저히 손실되는 현상이다.

_Avoid_: Backward Transfer와 동의어로 사용

## Continual Learning

학습 대상이 순차적으로 주어지며, 학습자가 이전에 획득한 지식을 보존하거나 활용하면서 계속 학습해야 하는 문제 설정이다. 학습 성능이나 catastrophic forgetting 방지의 성공 여부와 무관하며, 빠른 adaptation을 위한 [[#Meta-Learning|Meta-Learning]], data 선택을 위한 [[#Active Learning|Active Learning]], 특정 업데이트 절차인 [[#Fine-Tuning|Fine-Tuning]]을 전제하지 않는다.

_Avoid_: Sequential Training과 동의어로 사용

## Cross-Embodiment Continual Reinforcement Learning

서로 다른 embodiment가 학습 순서에 따라 등장하는 Continual Learning 문제 설정이다. 이 연구에서는 locomotion 목적을 가능한 한 유지하면서 morphology와 observation/action interface가 달라지는 경우를 다룬다.

## Cross-Embodiment Learning

서로 다른 embodiment 사이에서 experience, representation, model 또는 policy를 공유하거나 이전하는 연구 범위다. Reinforcement Learning뿐 아니라 imitation learning, representation learning, policy transfer와 embodiment 간 controller knowledge를 활용하는 robot co-design 등 여러 학습 설정을 포괄할 수 있다.

## Cross-Embodiment Reinforcement Learning

Cross-Embodiment Learning 중 RL experience, value function 또는 policy를 여러 embodiment 사이에서 공유하거나 이전하는 하위 연구 범위다. 그 자체로 sequential training이나 과거 성능 보존을 전제하지 않는다.

## Embodiment

하나의 robot 학습 단위를 이루는 구체적인 구성이다. robot의 morphology와 해당 robot에 결합된 observation/action interface를 포함한다.

_Avoid_: Morphology와 동의어로 사용

## Fine-Tuning

pre-trained model을 초기값으로 사용하여 target task, domain 또는 embodiment의 data와 objective로 parameter 일부 또는 전체를 추가 학습하는 절차다. [[#Adaptation|Adaptation]]에 사용할 수 있지만, 그 자체가 [[#Continual Learning|Continual Learning]]이나 [[#Meta-Learning|Meta-Learning]]을 뜻하지 않는다.

## Forward Transfer

이전 embodiment에서의 학습이 이후 embodiment의 학습에 미치는 시간 방향의 transfer를 말한다. 영향이 유익한지 해로운지는 이 용어 자체에 포함하지 않는다.

## Meta-Learning

여러 task나 embodiment에 걸쳐 model initialization, representation 또는 learning rule을 최적화하여 새로운 target을 적은 data나 update로 학습하도록 하는 접근이다. 빠른 [[#Adaptation|Adaptation]]이 핵심이며, 학습 대상의 순차적 도착이나 이전 target의 성능 보존을 반드시 요구하지 않는다.

## Morphology

이 연구에서 embodiment를 구성하는 robot 고유의 구조적·물리적 속성이다. kinematic topology, DoF, link/joint geometry, mass/inertia, joint limits, actuator limits 등을 포함하며 terrain, environment friction, reward, target velocity는 포함하지 않는다.

_Avoid_: Embodiment와 동의어로 사용

## Morphology-Aware Policy Learning

morphology를 policy의 입력으로 제공하거나 policy architecture에 명시적으로 반영하는 학습 접근이다. 그 자체로 multi-embodiment learning이나 Continual Learning을 뜻하지는 않는다.

## Plasticity

새로운 embodiment에 필요한 control knowledge를 습득하고 기존 model에 통합할 수 있는 능력이다. 과거 지식이 이후 학습에 미치는 영향인 Forward Transfer와는 구분한다.

## Sequential Training

여러 embodiment를 정해진 순서에 따라 하나씩 학습하는 training procedure다. Continual Learning 문제 설정에서 사용할 수 있지만, 그 자체가 이전 지식의 보존이나 catastrophic forgetting의 방지를 뜻하지는 않는다.

_Avoid_: Continual Learning과 동의어로 사용

## Stability

새로운 embodiment를 학습하는 동안 이전 embodiment에서 획득한 control knowledge와 제어 능력을 유지하는 능력이다. 능력의 보존을 뜻하므로, 성능 손실 현상인 Catastrophic Forgetting과는 구분한다.

## Stability–Plasticity Trade-off

새로운 embodiment를 학습하고 통합하는 [[#Plasticity|plasticity]]와 이전 embodiment에서 획득한 제어 능력을 보존하는 [[#Stability|stability]] 사이의 균형 문제다.
