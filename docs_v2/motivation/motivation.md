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

Continual reinforcement learning은 RL task가 순차적으로 변하는 [[wiki#Continual Learning|Continual Learning]] 문제 설정이다. 이 설정의 핵심 관심은 agent가 새로운 능력을 습득하면서 이전에 학습한 능력을 얼마나 유지하고 재사용할 수 있는지에 있다. 따라서 장기간 학습하는 robot agent의 목표는 각 task를 독립적으로 해결하는 데 그치지 않고, 경험이 늘어날수록 재사용 가능한 control knowledge를 축적하는 것이어야 한다.

## 기존 Continual RL은 대체로 고정된 robot interface를 전제한다

기존 continual reinforcement learning은 새로운 task를 순차적으로 학습하면서 [[wiki#Catastrophic Forgetting|catastrophic forgetting]]을 완화하는 데 초점을 두지만, 많은 연구가 task 사이에 동일한 observation/action space가 존재한다고 가정한다. 서로 다른 state/action dimension을 공통 latent space로 정렬하는 heterogeneous continual RL도 등장했지만, 각 task를 일반적인 vector space로 취급하므로 robot의 joint, link, actuator, kinematic connectivity와 같은 물리적 구조를 충분히 활용하지 않는다.

## 새로운 embodiment는 interface 변화와 반복 학습 비용을 만든다

실제 로봇 시스템에는 형태와 자유도가 서로 다른 robot들이 존재하며, 새로운 [[wiki#Embodiment|embodiment]]가 추가될 때마다 모든 제어 지식을 처음부터 학습하는 것은 비효율적이다. 특히 locomotion처럼 embodiment가 달라도 목적이 유사한 경우에는 균형 유지, 접촉 제어, 관절 협응, 목표 속도 추종과 같이 재사용 가능한 제어 지식이 존재할 가능성이 높다.

## Cross-Embodiment Learning은 embodiment 사이의 지식 공유 가능성을 보여준다

[[wiki#Cross-Embodiment|Cross-Embodiment Learning]] 연구는 서로 다른 joint 수와 observation/action space를 가진 robot 사이에서도 representation, model 또는 policy를 공유하거나 이전할 수 있음을 보여준다. 특히 [[wiki#Morphology|morphology]] 정보를 이용한 shared policy와 policy transfer의 성과는 서로 다른 embodiment가 완전히 독립된 문제가 아니라 일부 구조와 제어 원리를 공유하는 관련된 문제임을 시사한다.

## Cross-Embodiment Continual RL은 embodiment별 data를 순차적으로 학습한다

Cross-embodiment continual RL에서는 각 stage마다 하나의 embodiment에 해당하는 task와 dataset이 주어지고, 다음 stage에서는 다른 embodiment의 task와 dataset을 학습한다. 모든 robot dataset에 동시에 접근하는 joint 또는 pooled training과 달리, 현재 stage에서는 현재 embodiment의 data만 사용한다. 따라서 observation/action interface가 달라지는 새로운 embodiment를 학습하면서 이전 embodiment의 능력을 유지하고, 과거에 획득한 control knowledge를 새로운 embodiment에 활용해야 한다.

## Cross-Embodiment Continual RL은 새로운 학습과 이전 성능 유지를 함께 요구한다

새로운 embodiment에 맞게 parameter를 자유롭게 갱신하면 새로운 control knowledge를 학습하기 쉬워질 수 있지만 이전 robot의 policy가 손상될 수 있다. 반대로 과거 지식을 지나치게 보호하면 [[wiki#Stability|stability]]는 높아질 수 있지만 새로운 embodiment의 학습과 [[wiki#Forward Transfer|forward transfer]]가 제한될 수 있다. 따라서 cross-embodiment continual RL은 새로운 control knowledge의 학습과 이전 embodiment의 성능 유지를 함께 고려해야 한다.

## Morphology는 선택적 지식 재사용을 위한 structural prior가 될 수 있다

서로 다른 embodiment의 전체 state/action dimension이 달라도 각 robot은 joint와 link의 구성, 연결 관계, geometry와 물리적 특성으로 이루어진 morphology를 가진다. 이 중 [[wiki#Topology|topology]]는 어떤 body component가 서로 연결되는지를 나타내는 구조적 하위 요소다. 이러한 morphology 정보, 특히 topology는 과거에 획득한 지식 중 새로운 embodiment에서도 재사용할 부분과 이전 embodiment의 성능을 위해 보존할 부분을 구분하는 단서가 될 수 있다. 이 구분이 가능하다면 새로운 embodiment를 학습하기 위해 모든 shared representation을 동일하게 수정할 필요가 없다.

## 본 연구는 control knowledge의 지속적인 축적과 재사용을 목표로 한다

본 연구의 핵심은 서로 다른 observation/action shape를 하나의 model로 처리하는 데 그치지 않고, embodiment 사이의 구조적 관계를 이용해 control knowledge를 축적하고 재사용하는 것이다. 구체적으로 과거 embodiment의 지식을 이용해 이후 embodiment에 대한 forward transfer를 높이는 동시에, 새 embodiment를 학습할 때 이전 embodiment에서 발생하는 [[wiki#Catastrophic Forgetting|catastrophic forgetting]]을 완화하고자 한다.

## Research Question

> **Can structural knowledge of robot embodiments enable continual reinforcement learning to acquire control capabilities for new embodiments more efficiently while retaining previously learned capabilities?**

이 질문을 통해 서로 다른 embodiment를 독립적인 task의 집합으로 다루는 것을 넘어, 여러 robot에서 얻은 제어 지식을 지속적으로 축적하고 새로운 embodiment에 활용하는 continual robot learning의 가능성을 탐구한다. 관련 문헌과 구체적인 연구 공백은 Related Works에 정리한다.


## 추후 고려해야 할 부분
별도로 offline rl이 continual learing 설정이 자연스러운지는 추후에 계속 논의 필요. 향후 방향을 multi task 쪽을 추가할수도 있음.