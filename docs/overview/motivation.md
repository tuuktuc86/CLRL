---
title: Motivation
aliases:
  - 연구 동기
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - motivation
status: draft
---

# Motivation

## Continual RL은 순차적 경험에서 제어 지식을 축적해야 한다

Continual reinforcement learning은 RL 학습 대상이 순차적으로 변하는 [[overview/glossary#Continual Learning|Continual Learning]] 문제 설정이다. 이 설정의 핵심 관심은 agent가 새로운 능력을 습득하면서 이전에 학습한 능력을 얼마나 유지하고 재사용할 수 있는지에 있다. 따라서 장기간 학습하는 robot agent의 목표는 각 학습 대상을 독립적으로 해결하는 데 그치지 않고, 경험이 늘어날수록 재사용 가능한 control knowledge를 축적하는 것이어야 한다.

## 기존 Continual RL은 대체로 고정된 robot interface를 전제한다

기존 continual reinforcement learning은 새로운 task를 순차적으로 학습하면서 [[overview/glossary#Catastrophic Forgetting|catastrophic forgetting]]을 완화하는 데 초점을 두지만, 많은 연구가 task 사이에 동일한 observation/action space가 존재한다고 가정한다. 서로 다른 state/action dimension을 공통 latent space로 정렬하는 heterogeneous continual RL도 등장했지만, 각 task를 일반적인 vector space로 취급하므로 robot의 joint, link, actuator, kinematic connectivity와 같은 물리적 구조를 충분히 활용하지 않는다.

## 새로운 embodiment는 interface 변화와 반복 학습 비용을 만든다

실제 로봇 시스템에는 형태와 자유도가 서로 다른 robot들이 존재하며, 새로운 [[overview/glossary#Embodiment|embodiment]]가 추가될 때마다 모든 제어 지식을 처음부터 학습하는 것은 비효율적이다. 특히 locomotion처럼 embodiment가 달라도 목적이 유사한 경우에는 균형 유지, 접촉 제어, 관절 협응, 목표 속도 추종과 같이 재사용 가능한 제어 지식이 존재할 가능성이 높다.

## Cross-Embodiment Learning은 embodiment 사이의 지식 공유 가능성을 보여준다

[[overview/glossary#Cross-Embodiment Learning|Cross-Embodiment Learning]] 연구는 서로 다른 joint 수와 observation/action space를 가진 robot 사이에서도 representation, model 또는 policy를 공유하거나 이전할 수 있음을 보여준다. 특히 [[overview/glossary#Morphology|morphology]] 정보를 이용한 shared policy와 policy transfer의 성과는 서로 다른 embodiment가 완전히 독립된 문제가 아니라 일부 구조와 제어 원리를 공유하는 관련된 문제임을 시사한다.

## 기존 Cross-Embodiment Learning은 continual retention을 직접 다루지 않는다

기존 연구는 주로 여러 robot의 데이터를 동시에 사용하는 joint training, 여러 embodiment에서 학습한 model을 하나의 target embodiment로 이전하는 adaptation, 또는 모든 robot dataset을 함께 사용하는 pooled training을 고려한다. 이러한 설정에서는 과거 데이터나 model에 계속 접근할 수 있으므로 새로운 embodiment를 학습하면서 이전 embodiment의 능력을 보존해야 하는 continual learning 문제가 직접적으로 드러나지 않는다.

## Cross-Embodiment Continual Reinforcement Learning은 plasticity와 stability를 함께 요구한다

새로운 embodiment에 맞게 parameter를 자유롭게 갱신하면 [[overview/glossary#Plasticity|plasticity]]는 높아질 수 있지만 이전 robot의 policy가 손상될 수 있다. 반대로 과거 지식을 지나치게 보호하면 [[overview/glossary#Stability|stability]]는 높아질 수 있지만 새로운 embodiment의 학습과 [[overview/glossary#Forward Transfer|forward transfer]]가 제한될 수 있다. 따라서 [[overview/glossary#Cross-Embodiment Continual Reinforcement Learning|cross-embodiment continual RL]]은 새 지식을 학습하는 능력과 기존 지식을 유지하는 능력 사이의 [[overview/glossary#Stability–Plasticity Trade-off|stability–plasticity trade-off]]를 다뤄야 한다.

## Morphology는 선택적 지식 재사용을 위한 structural prior가 될 수 있다

서로 다른 embodiment의 전체 state/action dimension이 달라도 각 robot은 joint와 link의 구성, 연결 관계, 물리적 특성이라는 구조를 가진다. 이러한 morphology 정보는 과거에 획득한 지식 중 새로운 embodiment에서도 재사용할 부분과 이전 embodiment의 성능을 위해 보존할 부분을 구분하는 단서가 될 수 있다. 이 구분이 가능하다면 새로운 embodiment를 학습하기 위해 모든 shared representation을 동일하게 수정할 필요가 없다.

## 본 연구는 control knowledge의 지속적인 축적과 재사용을 목표로 한다

본 연구의 핵심은 서로 다른 observation/action shape를 하나의 model로 처리하는 데 그치지 않고, embodiment 사이의 구조적 관계를 이용해 control knowledge를 축적하고 재사용하는 것이다. 구체적으로 과거 embodiment의 지식을 이용해 이후 embodiment에 대한 forward transfer를 높이는 동시에, 새 embodiment를 학습할 때 이전 embodiment에서 발생하는 catastrophic forgetting을 완화하고자 한다.

## Research Question

> **Can structural knowledge of robot embodiments enable continual reinforcement learning to acquire control capabilities for new embodiments more efficiently while retaining previously learned capabilities?**

이 질문을 통해 서로 다른 embodiment를 독립적인 task의 집합으로 다루는 것을 넘어, 여러 robot에서 얻은 제어 지식을 지속적으로 축적하고 새로운 embodiment에 활용하는 continual robot learning의 가능성을 탐구한다. 관련 문헌과 구체적인 연구 공백은 [[overview/related_works|Related Works]]에 정리한다.
