---
title: Round 1 Baseline Reproduce
aliases:
  - Round 1 Baseline 재현
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - baseline-reproduction
status: draft
---

# Round 1 Baseline Reproduce
## 1. 공통 설정

### 1.1 Cross-embodiment 입출력 처리

IQL 기반 baseline은 모두 URMA-style shared component encoder를 사용한다 [@bohlinger2025urma]. Robot마다 joint 수와 foot 수가 다르므로 observation을 joint, foot과 general feature로 분리하고, 실제 component만 shared encoder에 입력한다. 뒤쪽 zero padding은 network 입력과 loss 계산에서 제외한다.

Actor는 active joint마다 Gaussian mean과 log standard deviation을 하나씩 출력한다. 따라서 network 출력은 `[B, J, 2]`이며, 저장 또는 environment interface에서만 최대 action dimension에 맞춰 padding한다.

### 1.2 학습 예산과 seed

| 항목 | 값 |
|---|---|
| Robot당 update | `100k` |
| Batch | `256` |
| 최종 seed | `43`, `44`, `45` |
| 개발 seed | `42` (configuration 선택에만 사용, 보고 표에서 제외) |

`100k`는 dataset 노출량과 학습 곡선의 포화 시점으로 정했다. `100k × 256 = 25.6M` transition draw이므로 `1M` transition dataset을 약 `25.6`회 본 것과 같은 노출량이다. 단독 학습 곡선이 robot에 따라 `10k`–`50k`에서 포화하므로 가장 늦은 robot의 두 배까지 학습하며, `10k` 간격의 checkpoint로 성능 변화 구간을 관측한다.

### 1.3 평가

평가 절차, normalized score의 정의와 기준점, continual RL metric은 [[metric]]에 정의한 것을 그대로 사용한다.

- 각 `100k` 학습 stage는 `0, 10k, ..., 100k`의 `11`개 checkpoint에서 평가한다.
- 각 checkpoint에서 environment reset seed `1000`–`1049`를 사용하여 robot당 `50` episode를 실행한다.
- 각 training seed의 성능은 위 `50` episode의 평균으로 계산한다.
- Final seed `43`, `44`, `45`는 서로 독립적으로 학습하고, 최종 결과는 training seed별 성능의 평균과 표준편차로 보고한다.
- 세 training seed의 evaluation episode를 하나의 표본으로 합치지 않는다.
- 모든 method와 checkpoint에서 같은 evaluation seed를 사용한다.
- Stochastic inference가 필요한 method의 sampling noise는 evaluation seed에서 결정적으로 생성하여 같은 seed의 재평가 결과가 재현되도록 한다.
- AUC와 forward transfer는 training seed마다 먼저 계산한 뒤 세 final seed의 평균과 표준편차를 구한다.
- Episode별 raw return은 저장하고, Markdown 결과표에는 training seed별 episode 평균을 기록한다.

### 1.4 Task

Round1에서 돌리는 task 로봇은 5종으로 각각은 다음과 같다.

A1, go1, barkour_vb, hexapod, h1
Fused gromov-Wasserstein [@vayer2019fgw]방법에 의하여 A1과의 차이는 다음으로 구성된다.

| Pair | 원본 FGW | 정규화 | 구조적 차이 |
|---|---:|---:|---|
| `a1 → go1` | `0.0222` | `0.000` | 16종 전체 최소 |
| `a1 → barkour_vb` | `0.0838` | `0.186` | 같은 topology, 다른 규모 |
| `a1 → hexapod` | `0.1858` | `0.495` | 다리 수가 바뀜 |
| `a1 → h1` | `0.2821` | `0.786` | 4족 → 2족 |

학습은 expert forward에서, replay forward에서, replay 70 forward에서 진행하고 성능을 평가한다.

학습 순서는 A1, go1, barkour_vb, hexapod, h1 이 순서대로 진행한다.

## 2. IQL 공통 재현

이 절은 IQL 기반 baseline이 공통으로 사용하는 network와 학습 설정을 정의한다. 각 continual learning mechanism의 적용 방법은 baseline별 절에서 별도로 정한다.

### 2.1 Hyperparameter

| Hyperparameter | 값 |
|---|---:|
| Learning rate | `3e-4` |
| Batch size | `256` |
| Discount factor ($\gamma$) | `0.99` |
| Target EMA coefficient | `0.005` |
| Expectile | `0.7` |
| Advantage temperature | `3.0` |
| Advantage weight clipping | `100` |
| Gradient norm clipping | `1.0` |
| Optimizer | `Adam` |

각 gradient update의 minibatch는 현재 task dataset의 전체 transition에서 uniform sampling with replacement로 구성한다.

### 2.2 Network 입력

`B`는 batch size, `J`는 robot의 active joint 수, `F`는 active foot 수다. Component latent는 `128`, policy/Q/V core는 hidden width `512`, hidden depth `4`로 고정한다.

| Feature | 저장 차원 | Encoder 입력 | Batch shape |
|---|---:|---:|---|
| Joint descriptor | `23` | `16` | `[B, J, 16]` |
| Joint state | `3` | `3` | `[B, J, 3]` |
| Foot descriptor | `10` | `3` | `[B, F, 3]` |
| Foot state | `2` | `2` | `[B, F, 2]` |
| Global dynamic | `13` | `13` | `[B, 13]` |
| Robot context | `7` | `7` | `[B, 7]` |

Joint descriptor와 foot descriptor의 마지막 `7`차원은 robot context와 중복되므로 component encoder에서는 제외한다. Global dynamic과 robot context를 합친 general feature는 `[B, 20]`이다.

학습 전에 robot별 action 범위와 environment의 action clipping 및 scaling 방식을 확인해야 한다.

### 2.3 URMA-style component encoder

Joint descriptor와 joint state는 각각 모든 joint가 parameter를 공유하는 MLP로 encoding한다.

```text
joint_desc  [B, J, 16] ── MLP 16 → 128 → 128 ──→ jd [B, J, 128]
joint_state [B, J,  3] ── MLP  3 → 128 → 128 ──→ js [B, J, 128]

jd ── Linear 128 → 128 ── softmax over J ──→ w [B, J, 128]
joint_latent = sum_J(w * js)                  → [B, 128]
```

Foot feature도 같은 방식으로 처리한다.

```text
foot_desc  [B, F, 3] ── MLP 3 → 128 → 128 ──→ fd [B, F, 128]
foot_state [B, F, 2] ── MLP 2 → 128 → 128 ──→ fs [B, F, 128]

fd ── Linear 128 → 128 ── softmax over F ──→ w [B, F, 128]
foot_latent = sum_F(w * fs)                  → [B, 128]
```

Batch padding을 사용하는 경우 padding component는 softmax 전에 mask한다. 두 component latent와 general feature를 concatenate하면 robot 종류와 관계없이 같은 크기의 representation을 얻는다.

```text
joint_latent [B, 128]
foot_latent  [B, 128]
general      [B,  20]
          concatenate
               ↓
representation [B, 276]
```

모든 MLP는 첫 hidden layer 뒤에 LayerNorm을 적용하고 hidden activation으로 ReLU를 사용한다. 따라서 component MLP는 `Linear → LayerNorm → ReLU → Linear → ReLU`, actor decoder는 `Linear → LayerNorm → ReLU → Linear` 순서다. Core의 첫 번째 `512`차원 hidden layer에도 같은 LayerNorm을 적용한다.

Linear weight는 orthogonal initialization을 사용한다. Hidden layer와 Q/V 및 actor mean 출력의 gain은 $\sqrt{2}$, actor log standard deviation 출력의 gain은 `1e-3`으로 두고 bias는 `0`으로 초기화한다. LayerNorm의 scale과 bias는 각각 `1`과 `0`으로 초기화한다.

### 2.4 Actor, Q와 V

Actor, twin Q와 V는 같은 component encoder와 core 설계를 사용하지만 parameter를 공유하지 않는다. 각 core에는 `512`차원 hidden layer를 네 개 사용한다.

#### V-network

V-network는 state representation에서 scalar value를 출력한다.

```text
representation [B, 276]
  ── core 276 → 512 → 512 → 512 → 512
  ── Linear 512 → 1
  ── V(s) [B]
```

#### Q-network

Q-network는 현재 action scalar를 각 joint state에 concatenate한다. 따라서 joint state encoder의 입력은 `[B, J, 4]`이고, 나머지 encoding과 pooling은 V-network와 같다. Q1과 Q2는 서로 독립된 parameter를 사용한다.

```text
concat(joint_state, action) [B, J, 4]
  ── URMA-style encoder
  ── representation [B, 276]
  ── core 276 → 512 → 512 → 512 → 512
  ── Linear 512 → 1
  ── Q(s, a) [B]
```

Target Q1과 Target Q2는 각각 online Q-network에서 초기화하고 다음 식으로 갱신한다.

$$
\theta_{target}\leftarrow0.995\theta_{target}+0.005\theta_{online}.
$$

Target network는 optimizer로 직접 학습하지 않는다.

#### Actor

Actor core가 만든 global latent를 active joint 수만큼 복제한 뒤, 각 joint의 descriptor latent와 state latent를 concatenate한다. 모든 joint는 같은 decoder parameter를 사용한다.

```text
representation [B, 276]
  ── core 276 → 512 → 512 → 512 → 512
  ── h [B, 512]

repeat(h, J) [B, J, 512]
jd           [B, J, 128]
js           [B, J, 128]
       concatenate
            ↓
joint feature [B, J, 768]
  ── shared decoder 768 → 128 → 2
  ── mean, log standard deviation [B, J, 2]
```

Actor는 joint별 독립 Gaussian distribution을 사용하며 mean에는 `tanh`를 적용한다. 학습에는 dataset action의 likelihood를 사용하고, 평가에는 deterministic mean action을 environment action 단위로 역변환하여 사용한다.

### 2.5 IQL 학습

학습식의 reward $r$은 dataset에 저장된 raw reward를 사용한다. 평가는 [[metric]]에 정의한 tracking-only reward를 사용한다.

Target Q의 보수적인 추정값과 V의 차이를 다음과 같이 정의한다.

$$
q_{target}(s,a)=\min\left(Q_{1,target}(s,a),Q_{2,target}(s,a)\right),
\qquad
\delta=q_{target}(s,a)-V(s).
$$

V-network는 expectile regression으로 학습한다.

$$
L_V
=
\mathbb E\left[
\left|\tau-\mathbb 1(\delta<0)\right|\delta^2
\right],
\qquad \tau=0.7.
$$

Q target은 다음 state의 V를 사용한다. 실제 termination에서는 bootstrap을 중단하고 time-limit truncation에서는 유지한다.

$$
y=r+\gamma(1-\mathrm{terminated})V(s'),
$$

$$
L_Q
=
\mathbb E\left[(Q_1(s,a)-y)^2+(Q_2(s,a)-y)^2\right].
$$

Actor는 advantage-weighted behavior cloning으로 학습한다.

$$
A(s,a)=q_{target}(s,a)-V(s),
$$

$$
w(s,a)=\min\left(\exp(3.0A(s,a)),100\right),
$$

$$
L_{actor}
=
-\mathbb E\left[w(s,a)\log\pi(a\mid s)\right].
$$

Joint별 Gaussian log probability는 active joint에 대해 합산한다.

$$
\log\pi(a\mid s)
=
\sum_{j=1}^{J}\log\pi_j(a_j\mid s).
$$

한 gradient update에서 V, Q1/Q2와 actor를 각각 한 번 갱신한다. Loss 사이로 gradient가 전달되지 않도록 $q_{target}$, $y$, $A$와 $w$는 해당 loss에서 detach한다. Online Q를 갱신한 뒤 Target Q를 EMA로 갱신한다.

### 2.6 Shape와 parameter 검증

| Robot | J | F | Joint descriptor | Joint state | Foot descriptor | Foot state | Representation | Actor output |
|---|---:|---:|---|---|---|---|---|---|
| `unitree_a1` | `12` | `4` | `[B,12,16]` | `[B,12,3]` | `[B,4,3]` | `[B,4,2]` | `[B,276]` | `[B,12,2]` |
| `unitree_go1` | `12` | `4` | `[B,12,16]` | `[B,12,3]` | `[B,4,3]` | `[B,4,2]` | `[B,276]` | `[B,12,2]` |
| `barkour_vb` | `12` | `4` | `[B,12,16]` | `[B,12,3]` | `[B,4,3]` | `[B,4,2]` | `[B,276]` | `[B,12,2]` |
| `hexapod` | `18` | `6` | `[B,18,16]` | `[B,18,3]` | `[B,6,3]` | `[B,6,2]` | `[B,276]` | `[B,18,2]` |
| `unitree_h1` | `19` | `2` | `[B,19,16]` | `[B,19,3]` | `[B,2,3]` | `[B,2,2]` | `[B,276]` | `[B,19,2]` |

구현 후 다음 parameter 수와 일치하는지 확인한다.

| Network | Trainable parameters |
|---|---:|
| Actor | `1,133,442` |
| Q1 | `1,035,137` |
| Q2 | `1,035,137` |
| V | `1,035,009` |
| **합계** | **`4,238,725`** |

Target Q1과 Target Q2는 위 합계에 포함하지 않는다.

## 3. Baseline별 재현

### 3.1 Single-task IQL

Single-task IQL은 2절에서 정의한 IQL을 사용하여 각 task의 성능을 독립적으로 측정한다. 하나의 task는 dataset과 robot의 조합이며, 예를 들어 `expert–A1`, `replay–Go1`을 각각 따로 학습한다. 이를 통해 각 task에서 IQL로 달성할 수 있는 최고 성능을 조사한다.


### 3.2 Sequential IQL

Sequential IQL은 각 task를 순차적으로 학습하는 시나리오를 말한다. 각 task를 학습할 때마다 metric에 필요한 지표를 측정한다. results와 notes를 참고하여 해당 부분에 숫자를 채워라.

### 3.3 IQL + EWC

IQL + EWC는 2절의 IQL actor 전체에 EWC를 적용한다. Actor의 component encoder, core와 joint decoder를 보호하며 Q1, Q2와 V에는 EWC를 적용하지 않는다.

각 task의 학습이 끝나면 actor parameter snapshot $\theta_t^*$를 저장하고, 해당 task의 dataset에서 `10,240`개 transition을 uniform sampling하여 diagonal policy Fisher $F_t$를 계산한다. State $s$는 dataset에서 가져오고 action $\tilde a$는 학습이 끝난 actor에서 sampling한 뒤 detach한다.

$$
F_{t,j}
=
\frac{1}{N}\sum_{n=1}^{N}
\left(
\frac{\partial\log\pi_{\theta_t}(\tilde a_n\mid s_n)}
{\partial\theta_{t,j}}
\right)^2,
\qquad
\tilde a_n\sim\pi_{\theta_t}(\cdot\mid s_n).
$$

Log probability는 2.5절과 같이 active joint에 대해 합산한다. Fisher는 transition별 log probability gradient를 제곱한 뒤 평균하며 IQL advantage weight는 사용하지 않는다. 마지막 task는 이후 학습에 사용할 Fisher가 필요하지 않으므로 계산을 생략한다.

Task $t$의 actor loss에는 이전 task별 Fisher와 snapshot에 대한 penalty를 합산한다.

$$
L_{actor}^{EWC}
=
L_{actor}^{IQL}
+
\frac{\lambda}{2}
\sum_{k<t}\sum_j
F_{k,j}(\theta_j-\theta_{k,j}^*)^2.
$$

Expert, replay와 suboptimal70은 독립된 실험으로 실행하며 EWC state를 서로 공유하지 않는다. Training seed마다 snapshot과 Fisher를 별도로 생성한다.

$\lambda=1.0$으로 고정하여 모든 dataset, task와 training seed에 동일하게 사용한다. EWC penalty가 actor update를 거의 제한하지 못하거나 새로운 task 학습을 막는 경우에만 추가 tuning을 진행한다.

각 run에는 Fisher의 평균·최댓값·nonzero 비율, EWC penalty와 기본 actor loss의 비율, snapshot/Fisher 저장 용량과 Fisher 계산 시간을 기록한다.

### 3.4 IQL + ER

IQL + ER은 이전 task의 transition 일부를 replay buffer에 저장하고, 현재 task transition과 함께 2절의 IQL loss를 계산한다. 첫 번째 task에는 과거 transition이 없으므로 Sequential IQL과 동일하게 학습한다.

Replay buffer의 전체 크기는 `50,000` transition으로 고정한다. Task $t$가 끝나면 완료된 $t$개 task에 buffer를 균등하게 할당한다. 기존 task partition은 새 할당량에 맞게 uniform sampling without replacement로 축소하고, 새로 완료된 task dataset에서도 같은 수의 transition을 uniform sampling without replacement로 저장한다.

Task $t>1$의 minibatch는 다음과 같이 구성한다.

| 구성 | Transition 수 |
|---|---:|
| 현재 task | `128` |
| Replay buffer | `128` |
| 전체 batch | `256` |

현재 task transition은 전체 dataset에서 uniform sampling with replacement로 가져온다. Replay transition은 먼저 과거 task를 uniform하게 선택하고 해당 task partition에서 uniform sampling with replacement로 가져온다. Actor, Q1/Q2와 V의 IQL loss는 모두 current/replay가 혼합된 batch에서 계산하며 별도의 behavioral cloning이나 value cloning loss는 추가하지 않는다.

Replay buffer에는 observation, action, reward, next observation, terminated, truncated, robot identity와 component mask를 저장한다. Expert, replay와 suboptimal70은 독립된 실험으로 실행하며 replay buffer를 서로 공유하지 않는다. Training seed마다 buffer sample을 별도로 생성한다.

각 run에는 task별 저장 transition 수, current/replay에서 sampling한 transition 수, replay buffer의 실제 byte 크기와 replay loading 시간을 기록한다.

### 3.5 IQL + PackNet

IQL + PackNet은 2절의 IQL actor에 PackNet의 iterative pruning과 task별 parameter mask를 적용한다. Q1, Q2와 V에는 PackNet을 적용하지 않는다.

첫 번째 task에서는 actor 전체를 학습한다. 이후 각 non-final task의 학습 과정은 다음과 같다.

1. 현재 사용할 수 있는 free weight로 task를 `100k` update 학습한다.
2. 현재 task에서 새로 학습된 Linear weight를 layer별 absolute magnitude 기준으로 정렬한다.
3. 크기가 작은 weight `75%`를 one-shot pruning하여 `0`으로 만들고 다음 task를 위한 free weight로 반환한다.
4. 남은 weight만 `25k` update 동안 추가 학습하여 pruning으로 감소한 성능을 회복한다.
5. 회복 학습이 끝난 weight를 현재 task 소유로 지정하고 이후 task에서 고정한다.

Recovery update `25k`는 공통 `100k` budget에 포함하지 않고 PackNet의 추가 계산량으로 별도 기록한다. 마지막 task에서는 이후 사용할 free weight가 필요하지 않으므로 pruning과 recovery를 수행하지 않는다.

이전 task에 할당된 weight는 이후 task 학습에서도 forward computation에 사용하지만 gradient update는 허용하지 않는다. 새 task에서는 아직 할당되지 않은 free weight만 학습한다. Task가 바뀔 때 actor optimizer를 다시 생성하여 frozen weight에 남아 있는 Adam state가 parameter를 변경하지 않도록 한다.

Linear weight만 pruning한다. Bias와 LayerNorm parameter는 첫 번째 task의 pruning 및 recovery가 끝난 뒤 고정하며 이후 task에서 갱신하지 않는다.

각 weight에는 처음 할당된 task index를 저장한다. Task $k$를 평가할 때는 task $k$까지 할당된 weight만 활성화하고 이후 task에서 학습한 weight는 mask한다. 따라서 evaluation에는 task identity가 필요하다.

Expert, replay와 suboptimal70은 독립된 실험으로 실행하며 parameter mask와 ownership state를 서로 공유하지 않는다. Training seed마다 pruning 결과와 mask를 별도로 생성한다.

각 run에는 task별 active/frozen/free parameter 수, mask 저장 용량, pruning 직전·직후·recovery 이후 성능과 recovery update 수를 기록한다.

### 3.6 IQL + EG

IQL + EG는 실험에 포함된 다섯 robot의 dataset에 동시에 접근하여 2절의 IQL을 하나의 shared model로 학습한다. Expert, replay와 suboptimal70은 서로 섞지 않고 각각 별도의 run으로 실행한다. 순차적 dataset 접근을 사용하는 continual learning baseline이 아니라 full-access morphology-aware reference로 사용한다 [@abe2026cross].

#### Morphology grouping

공식 EG 구현에서 제공하는 16종 robot의 `embodiment_distance.npy`를 사용한다. 이 matrix에 Ward hierarchical clustering을 적용하고 dendrogram을 `M=2`에서 잘라 group을 고정한다.

Round 1의 다섯 robot은 16종 기준의 assignment에서 다음 두 group만 가져온다.

| Group | Robot |
|---|---|
| Group 1 | `unitree_a1`, `unitree_go1`, `barkour_vb`, `hexapod` |
| Group 2 | `unitree_h1` |

다섯 robot만으로 distance나 clustering을 다시 계산하지 않으며 return에 맞추어 group을 조정하지 않는다. 실행 전에 사용한 distance matrix와 `robot -> group` mapping을 artifact로 저장한다.

#### 학습 방법

Actor, Q1, Q2와 V는 robot 또는 group별로 복제하지 않고 모든 robot이 하나의 parameter를 공유한다. 각 outer update는 다음 순서로 수행한다.

1. 다섯 robot에서 각각 `256`개 transition을 uniform sampling하여 총 `1,280`개의 global batch를 구성한다.
2. Global batch 전체로 V를 한 번, Q1과 Q2를 한 번 update하고 target Q에 EMA를 적용한다.
3. 현재 batch에 포함된 morphology group의 순서를 무작위로 섞는다.
4. Group 1의 `1,024`개 transition과 Group 2의 `256`개 transition으로 shared actor를 group마다 한 번 update한다.

Actor loss와 IQL advantage weight는 2절의 공통 설정을 그대로 사용한다. 같은 group에 robot이 여러 개 있으면 해당 robot의 transition을 하나의 actor sub-batch로 합친다. Round 1 robot이 포함되지 않은 group은 건너뛴다.

각 dataset regime에서 `100k` outer update를 수행한다. 따라서 robot별 transition draw는 `100k × 256 = 25.6M`이며, global V/Q optimizer step은 각각 `100k`다. 두 group을 각각 update하므로 actor optimizer step은 총 `200k`다.

각 run에는 16종 FGW matrix와 group mapping의 version, robot별 transition draw, global V/Q update 수, actor update 수, group별 actor sub-batch 크기, parameter 수와 wall-clock time을 기록한다. IQL + EG는 모든 task dataset을 동시에 사용하므로 forgetting과 forward transfer는 계산하지 않고, checkpoint별 robot 성능과 최종 average performance를 보고한다.

### 3.7 VQ-CD [@hu2025vqcd]

VQ-CD는 IQL을 사용하지 않는다. Robot마다 다른 observation/action space를 QSA(Quantized Spaces Alignment)로 고정된 latent space에 정렬한 뒤, SWA(Selective Weights Activation) diffusion model을 task별 sparse mask로 학습한다.

#### Data 구성

Expert, replay와 suboptimal70은 서로 섞지 않고 각각 별도의 continual run으로 실행한다. 각 task에서는 뒤쪽 zero padding을 제거한 robot-native observation과 active action만 사용한다. Observation은 dataset에 저장된 scaling을 유지하고, action은 2.2절의 범위 확인 후 `[-1, 1]`로 변환한다.

길이 `8`의 연속 state-action sequence를 episode boundary를 넘지 않도록 uniform sampling한다. Return condition은 dataset의 raw reward와 $\gamma=0.99$로 계산한 discounted return을 task별 최솟값과 최댓값으로 `[0, 1]`에 정규화한다. 각 task에는 고정된 `1,024`차원 random task embedding을 하나 부여하고 training seed와 함께 저장한다.

#### QSA

State와 action에는 서로 독립된 QSA를 사용한다. 각 task는 자신의 encoder, decoder와 codebook 구간을 가지며, 이미 학습한 task의 QSA parameter는 이후 task에서 고정한다.

| 대상 | Encoder | Aligned feature | Decoder | Codebook |
|---|---|---:|---|---|
| State | $d_s^i \rightarrow 256 \rightarrow 256 \rightarrow 256 \rightarrow 20$ | `10 × 2` | $20 \rightarrow 128 \rightarrow 128 \rightarrow 128 \rightarrow d_s^i$ | task당 `512 × 2` |
| Action | $d_a^i \rightarrow 256 \rightarrow 256 \rightarrow 256 \rightarrow 10$ | `5 × 2` | $10 \rightarrow 128 \rightarrow 128 \rightarrow 128 \rightarrow d_a^i$ | task당 `512 × 2` |

MLP activation은 LeakyReLU를 사용한다. 다섯 task의 codebook은 하나의 tensor 안에 task당 `512`개씩 배타적인 구간을 할당하므로 state/action codebook의 전체 크기는 각각 `2,560 × 2`다.

State와 action에 각각 다음 loss를 적용한다.

$$
L_{QSA}(x)
=
\lVert x-D_i(z_q)\rVert_2^2
+\lVert \operatorname{sg}(z_e)-z_q\rVert_2^2
+0.25\lVert z_e-\operatorname{sg}(z_q)\rVert_2^2.
$$

각 codebook embedding은 update 후 `[-3, 3]`으로 clipping한다. Task가 도착하면 현재 task dataset만으로 QSA를 먼저 학습하고, QSA를 고정한 뒤 같은 task의 SWA 학습을 시작한다. 따라서 미래 task dataset을 QSA 사전학습에 미리 사용하지 않는다.

#### SWA diffusion model

SWA는 aligned state sequence `[B, 8, 20]`의 noise를 예측하는 temporal U-Net이다. U-Net의 base dimension은 `256`, channel multiplier는 `(2, 2, 4)`로 둔다. 별도의 task-specific inverse dynamics MLP가 연속한 두 state latent를 action latent로 변환한다.

```text
aligned state pair [B, 40]
  ── inverse dynamics 40 → 256 → 256 → 10
  ── aligned action [B, 10]
  ── task action decoder 10 → native action dimension
```

Diffusion loss와 inverse dynamics loss를 함께 최적화한다.

$$
L_{VQ-CD}
=
\lVert \epsilon-\epsilon_\theta(z_s^k,k,R,i)\rVert_2^2
+\lVert z_a-\Psi_i(z_{s,t},z_{s,t+1})\rVert_2^2.
$$

다섯 task를 알고 있으므로 mask rate는 $1/I=0.2$로 고정한다. 각 task mask는 U-Net의 masked one-dimensional convolution weight 전체 위치 중 `20%`를 아직 할당되지 않은 위치에서 무작위로 선택한다. Task 사이 mask는 겹치지 않으며 mask 생성 결과는 training seed별로 저장한다.

각 task의 SWA 학습은 이전 task checkpoint가 아니라 동일한 초기 U-Net checkpoint에서 시작한다. 현재 task mask를 적용하여 현재 task sequence만 학습하고, 종료 checkpoint에서 해당 mask가 선택한 weight를 추출한다. Task $t$까지 학습한 model은 다음과 같이 조립한다.

$$
W^{(t)}=\sum_{i=1}^{t}M_i\odot W_i.
$$

Task-specific QSA, return-conditioning module과 inverse dynamics model은 task identity로 선택한다. 이전 task transition이나 latent sequence는 이후 task 학습에 사용하지 않는다.

#### Hyperparameter와 budget

| 항목 | 값 |
|---|---:|
| QSA learning rate | `1e-3 → 1e-4` cosine decay |
| QSA update | task당 `100k` |
| SWA learning rate | `3e-4` |
| SWA update | task당 `100k` |
| Batch / sequence length | `32 / 8` |
| Optimizer | `Adam` |
| Diffusion step | `200` |
| Condition dropout | `0.25` |
| EMA decay | `0.995` |
| Guidance value / weight | `0.95 / 1.2` |
| Evaluation sampler | `DDIM`, stride `20` (`10` reverse steps) |

QSA와 SWA는 각각 task당 `100k × 32 × 8 = 25.6M` transition token을 처리한다. 두 단계를 합친 task당 exposure는 `51.2M`이며 IQL의 transition draw와 별도로 기록한다. 공식 공개 configuration은 QSA와 SWA를 각각 `500k` update 학습하므로, 현재 `100k`는 Round 1용 축소 budget이다. Reconstruction이나 SWA learning curve가 수렴하지 않으면 정상적인 method failure로 결론내리지 않고 추가 budget 필요로 기록한다.

SWA는 `0, 10k, ..., 100k`에서 checkpoint를 저장한다. 각 checkpoint에서는 이전 task에서 추출한 weight와 현재 checkpoint의 masked weight를 임시로 조립하여 지금까지 학습한 모든 task를 평가한다. 이를 통해 forgetting matrix와 현재 task의 AUC curve를 기록한다.

#### Evaluation과 재현 확인

Evaluation에서는 현재 observation을 state QSA로 encoding하고 return condition `0.95`를 주어 latent state sequence를 생성한다. 첫 두 state latent를 inverse dynamics model에 넣고, 생성된 action latent를 현재 task의 action decoder로 복원한 뒤 robot action 범위로 변환한다.

VQ-CD의 forward transfer를 계산하려면 같은 QSA, mask rate, 초기화와 `100k` budget을 사용하는 VQ-CD task-from-scratch curve가 별도로 필요하다. 이 reference가 없으면 VQ-CD의 target AUC는 학습 속도로만 보고하고 forward transfer로 표기하지 않는다.

각 run에는 state/action reconstruction loss, codebook utilization과 dead-code 비율, diffusion/inverse-dynamics loss, task별 mask와 checkpoint 크기, QSA·SWA parameter 수, transition token, peak GPU memory와 wall-clock time을 기록한다. QSA reconstruction이 유한하지 않거나 decoded action이 NaN 또는 action 범위를 벗어나거나 SWA policy 평가를 완료하지 못하면 결과표에 정상 baseline으로 넣지 않고 `VQ-CD pending reproduction`으로 기록한다.

## 4. 실행 계획

### 4.1 사전 검증

최종 실험 전에 다음 공통 항목을 확인한다.

- 다섯 robot의 observation/action dimension과 component mask
- Dataset에 저장된 action 범위
- Sequence가 episode boundary를 넘지 않는지 여부
- Tracking-only reward를 사용하는 evaluation 환경
- Actor, Q와 V의 입출력 shape와 loss의 NaN 여부
- Expert, replay와 suboptimal70 사이의 dataset, checkpoint와 persistent state 분리

### 4.2 개발 실험

개발 seed `42`는 구현 검증과 configuration 선택에만 사용하며 최종 결과에는 포함하지 않는다. 다음 순서로 점검한다.

1. Single-task IQL로 공통 IQL 구현을 검증한다.
2. Sequential IQL로 task 전달과 evaluation matrix 생성을 검증한다.
3. IQL + EWC, IQL + ER과 IQL + PackNet의 추가 mechanism을 검증한다.
4. IQL + EG의 두 morphology group 학습을 검증한다.
5. VQ-CD는 QSA reconstruction을 먼저 확인한 뒤 SWA 학습과 조립된 model의 evaluation을 검증한다.

이 단계에서는 최종 성능을 보고하지 않는다. 각 방법이 정상적으로 학습·평가되는지 확인하고, 실패한 구현과 추가 tuning이 필요한 configuration을 구분한다.

### 4.3 최종 실험

Configuration을 고정한 뒤 final seed `43`, `44`, `45`를 각각 실행한다. Baseline은 다음 순서로 진행한다.

1. Single-task IQL
2. Sequential IQL
3. IQL + EWC
4. IQL + ER
5. IQL + PackNet
6. IQL + EG
7. VQ-CD

Single-task IQL은 normalized score와 IQL 계열 forward transfer의 reference로 먼저 확보한다. Sequential IQL은 나머지 IQL 기반 continual learning 방법의 기본 비교군으로 사용한다.

각 방법은 expert, replay와 suboptimal70에서 별도의 run으로 실행한다. 세 dataset regime 사이에는 model checkpoint, optimizer, replay buffer, Fisher, parameter mask, QSA와 SWA state를 공유하지 않는다.

### 4.4 평가와 완료 조건

- 매 `10k` update마다 현재까지 학습한 모든 task를 평가한다.
- 각 training seed의 evaluation matrix와 learning curve를 개별적으로 저장한다.
- Seed별 결과를 먼저 기록한 뒤 세 final seed의 평균과 표준편차를 계산한다.
- IQL 기반 continual learning 방법은 Single-task IQL curve와 비교하여 forward-transfer AUC를 계산한다.
- IQL + EG는 sequential metric을 계산하지 않고 checkpoint별 robot 성능과 최종 average performance를 보고한다.
- VQ-CD는 같은 architecture의 task-from-scratch reference가 있을 때만 forward transfer를 계산한다.
- VQ-CD의 QSA 또는 SWA가 재현 조건을 통과하지 못하면 `VQ-CD pending reproduction`으로 기록한다.
- 마지막 checkpoint에서도 learning curve가 계속 상승하면 추가 budget 필요로 기록한다.

모든 final seed에서 필요한 checkpoint와 평가 결과가 생성되고, 실패 또는 미수렴 run의 상태가 구분되어야 해당 baseline의 Round 1 실행이 완료된 것으로 본다.
