---
title: Metric
aliases:
  - 측정 지표
tags:
  - cross-embodiment
  - metric
  - offline-reinforcement-learning
status: draft
bibliography: ../related_works/reference.bib
link-citations: true
---

# Metric

## 1. Overview

본 문서에서는 *Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets*에서 제공하는 dataset을 continual reinforcement learning 환경에서 평가하기 위해 사용하는 성능 지표와 측정 방법을 기술한다.

## 2. Return과 score

$$S = 100 \times \frac{R - R_{random}}{R_{expert} - R_{random}}$$

우리는 다음과 같이 normalized score를 이용한다. 해당 score는 D4RL에서 사용하는 normalized score로 offline RL 분야에서는 이미 많이 사용되는 점수 체계이다. 100점은 expert 수준에 해당하는 policy이며 0점은 random하게 움직이는 것과 동일한 수준의 policy이다.

먼저 $R$을 어떻게 얻었는지 설명한다.

그리고 $R_{expert}$나 $R_{random}$를 어떻게 얻었는지도 이어서 설명한다. D4RL과 다르게 Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets 논문에 기술된 dataset에는 $R_{expert}$나 $R_{random}$에 대한 구체적인 명시가 없기 때문에, dataset과 시뮬레이션 환경에서 이를 추론한다.

### 2.1 $R$ 측정

$R$은 평가 대상 policy를 environment에서 굴려 얻은 episode return의 평균이다. 측정은 clean 환경에서 하며 observation noise, dropout, domain randomization, perturbation을 적용하지 않는다. command는 전 구간에서 전진 `(1, 0, 0)` 상수이고, 고정된 seed `1000`–`1049` 50개로 seed당 한 episode씩 총 50개의 episode를 굴린다. 각 episode는 최대 `1,000` step이며 넘어지거나 특정 body collision이 발생하면 일찍 종료된다.

policy는 stochastic policy의 평균 action을 사용하며, 그 action은 저장 규약인 `[-1, 1]`에서 robot별 action bound를 곱해 environment 단위로 되돌린다. 같은 절차를 학습 중 지정한 checkpoint마다 반복하여 학습 곡선을 얻는다.

#### 어떤 reward를 측정하는가

reward는 dataset을 만들 때와 동일한 reward function을 그대로 사용한다. 이 function은 tracking 2개 term과 penalty 13개 term으로 구성되며, penalty에만 curriculum 계수 $c_{\text{cur}}=\min(\text{total\_timesteps}/\text{curriculum\_steps},\,1)$이 곱해진다.

$c_{\text{cur}}$은 environment가 누적한 timestep에 따라 `0`에서 `1`로 증가한다. `curriculum_steps`는 PPO 학습 전체 기간을 기준으로 정해진 값이어서 robot에 따라 `12e6`–`50e6`인 반면, 평가용 environment는 매번 새로 만들어져 episode당 `1,000` step만 쌓인다. 따라서 평가 시점의 $c_{\text{cur}}$은 `10⁻⁴` 수준이며, **우리가 측정하는 $R$은 사실상 tracking term만으로 구성된다.**

offline RL이므로 학습에는 environment reward를 사용하지 않고 dataset에 저장된 reward를 읽는다. 그 값에는 수집 당시의 curriculum이 이미 반영되어 있다.

### 2.2 $R_{random}$과 $R_{expert}$

#### $R_{expert}$

expert dataset은 수렴한 PPO policy를 rollout하여 수집한 것이므로 이 dataset의 episode return이 곧 expert 수준이다. 다만 저장된 return에는 penalty가 반영되어 있어 2.1절의 $R$과 단위가 다르므로 그대로 쓸 수 없다.

따라서 expert dataset의 tracking-only return을 재구성하여 $R_{expert}$로 사용한다. tracking term은 다음과 같고 필요한 값이 모두 observation에 저장되어 있다.

$$r_{\text{track}}=c_{xy}\,\Delta t\,\exp\!\left(-\frac{\lVert v_{\text{goal}}-v_{xy}\rVert^{2}}{T}\right)+c_{\text{yaw}}\,\Delta t\,\exp\!\left(-\frac{\omega_{z}^{2}}{T}\right)$$

$\Delta t = 0.02$ (`50 Hz`), $T = 0.25$이며 episode horizon은 16종 모두 `1,000` step이다. $c_{xy}$와 $c_{\text{yaw}}$는 robot마다 다르므로 tracking 상한도 robot마다 다르며, 값은 upstream의 robot별 `rudin_own_var.py`에 있다. $v_{xy}$는 general dynamic state의 trunk linear velocity로, reward가 사용하는 것과 같은 local frame으로 저장되어 있다. $\omega_z$는 trunk yaw velocity다. 두 값 모두 저장 시 scaling (`/10`, `/50`)이 적용되어 있으므로 역변환 후 사용한다.


이 방식은 재평가 없이 expert를 우리 평가와 같은 단위로 옮긴다.

#### $R_{random}$

random dataset은 별도로 제공되지 않으므로 random policy를 직접 구현하여 측정한다. 매 step 각 joint에 대해 해당 robot의 action bound 안에서 uniform하게 action을 뽑는다.

#### $R_{zero}$

모든 joint에 `0`을 주는 policy도 같은 절차로 측정하여 함께 기록한다. action이 position offset이므로 0은 nominal 자세 유지를 뜻하고, 정적으로 안정한 robot에서는 넘어지지 않는다.
가만히 서있을때 0이 아닌 이유는 xy tracking reward는 거의 받지 못하지만 yaw tracking에서 거의 만점을 받기 때문이다.

| env | robot | $R_{random}$ | $R_{zero}$ | $R_{expert}$ | tracking 상한 |
|---:|---|---:|---:|---:|---:|
| 0 | `unitree_a1` | `3.38` | `20.09` | `58.32` | `60` |
| 1 | `unitree_go1` | `2.02` | `20.34` | `58.25` | `60` |
| 2 | `unitree_go2` | `0.39` | `17.25` | `58.35` | `60` |
| 3 | `anymal_b` | `1.40` | `20.49` | `58.17` | `60` |
| 4 | `anymal_c` | `1.52` | `20.56` | `56.95` | `60` |
| 5 | `barkour_v0` | `1.86` | `27.09` | `85.62` | `90` |
| 6 | `barkour_vb` | `5.22` | `20.38` | `58.24` | `60` |
| 7 | `badger` | `12.15` | `19.49` | `57.92` | `60` |
| 8 | `bittle` | `6.65` | `42.43` | `49.85` | `150` |
| 9 | `unitree_h1` | `0.09` | `0.40` | `57.20` | `60` |
| 10 | `unitree_g1` | `0.21` | `1.14` | `83.93` | `90` |
| 11 | `talos` | `0.19` | `0.68` | `116.45` | `120` |
| 12 | `robotis_op3` | `0.50` | `35.56` | `95.33` | `120` |
| 13 | `nao_v5` | `0.41` | `22.66` | `97.90` | `120` |
| 14 | `cassie` | `0.12` | `0.73` | `85.36` | `90` |
| 15 | `hexapod` | `6.67` | `41.17` | `96.29` | `120` |

- tracking 상한은 목표 속도를 완벽하게 추종하며 `1,000` step을 완주했을 때 받는 점수다. 동일한 제어 품질이라도 robot에 따라 `60`에서 `150`까지 달라지므로 robot 사이의 raw return은 직접 비교할 수 없다.
- reward는 음수를 가질 수 없다. 따라서 return이 episode 길이에 단조 증가한다.

## 3. Continual RL 실험을 위한 metric

continual RL method는 이전 task의 성능 유지, 새로운 task의 학습과 과거 지식의 활용을 함께 평가해야 한다. 하나의 값만으로 이 특성을 모두 설명하기 어렵기 때문에 average performance, forgetting과 forward transfer를 함께 보고한다 [@wolczyk2021continualworld].

task가 `1, 2, ..., T` 순서로 주어진다고 하자. 다음과 같이 score를 정의한다.

$$
S_{t,i}
=
\text{task }t\text{까지 학습한 뒤 task }i\text{에서 측정한 score}
$$

예를 들어 세 task를 순차적으로 학습하면 다음 evaluation matrix를 얻는다.

| 학습이 끝난 시점 | task 1 평가 | task 2 평가 | task 3 평가 |
|---|---:|---:|---:|
| task 1 | $S_{1,1}$ | - | - |
| task 2 | $S_{2,1}$ | $S_{2,2}$ | - |
| task 3 | $S_{3,1}$ | $S_{3,2}$ | $S_{3,3}$ |

서로 다른 robot의 raw return은 scale이 다르므로 task 사이를 평균하는 metric에는 2절의 normalized score를 사용한다. robot별 raw return은 별도로 함께 보고한다.

### 3.1 Average performance

average performance는 전체 sequence 학습이 끝난 뒤 지금까지 학습한 모든 task의 평균 성능을 나타낸다.

$$
P_T=\frac{1}{T}\sum_{i=1}^{T}S_{T,i}
$$

세 task를 학습했다면 다음과 같다.

$$
P_3=\frac{S_{3,1}+S_{3,2}+S_{3,3}}{3}
$$

average performance는 method의 전체 성능을 요약하지만, 이전 task 유지와 새로운 task 학습 중 어느 쪽에서 차이가 발생했는지는 보여주지 못한다. 따라서 forgetting과 forward transfer를 함께 해석한다.

### 3.2 Forgetting

forgetting은 task를 학습한 직후의 성능과 전체 sequence 학습이 끝난 뒤의 성능 차이다.

$$
F_i=S_{i,i}-S_{T,i}
$$

이전 task 전체의 평균 forgetting은 다음과 같다.

$$
F=\frac{1}{T-1}\sum_{i=1}^{T-1}F_i
$$

양수는 성능 감소, `0`은 성능 유지, 음수는 이후 task 학습으로 이전 task의 성능이 개선되었음을 의미한다. 세 task를 학습했다면 다음과 같다.

$$
F=\frac{(S_{1,1}-S_{3,1})+(S_{2,2}-S_{3,2})}{2}
$$

### 3.3 Forward transfer

forward transfer는 이전 task의 학습이 새로운 task의 학습 속도에 미친 영향을 나타낸다. reference는 동일한 architecture, algorithm, dataset, update budget으로 그 task를 처음부터 학습한 single-task learning curve다.

task $i$를 학습하는 구간의 진행률을 $u\in[0,1]$로 두고 sequential learning과 single-task learning의 score curve를 각각 $S_i^{seq}(u)$, $S_i^{single}(u)$라 하면

$$
FT_i=\int_0^1 S_i^{seq}(u)\,du-\int_0^1 S_i^{single}(u)\,du
$$

이다. 양수면 이전 task의 학습이 새 task의 학습을 빠르게 했고, 음수면 방해했다.

실제로는 학습 구간을 10등분하여 `0%, 10%, ..., 100%`의 11개 checkpoint에서 평가하고 사다리꼴 공식으로 적분을 근사한다. 두 방식은 동일한 checkpoint, evaluation seed, episode 수를 사용한다.

`0%` checkpoint도 포함한다. sequential learning의 `0%`는 이전 task를 학습한 policy의 zero-shot 성능이고 single-task learning의 `0%`는 초기화 직후의 성능이므로, 이 지점의 차이가 곧 zero-shot transfer다.

원 정의 [@wolczyk2021continualworld]는 남은 여유로 나눈 $(AUC^{seq}-AUC^{single})/(1-AUC^{single})$이지만, 본 연구의 score는 `100`을 넘을 수 있어 분모가 음수가 되는 경우가 생긴다. 따라서 차이만 사용한다.
