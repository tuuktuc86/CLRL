---
title: Round 1 Notes
aliases:
  - Round 1 실험 메모
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - experiment-notes
status: living
---

# Round 1 Notes

> [!abstract] 문서 목적
> Round 1 실험 과정에서 생성되는 seed별 checkpoint 성능, stage 종료 성능, 학습 상태와 관찰 내용을 dataset별로 기록한다.
> 모든 final seed의 실행이 끝난 뒤 계산한 주요 결과와 결론은 [[experiments/round1/round1_results|Round 1 Results]]에 정리한다.

> [!info] 기록 형식
> - Final seed `43`, `44`, `45`는 서로 다른 표에 기록한다.
> - 각 seed의 checkpoint 성능은 evaluation seed `1000`–`1049`에서 얻은 `50` episode의 평균이다.
> - 개별 seed의 checkpoint 성능 cell은 `normalized score (raw return)` 형식을 사용한다.
> - Seed aggregate의 checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)` 형식을 사용한다. 표준편차의 표본 단위는 training seed다.
> - Learning curve는 `0%, 10%, ..., 100%` checkpoint에서 기록한다.
> - AUC와 forward transfer는 normalized score curve로 training seed마다 계산한다. Metric cell에는 raw return을 병기하지 않는다.
> - `Target AUC`는 sequential learning curve의 AUC, `Scratch AUC`는 같은 architecture의 task-from-scratch curve의 AUC다.
> - IQL 기반 sequential method의 Scratch AUC는 같은 dataset과 seed의 Single-task IQL을 사용한다.
> - VQ-CD의 Scratch AUC는 VQ-CD task-from-scratch reference가 있을 때만 기록하며, 해당 run이 없으면 Scratch AUC와 forward transfer를 `N/A`로 표시한다.
> - AUC와 forward transfer의 aggregate는 seed별 metric을 먼저 계산한 뒤 평균과 표준편차를 구한다.

## 1. Expert forward

### 1.1 Single-task IQL

#### Seed 43

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.2 Sequential IQL

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.3 IQL + EWC

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.4 IQL + ER

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.5 IQL + PackNet

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.6 IQL + EG

IQL + EG는 모든 robot dataset에 동시에 접근하므로 sequential evaluation matrix, forgetting과 forward transfer를 기록하지 않는다.

#### Seed 43

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.7 VQ-CD

VQ-CD task-from-scratch reference가 있을 때만 forward transfer를 기록한다.

#### Task-from-scratch reference

Forward transfer를 계산할 때만 실행한다. 실행하지 않으면 아래 표와 sequential learning curve 표의 Scratch AUC 및 forward transfer를 `N/A`로 표시한다.

##### Seed 43

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 44

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 45

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 평균과 표준편차

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |


#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.8 이전 task 성능 곡선

Target stage를 학습하는 동안 이전 task에서 측정한 checkpoint 성능을 기록한다. 현재 target의 learning curve는 각 method 표에 이미 있으므로 이 표에는 이전 task 평가만 추가한다.

#### Seed 43

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

#### Seed 44

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

#### Seed 45

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

### 1.9 Diagnostic

학습 상태를 판단하는 데 필요한 값만 기록한다. Method에 존재하지 않는 loss나 diagnostic은 `-`로 표시한다.

| Seed | Method | Run ID | Stage | Update | Actor 또는 policy loss | Q loss | V loss | Auxiliary loss 또는 metric | Gradient norm | GPU memory | Wall-clock time | 메모 |
|---:|---|---|---|---:|---:|---:|---:|---|---:|---:|---:|---|
| 43 |   |   |   |   |   |   |   |   |   |   |   |   |
| 44 |   |   |   |   |   |   |   |   |   |   |   |   |
| 45 |   |   |   |   |   |   |   |   |   |   |   |   |

### 1.10 실험 메모

-

## 2. Replay forward

### 2.1 Single-task IQL

#### Seed 43

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.2 Sequential IQL

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.3 IQL + EWC

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.4 IQL + ER

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.5 IQL + PackNet

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.6 IQL + EG

IQL + EG는 모든 robot dataset에 동시에 접근하므로 sequential evaluation matrix, forgetting과 forward transfer를 기록하지 않는다.

#### Seed 43

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.7 VQ-CD

VQ-CD task-from-scratch reference가 있을 때만 forward transfer를 기록한다.

#### Task-from-scratch reference

Forward transfer를 계산할 때만 실행한다. 실행하지 않으면 아래 표와 sequential learning curve 표의 Scratch AUC 및 forward transfer를 `N/A`로 표시한다.

##### Seed 43

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 44

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 45

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 평균과 표준편차

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |


#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.8 이전 task 성능 곡선

Target stage를 학습하는 동안 이전 task에서 측정한 checkpoint 성능을 기록한다. 현재 target의 learning curve는 각 method 표에 이미 있으므로 이 표에는 이전 task 평가만 추가한다.

#### Seed 43

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

#### Seed 44

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

#### Seed 45

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

### 2.9 Diagnostic

학습 상태를 판단하는 데 필요한 값만 기록한다. Method에 존재하지 않는 loss나 diagnostic은 `-`로 표시한다.

| Seed | Method | Run ID | Stage | Update | Actor 또는 policy loss | Q loss | V loss | Auxiliary loss 또는 metric | Gradient norm | GPU memory | Wall-clock time | 메모 |
|---:|---|---|---|---:|---:|---:|---:|---|---:|---:|---:|---|
| 43 |   |   |   |   |   |   |   |   |   |   |   |   |
| 44 |   |   |   |   |   |   |   |   |   |   |   |   |
| 45 |   |   |   |   |   |   |   |   |   |   |   |   |

### 2.10 실험 메모

-

## 3. Suboptimal 70 forward

### 3.1 Single-task IQL

#### Seed 43

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.2 Sequential IQL

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.3 IQL + EWC

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.4 IQL + ER

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.5 IQL + PackNet

#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.6 IQL + EG

IQL + EG는 모든 robot dataset에 동시에 접근하므로 sequential evaluation matrix, forgetting과 forward transfer를 기록하지 않는다.

#### Seed 43

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

| Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.7 VQ-CD

VQ-CD task-from-scratch reference가 있을 때만 forward transfer를 기록한다.

#### Task-from-scratch reference

Forward transfer를 계산할 때만 실행한다. 실행하지 않으면 아래 표와 sequential learning curve 표의 Scratch AUC 및 forward transfer를 `N/A`로 표시한다.

##### Seed 43

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 44

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 45

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |

##### Seed 평균과 표준편차

| Robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |


#### Seed 43

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 44

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 45

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

#### Seed 평균과 표준편차

Checkpoint 성능 cell은 `normalized score mean ± std (raw return mean ± std)`로 기록한다. AUC와 forward transfer는 seed별 metric의 `mean ± std`로 기록한다.

##### Stage 종료 evaluation matrix

| 학습 완료 stage | A1 | Go1 | Barkour VB | Hexapod | H1 |
|---|---:|---:|---:|---:|---:|
| A1 |   | - | - | - | - |
| Go1 |   |   | - | - | - |
| Barkour VB |   |   |   | - | - |
| Hexapod |   |   |   |   | - |
| H1 |   |   |   |   |   |

##### Target learning curve와 AUC

| Target stage | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% | Target AUC | Scratch AUC | Forward transfer (ΔAUC) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Go1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Barkour VB |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| Hexapod |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| H1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.8 이전 task 성능 곡선

Target stage를 학습하는 동안 이전 task에서 측정한 checkpoint 성능을 기록한다. 현재 target의 learning curve는 각 method 표에 이미 있으므로 이 표에는 이전 task 평가만 추가한다.

#### Seed 43

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

#### Seed 44

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

#### Seed 45

| Method | 학습 중인 stage | Evaluation robot | 0% | 10% | 20% | 30% | 40% | 50% | 60% | 70% | 80% | 90% | 100% |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |

### 3.9 Diagnostic

학습 상태를 판단하는 데 필요한 값만 기록한다. Method에 존재하지 않는 loss나 diagnostic은 `-`로 표시한다.

| Seed | Method | Run ID | Stage | Update | Actor 또는 policy loss | Q loss | V loss | Auxiliary loss 또는 metric | Gradient norm | GPU memory | Wall-clock time | 메모 |
|---:|---|---|---|---:|---:|---:|---:|---|---:|---:|---:|---|
| 43 |   |   |   |   |   |   |   |   |   |   |   |   |
| 44 |   |   |   |   |   |   |   |   |   |   |   |   |
| 45 |   |   |   |   |   |   |   |   |   |   |   |   |

### 3.10 실험 메모

-
