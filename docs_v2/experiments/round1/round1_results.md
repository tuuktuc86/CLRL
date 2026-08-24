---
title: Round 1 Results
aliases:
  - Round 1 결과
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - experiment-results
status: draft
---

# Round 1 Results

> [!abstract] 문서 목적
> `Expert forward`, `Replay forward`, `Suboptimal 70 forward`에서 얻은 Round 1의 최종 성능과 결론을 정리한다.
> 각 결과는 final seed `43`, `44`, `45`의 평균과 표준편차(`mean ± std`)로 기록한다.

## 1. 주요 결과

각 metric은 training seed마다 먼저 계산한 뒤 final seed `43`, `44`, `45`의 `mean ± std`로 기록한다. Forward transfer는 첫 task를 제외한 네 task의 $FT_i$ 평균이다.

$$
FT
=
\frac{FT_{Go1}+FT_{\text{Barkour VB}}+FT_{Hexapod}+FT_{H1}}{4}.
$$

Single-task IQL은 continual learning method가 아니므로 세 metric을 계산하지 않는다. VQ-CD는 같은 architecture의 task-from-scratch reference가 없으면 forward transfer를 `N/A`로 표시한다. IQL + EG는 sequential training을 하지 않으므로 forgetting과 forward transfer를 계산하지 않는다.

### 1.1 Expert forward

| Method | Average performance $\uparrow$ | Forgetting $\downarrow$ | Forward transfer $\uparrow$ |
|---|---:|---:|---:|
| Single-task IQL | - | - | - |
| Sequential IQL |  |  |  |
| IQL + EWC |  |  |  |
| IQL + ER |  |  |  |
| IQL + PackNet |  |  |  |
| IQL + EG |  | - | - |
| VQ-CD |  |  |  |

### 1.2 Replay forward

| Method | Average performance $\uparrow$ | Forgetting $\downarrow$ | Forward transfer $\uparrow$ |
|---|---:|---:|---:|
| Single-task IQL | - | - | - |
| Sequential IQL |  |  |  |
| IQL + EWC |  |  |  |
| IQL + ER |  |  |  |
| IQL + PackNet |  |  |  |
| IQL + EG |  | - | - |
| VQ-CD |  |  |  |

### 1.3 Suboptimal 70 forward

| Method | Average performance $\uparrow$ | Forgetting $\downarrow$ | Forward transfer $\uparrow$ |
|---|---:|---:|---:|
| Single-task IQL | - | - | - |
| Sequential IQL |  |  |  |
| IQL + EWC |  |  |  |
| IQL + ER |  |  |  |
| IQL + PackNet |  |  |  |
| IQL + EG |  | - | - |
| VQ-CD |  |  |  |
