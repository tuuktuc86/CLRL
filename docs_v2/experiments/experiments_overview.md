---
title: Experiments Overview
aliases:
  - Experiments 전체 설계
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - experiments
status: draft
---

# Experiments Overview

> [!abstract] 문서 목적
> 전체 실험의 목표와 Round별 진행 방향을 정리한다.

> [!info] 관련 문서
> [[experiments/dataset|Dataset]] · [[experiments/metric|Metric]] · [[experiments/baseline|Baseline]] · [[experiments/round1/round1_overview|Round 1 Overview]]

## 1. 실험 목표

실험의 목표는 morphology를 활용하는 제안 방법이 cross-embodiment continual reinforcement learning에서 기존 방법보다 높은 성능을 보이는지 검증하는 것이다. 이전 task의 성능을 유지하는 능력과 이전 지식을 새로운 task 학습에 활용하는 능력을 함께 평가한다.

순차적으로 dataset에 접근하는 continual learning 방법뿐 아니라 모든 task dataset에 접근하는 morphology-aware joint training도 함께 비교한다. 이를 통해 제안 방법이 순차 학습 조건에서 기존 continual learning 방법을 개선하는지, full-data-access 방법과 비교하면 어느 정도의 성능을 보이는지 확인한다.

## 2. Round 1: Baseline 재현

Round 1에서는 cross-embodiment offline RL dataset에서 baseline을 구현하고 성능 범위를 확인한다. parameter 수와 학습 조건은 가능한 범위에서 유사하게 맞추지만, 서로 다른 architecture를 사용하는 방법까지 엄격하게 같은 조건으로 제한하지는 않는다.

### 2.1 Dataset과 task sequence

다음 세 dataset을 서로 섞지 않고 독립적으로 실험한다.

- `expert forward`
- `replay forward`
- `suboptimal 70 forward`

각 dataset에서는 다음 다섯 robot을 하나의 task sequence로 학습한다.

```text
A1 → Go1 → Barkour VB → Hexapod → H1
```

### 2.2 비교 방법

| 구분 | Method | 역할 |
|---|---|---|
| Single-task reference | Single-task IQL | task별 독립 학습 성능과 scratch learning curve 측정 |
| Naive continual learning | Sequential IQL | 별도의 continual mechanism이 없는 순차 학습 기준선 |
| Regularization | IQL + EWC | parameter 보호를 통한 forgetting 완화 |
| Replay | IQL + ER | 제한된 과거 transition 재사용 |
| Parameter isolation | IQL + PackNet | task별 parameter 분리 |
| Full-access joint reference | IQL + EG | 모든 task dataset에 접근하는 morphology-aware joint training |
| Heterogeneous-space continual RL | VQ-CD | 서로 다른 observation/action space를 다루는 기존 방법 |

### 2.3 측정 지표

continual learning 방법은 다음 세 지표로 평가한다.

- **Average performance:** 전체 sequence 학습 후 모든 task의 평균 성능
- **Forgetting:** 각 task를 학습한 직후와 전체 sequence 종료 후의 성능 차이
- **Forward transfer:** sequential learning curve와 task-from-scratch learning curve의 AUC 차이

구체적인 baseline 구현과 실행 설정은 [[experiments/round1/round1_baseline_reproduce|Baseline Reproduction]], 실험 과정의 수치는 [[experiments/round1/round1_notes|Round 1 Notes]], 최종 비교 결과는 [[experiments/round1/round1_results|Round 1 Results]]에 기록한다.

## 3. Round 2: 제안 방법 평가

Round 2에서는 morphology를 활용하는 제안 방법을 구현하고 Round 1 baseline과 비교한다. 제안 방법은 하나의 고정된 구조로 미리 한정하지 않으며, 여러 설계안을 실험한 뒤 가능성 있는 방법을 발전시킨다.

dataset, task sequence와 평가 지표는 Round 1의 설정을 기준으로 유지한다. 최종적으로 제안 방법이 Round 1에서 확인한 baseline보다 average performance, forgetting과 forward transfer에서 어떤 차이를 보이는지 평가한다.
