---
title: Dataset
aliases:
  - 데이터셋
tags:
  - cross-embodiment
  - dataset
  - offline-reinforcement-learning
status: complete
bibliography: ../related_works/reference.bib
link-citations: true
---

# Dataset

> [!success] 1차 작성 완료
> 이 문서는 *Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets*에서 공개한 dataset 구성을 소개한다.
> dataset은 Hugging Face의 [`haruki-abe/cross_embodiment_offline_rl`](https://huggingface.co/datasets/haruki-abe/cross_embodiment_offline_rl)에서 확인할 수 있다.
> 이후 실험 구현이나 검증 과정에서 수정이 필요한 내용은 다시 보완한다.

## 1. Dataset Overview

서로 다른 morphology를 가진 여러 robot의 학습 data가 모여 있다.

전체 16종의 robot이 포함된다.

- quadruped 9종
- biped 6종
- hexapod 1종

robot당 1M transition으로 구성되어 있다.

이 dataset은 data quality에 따라 세 종류, command direction에 따라 두 종류로 구분한다. 따라서 전체 여섯 개의 dataset이 존재한다.

- `all_robots_expert_forward_1m`
- `all_robots_expert_backward_1m`
- `all_robots_replay_forward_1m`
- `all_robots_replay_backward_1m`
- `all_robots_suboptimal_70_forward_1m`
- `all_robots_suboptimal_70_backward_1m`

simulator는 MuJoCo 기반이다.

## 2. Dataset 구성

### 2.1 Dataset Quality

학습 data는 PPO policy로 수집되었으며, 각 종류에 따라 수집 방법이 다르다.

| quality | 수집 기준 | 1M transition 구성 | 특징 |
|---|---|---|---|
| expert | PPO를 convergence까지 학습한 최종 expert policy | 최종 policy를 rollout하여 수집 | 가장 높은 behavior quality |
| expert replay | PPO가 최종 성능의 90%에 처음 도달하기 이전의 training data | candidate set에서 일정 간격으로 sampling | PPO 학습 과정의 여러 policy quality를 포함 |
| 70% suboptimal replay | early PPO training data와 late PPO training data | early 70% + late 30% | replay보다 suboptimal data 비율이 높음 |

따라서 dataset quality는 `expert > expert replay > 70% suboptimal replay` 순으로 볼 수 있다.

- **forward / backward:** 각 dataset은 walking command의 방향에 따라 forward와 backward로 나뉜다. forward는 robot이 정면 방향으로 `1 m/s`, backward는 반대 방향으로 `1 m/s`로 걷도록 command가 주어진다.

replay 계열 dataset에서는 index가 PPO 학습 진행 순서를 따른다. 앞쪽은 수집 policy의 학습 초기 구간이고 뒤쪽은 더 학습된 구간이므로, 앞에서 잘라 쓴 부분집합은 전체 dataset을 대표하지 않는다. 속도를 위해 앞부분만 읽으면 학습 초기 data에 편향될 수 있다.

### 2.2 Dataset 구성 파일

여섯 종류의 dataset은 다음 일곱 개 `.npy` array와 `metadata.json`으로 구성된다.

```text
observations.npy       (1000000, 16, 668)  float32
next_observations.npy  (1000000, 16, 668)  float32
actions.npy            (1000000, 16,  24)  float32
rewards.npy            (1000000, 16)       float32
dones.npy              (1000000, 16)       bool     # terminated OR truncated
terminateds.npy        (1000000, 16,   1)  float32  # terminal condition 충족 여부(예: robot이 넘어짐)
truncateds.npy         (1000000, 16,   1)  float32  # time limit에 따른 episode 종료 여부
metadata.json          {"n_samples": 1000000, "n_envs": 16}
```

- 0번 축: transition을 의미한다.
- 1번 축: 각 robot을 나타낸다. index는 다음과 같다.

| env | robot | morphology |
|---:|---| ---|
| 0 | `unitree_a1` | quadruped |
| 1 | `unitree_go1` | quadruped |
| 2 | `unitree_go2` | quadruped |
| 3 | `anymal_b` | quadruped |
| 4 | `anymal_c` | quadruped |
| 5 | `barkour_v0` | quadruped |
| 6 | `barkour_vb` | quadruped |
| 7 | `badger` | quadruped |
| 8 | `bittle` | quadruped |
| 9 | `unitree_h1` | biped |
| 10 | `unitree_g1` | biped |
| 11 | `talos` | biped |
| 12 | `robotis_op3` | biped |
| 13 | `nao_v5` | biped |
| 14 | `cassie` | biped |
| 15 | `hexapod` | hexapod |

- 2번 축: feature dimension을 의미한다. robot마다 observation이나 action dimension이 다르므로 최대 dimension을 기준으로 padding한다.

### 2.3 Observation Structure

robot의 native observation dimension은 다음 식을 따른다.

```text
native_obs_dim = 26 * number_of_joints + 12 * number_of_feet + 20
```

최댓값인 668을 기준으로 설정되며 남는 dimension은 `0`으로 padding된다.

#### 2.3.1 Joint Block 구성

각 joint block은 [[wiki#Morphology Descriptor|morphology descriptor]] 23차원과 dynamic state 3차원으로 구성된다.

| dims | content | 종류 |
|---|---|---|
| 0–2 | relative joint position in the trunk frame, normalized | 형상 |
| 3–5 | relative joint axis | 형상 |
| 6 | number of direct child joints | 위상 |
| 7 | nominal joint position | 기준 자세 |
| 8 | torque limit | 구동기 동역학 물성 |
| 9 | joint velocity limit | 구동기 동역학 물성 |
| 10 | damping | 구동기 동역학 물성 |
| 11 | armature | 구동기 동역학 물성 |
| 12 | stiffness | 구동기 동역학 물성 |
| 13 | friction loss | 구동기 동역학 물성 |
| 14–15 | joint range | 기구학 한계 |
| 16–18 | P gain, D gain, action scaling factor | 제어기 설정 |
| 19 | robot mass | robot 설계 |
| 20–22 | robot length, width, height | robot 설계 |
| 23 | joint position | robot state |
| 24 | joint velocity | robot state |
| 25 | previous action | robot state |

- 이 중 dynamic state는 23, 24, 25번에 해당한다.
- domain randomization으로 일부 descriptor도 변할 수 있지만, 매 transition의 robot state를 나타내는 값은 23, 24, 25번이다.

#### 2.3.2 Foot Block

각 foot block은 descriptor 10차원과 dynamic state 2차원으로 구성된다.

| dims | content | 종류 |
|---|---|---|
| 0–2 | relative foot position | 발 상대 위치 |
| 3–5 | P gain, D gain, action scaling factor | 제어기 설정 |
| 6 | robot mass | robot 설계 |
| 7–9 | robot length, width, height | robot 설계 |
| 10 | ground contact | robot state |
| 11 | time since last ground contact | robot state |

- 이 중 dynamic state는 10, 11번에 해당한다.
- domain randomization이 적용되어 일부 descriptor 값도 변할 수 있다.

#### 2.3.3 General Dynamic State와 Robot Context

robot observation의 마지막 20차원은 general dynamic state와 robot context로 구성된다. 이 중 13차원은 general dynamic state, 7차원은 robot context에 해당한다.

- general dynamic state

| dims | content |
|---|---|
| 0–2 | trunk 선속도 x, y, z |
| 3–5 | trunk 각속도 x, y, z |
| 6 | goal vx |
| 7 | goal vy |
| 8 | goal wz |
| 9–11 | 중력 x, y, z |
| 12 | 높이 |

- robot context

| dims | content |
|---|---|
| 0 | P gain |
| 1 | D gain |
| 2 | action scaling |
| 3 | mass |
| 4 | length |
| 5 | width |
| 6 | height |

#### 2.3.4 Robot별 Observation 차원 수

각 robot의 joint 수와 foot 수를 나타낸다. action dimension은 3절에서 추가로 설명한다.

| env | robot | joints | feet | native obs | native act | obs padding | act padding |
|---:|---|---:|---:|---:|---:|---:|---:|
| 0 | `unitree_a1` | 12 | 4 | 380 | 12 | 288 | 12 |
| 1 | `unitree_go1` | 12 | 4 | 380 | 12 | 288 | 12 |
| 2 | `unitree_go2` | 12 | 4 | 380 | 12 | 288 | 12 |
| 3 | `anymal_b` | 12 | 4 | 380 | 12 | 288 | 12 |
| 4 | `anymal_c` | 12 | 4 | 380 | 12 | 288 | 12 |
| 5 | `barkour_v0` | 12 | 4 | 380 | 12 | 288 | 12 |
| 6 | `barkour_vb` | 12 | 4 | 380 | 12 | 288 | 12 |
| 7 | `badger` | 13 | 4 | 406 | 13 | 262 | 11 |
| 8 | `bittle` | 8 | 4 | 276 | 8 | 392 | 16 |
| 9 | `unitree_h1` | 19 | 2 | 538 | 19 | 130 | 5 |
| 10 | `unitree_g1` | 23 | 2 | 642 | 23 | 26 | 1 |
| 11 | `talos` | 24 | 2 | 668 | 24 | 0 | 0 |
| 12 | `robotis_op3` | 20 | 2 | 564 | 20 | 104 | 4 |
| 13 | `nao_v5` | 22 | 2 | 616 | 22 | 52 | 2 |
| 14 | `cassie` | 10 | 2 | 304 | 10 | 364 | 14 |
| 15 | `hexapod` | 18 | 6 | 560 | 18 | 108 | 6 |

`talos`가 가장 넓은 observation/action interface를 가지며 padding width `668/24`를 결정한다.

#### 2.3.5 Scaling

observation은 저장 전에 이미 다음 scaling이 적용된 상태다. 이 scaling은 observation에만 적용된다. scaling은 두 형태다.

| 패턴 | 식 | 대상 | 효과 |
|---|---|---|---|
| A | `x / d` | 0을 중심으로 대칭인 값 | `[-d, +d]` → `[-1, +1]` |
| B | `x / (max/2) - 1` | 0 이상만 갖는 값 | `[0, max]` → `[-1, +1]`, 중앙값이 `0` |

| field | 원본 단위 | 변환 | clip | 의미 |
|---|---|---|:---:|---|
| joint position | rad | `/ 4.6` | | descriptor의 nominal position, joint range와 **같은 눈금** |
| joint velocity | rad/s | `/ 35` | | |
| previous action | action 단위 | `/ 10` | | |
| ground contact | `{0, 1}` | `/ 0.5 - 1` | | 안 닿음 `-1`, 닿음 `+1` |
| time since last contact | cycle | `/ 2.5 - 1` | O | `0 cycle` → `-1`, `5 cycle` → `+1`, 이후 포화 |
| trunk linear velocity | m/s | `/ 10` | O | 명령 `1 m/s`는 저장값 `0.1` |
| trunk angular velocity | rad/s | `/ 50` | O | |
| height | m | `/ robot_height - 1` | O | **nominal 높이가 `0`** |

`joint position`의 `/ 4.6`은 descriptor에도 동일하게 적용된다.

```text
environment.py:404   seen_joint_nominal_position / 4.6    descriptor 슬롯 7
environment.py:411   seen_joint_range            / 4.6    descriptor 슬롯 14-15
environment.py:583   joint_positions            /= 4.6    state 슬롯 23
```

셋이 같은 눈금이므로 policy는 observation만으로 `현재 각도 - nominal 각도`와 `가동 한계까지의 여유`를 직접 계산할 수 있다. descriptor와 state의 단위를 맞춘 것은 설계 의도다.

추가 normalization은 원본 dataset에 없던 처리다. 특히 per-robot z-normalization은 descriptor에 남아 있는 robot 사이의 scale 차이를 지우므로 적용하지 않는다.

#### 2.3.6 Observation 손상

본 dataset은 sim-to-real gap을 줄이기 위해 다음 세 가지 처리를 적용해 data를 수집했다.

- **dropout:** 매 step마다 joint별·foot별 독립 mask를 뽑아 dynamic field를 지운다. mask 하나가 해당 joint의 position, velocity와 previous action을 동시에 덮으며, foot도 contact와 time-since-contact가 함께 지워진다. 확률은 quadruped 9종이 `0.05`, 나머지 7종이 `0.001`이다.
- **noise:** scaling 전에 uniform noise를 더한다. quadruped 기준으로 joint position ±0.01, joint velocity ±1.5, trunk angular velocity ±0.2, projected gravity ±0.05다. `previous_action`과 foot field에는 noise가 없고 dropout만 적용된다.
- **action delay:** `max_nr_delay_steps=1`이므로 일부 step에서 `action[t]` 대신 `action[t-1]`이 적용되고, observation은 실제로 적용된 action을 기록한다.

mask는 저장되지 않는다. 지워진 값에는 scaling 전에 `0`이 기록되므로 정상 값과 구분할 수 없다.

## 3. Action Structure

action dimension은 robot의 joint 수와 같다. raw dataset의 `actions.npy`는 최대 action dimension인 24에 맞춰 zero-padding되어 있고, processed representation의 `actions.npy`는 robot별 native dimension `[N, J]`로 저장한다.

action은 joint가 도달해야 할 위치 자체가 아니라 nominal 자세로부터의 offset이다. environment는 `target_q = nominal_q + action × scaling_factor`로 목표 각도를 만들며, `unitree_a1`의 `scaling_factor`는 0.25다. 따라서 `action = 0`은 nominal 자세 유지를 뜻한다.

torque는 PD 제어로 얻는다. `torque = P × (target_q − q) − D × dq`이며, P 항은 목표와 현재 joint position의 차이를, D 항은 joint velocity를 사용한다. `unitree_a1`의 기본값은 `P = 20`, `D = 0.5`, 제어 주기 `50 Hz`다.

`nominal_q`, `scaling_factor`, `P`, `D`는 모두 joint descriptor에 들어 있어(slot 7, 18, 16, 17), policy가 자신의 출력이 torque로 변환되는 방식을 observation에서 읽을 수 있다. 이것이 gain이 다른 robot에 하나의 policy가 동작할 수 있는 근거다.

## 4. Reward Structure

dataset의 `rewards.npy`에는 robot policy를 학습할 때 사용한 reward가 그대로 저장되어 있다.

reward는 목표 속도를 따라가게 하는 tracking reward와 안정적인 보행을 위한 penalty로 구성된다. 전체 reward는 15개 term을 가중합한 뒤 0 이하의 값을 clipping한다.

$$
r_t=\max\left(0,\ \underbrace{r_{\text{track}}}_{\text{2 terms}}+\ c_{\text{cur}}(t)\underbrace{\sum r_{\text{penalty}}}_{\text{13 terms}}\right)
$$

| term | name | curriculum 적용 |
|---|---|:---:|
| T1 | XY velocity tracking | |
| T2 | Yaw velocity tracking | |
| T3 | Z velocity penalty | O |
| T4 | Pitch/Roll velocity penalty | O |
| T5 | Pitch/Roll position penalty | O |
| T6 | Joint nominal difference penalty | O |
| T7 | Joint limit penalty | O |
| **T8** | **Joint velocity penalty** | O |
| T9 | Joint acceleration penalty | O |
| T10 | Joint torque penalty | O |
| T11 | Action rate penalty | O |
| T12 | Collision penalty | O |
| T13 | Walking height penalty | O |
| T14 | Air-time term | O |
| T15 | Symmetry term | O |

모든 robot은 동일한 reward term을 사용하지만 각 term의 coefficient $w_i$는 robot마다 다르다. 따라서 서로 다른 robot 사이에서 raw reward 또는 episode return의 크기를 직접 비교하는 것은 적절하지 않다.

forward와 backward는 동일한 reward 구조를 사용하며 tracking target의 방향만 다르다.

- forward: target x velocity = `+1 m/s`
- backward: target x velocity = `-1 m/s`

즉, 두 dataset은 reward function의 형태는 같고 commanded walking direction만 다르다.

## 5. Episode Statistics

episode는 최대 1,000 step이며 fall 또는 특정 body collision이 발생하면 일찍 종료된다.

| robot | episodes | mean len | min | max | terminated % | mean reward | mean return |
|---|---:|---:|---:|---:|---:|---:|---:|
| `unitree_a1` | 1,056 | 947.0 | 60 | 1000 | 15.5 | 0.0201 | 18.99 |
| `unitree_go1` | 1,045 | 956.9 | 86 | 1000 | 11.9 | 0.0241 | 23.03 |
| `unitree_go2` | 1,434 | 697.4 | 11 | 1000 | 47.9 | 0.0131 | 9.12 |
| `anymal_b` | 1,108 | 902.5 | 48 | 1000 | 17.9 | 0.0132 | 11.95 |
| `anymal_c` | 1,304 | 766.9 | 38 | 1000 | 38.8 | 0.0127 | 9.77 |
| `barkour_v0` | 1,077 | 928.5 | 42 | 1000 | 17.1 | 0.0135 | 12.51 |
| `barkour_vb` | 1,035 | 966.2 | 101 | 1000 | 8.2 | 0.0238 | 23.02 |
| `badger` | 1,038 | 963.4 | 62 | 1000 | 9.6 | 0.0130 | 12.54 |
| `bittle` | 1,036 | 965.3 | 42 | 1000 | 5.5 | 0.0360 | 34.70 |
| `unitree_h1` | 3,630 | 275.5 | 19 | 1000 | 79.6 | 0.0291 | 8.02 |
| `unitree_g1` | 3,555 | 281.3 | 19 | 1000 | 79.4 | 0.0278 | 7.82 |
| `talos` | 2,328 | 429.6 | 16 | 1000 | 64.5 | 0.0510 | 21.91 |
| `robotis_op3` | 1,383 | 723.1 | 30 | 1000 | 38.1 | 0.0654 | 47.28 |
| `nao_v5` | 3,700 | 270.3 | 19 | 1000 | 83.9 | 0.0486 | 13.14 |
| `cassie` | 6,693 | 149.4 | 13 | 1000 | 93.7 | 0.0451 | 6.73 |
| `hexapod` | 1,002 | 998.0 | 14 | 1000 | 0.4 | 0.0443 | 44.18 |

## 6. Processed Data Format

공식 dataset은 모든 robot을 동일한 shape로 맞추기 위한 padding과 반복되는 descriptor 때문에 저장 공간을 많이 사용한다. data를 전처리해 robot별로 분리하고 반복되는 값을 압축하면 저장 용량과 loading 효율을 개선할 수 있다. 모든 robot을 한 번에 학습하지 않는 현재 실험에서는 다음 구조가 더 효율적이라고 판단한다. 해당 전처리로 용량을 약 82 GB에서 5.4 GB까지 줄인다.

processed representation은 robot별 directory와 memory-mapped `.npy` array를 사용한다.
파일 목록은 다음과 같다.

```text
processed/
  all_robots_replay_forward_1m/
    PREPROCESSING_REPORT.md
    manifest.json
    robot_mapping.json
    robot_defs.json
    inspection.json
    analysis.json
    roundtrip_validation.json
    robots/<robot>/
        metadata.json
        joint_state.npy              [N, J, 3]
        foot_state.npy               [N, F, 2]
        global_dynamic.npy           [N, 13]
        morphology_segments.npz
        actions.npy                  [N, J]
        rewards.npy                  [N]
        terminated.npy               [N] bool
        truncated.npy                [N] bool
        episode_id.npy               [N] int32
        episode_start_idx.npy        [M] int64
        episode_length.npy           [M] int64
        next_state_exception_idx.npy [X] int64
        next_state_exception_obs.npy [X, native_obs_dim]
        topology.npz
```

여기서 모든 robot에 대해 `N=1,000,000`이다.

- `dones`는 `terminateds | truncateds`로 복원할 수 있으므로 별도로 저장하지 않는다.
- `morphology_segments.npz`는 morphology descriptor가 바뀌는 지점만 저장한다. segment `k`는 `[segment_start[k], segment_start[k+1])` 범위를 나타낸다. 연속 step의 약 `99.6%`에서 descriptor가 유지되므로 1,000,000개의 vector 대신 약 2,700–4,600개의 segment만 저장한다.
- 대부분의 transition에서 `next_obs[t] == obs[t+1]`이므로 terminal, 마지막 step, 실제 mismatch row만 exception으로 저장한다. reference에서는 exact equality를 사용하여 bit-exact reconstruction을 검증했다.

- `topology.npz`는 trunk, joint, foot을 하나의 graph로 표현한다.

```python
topology["node_type"]           # 0 trunk, 1 joint, 2 foot
topology["parent_idx"]          # -1 for root
topology["joint_node_indices"]
topology["foot_node_indices"]
topology["joint_names"]
topology["foot_names"]
```

`parent_idx`는 MJCF kinematic tree를 따른다. flattened observation 자체에는 adjacency가 없으므로 connectivity가 필요하면 `topology.npz`를 사용해야 한다.
