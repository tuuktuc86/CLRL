# Experiment Guideline

이 문서는 연구 전체의 실험 방향과 비교 원칙을 기록한다. Round 1의 model, 학습 budget, seed, hyperparameter, metric 계산과 baseline별 구현 명세는 [Round 1 Experiments Guideline](./experiments_1/experiments1_guideline.md)에 둔다.

용어는 [Glossary](../overview/glossary.md)를 따르고, observation/action 구성과 robot별 특성은 [Dataset](./dataset.md)을 기준으로 한다.

## Round 1. Pairwise Cross-Embodiment Continual Offline RL Baseline

### 목적

Round 1의 목적은 topology-aware continual learning 방법을 설계하기 전에, 기존 방법들이 서로 다른 두 embodiment를 순차적으로 학습할 때 보이는 다음 범위를 확보하는 것이다.

1. 새로운 embodiment를 학습한 뒤 이전 embodiment의 control capability가 얼마나 유지되는가?
2. 이전 embodiment의 학습 결과가 새로운 embodiment의 학습에 어떤 영향을 주는가?
3. 기존 continual learning mechanism이 retention을 높이는 과정에서 target acquisition을 얼마나 보존하는가?
4. Morphology 차이가 서로 다른 전환에서 transfer와 forgetting 양상이 어떻게 달라지는가?

Round 1은 새로운 방법의 우수성을 주장하는 단계가 아니다. 이후 제안 방법이 넘어야 할 baseline 범위와 공통 평가 protocol을 확정하는 단계다.

### Dataset과 robot 범위

모든 실험은 [Cross-Embodiment Offline RL Dataset](./dataset.md)의 `all_robots_replay_forward_1m` regime을 사용한다. 이 dataset에서 다음 네 robot을 선택한다.

| 역할 | Robot |
| --- | --- |
| 공통 source | `unitree_a1` |
| Target 1 | `unitree_go1` |
| Target 2 | `unitree_h1` |
| Target 3 | `hexapod` |

다음 세 pair를 독립적으로 실행한다.

```text
unitree_a1 -> unitree_go1
unitree_a1 -> unitree_h1
unitree_a1 -> hexapod
```

네 robot을 하나의 긴 sequence로 연결하지 않는다. Pairwise 실험을 통해 각 embodiment 전환의 target acquisition과 source retention을 먼저 분리해서 확인한다.

세 pair의 구조적 차이는 torso–joint–foot graph에서 계산한 canonical morphology distance로 함께 기록한다. 이 값은 pair 관계를 설명하는 보조 자료이며, Round 1에서는 세 pair만으로 거리와 transfer 사이의 통계적 상관관계를 주장하지 않는다.

### 비교 구조

비교군은 역할에 따라 구분한다.

| 구분 | Method | 역할 |
| --- | --- | --- |
| Single-task reference | Target-only IQL with URMA-style I/O | 각 robot의 독립 학습 성능과 forward-transfer 기준 |
| Backbone-matched | Sequential IQL with URMA-style I/O | 별도 continual mechanism이 없는 순차 fine-tuning 기준 |
| Backbone-matched | URMA-I/O IQL + actor-only EWC | 같은 IQL backbone에서 regularization의 효과 측정 |
| Cross-architecture reference | L2M (A1-pretrained adaptation) | Frozen backbone과 modulation을 사용하는 adaptation 기준 |
| Cross-architecture reference | VQ-CD | Heterogeneous state/action alignment를 사용하는 기준 |

Target-only IQL은 `unitree_a1`, `unitree_go1`, `unitree_h1`, `hexapod`에서 각각 실행한다. Target-only reference 없이 sequential target 성능을 forward transfer로 해석하지 않는다.

IQL/EWC의 URMA-style I/O는 joint와 foot에 공유되는 component encoder와 universal joint decoder를 사용하지만 explicit graph adjacency는 사용하지 않는다. PPO 기반 URMA 전체를 재현하는 것이 아니라 heterogeneous input/output dimension을 처리하기 위한 topology-agnostic backbone이다.

L2M과 VQ-CD는 IQL/EWC와 같은 backbone을 사용하지 않는다. 따라서 동일-backbone 비교와 분리하여 해석하고, architecture와 사전학습 차이를 계산 자원 및 auxiliary state와 함께 보고한다.

### 측정 대상

Target embodiment 학습이 시작된 뒤 각 평가 시점에서 다음 두 성능을 함께 측정한다.

1. Source Embodiment: `unitree_a1`의 유지 성능
2. Target Embodiment: 현재 target robot의 획득 성능

이를 통해 최종적으로 다음을 비교한다.

- Target 학습 전후의 A1 forgetting과 retention
- Target robot의 최종 성능
- Target-only reference 대비 sequential learning의 forward transfer
- Target 학습 중 A1 성능이 감소하는 양상
- Canonical morphology distance가 다른 pair에서 나타나는 acquisition–retention 차이

Robot별 raw return은 반드시 별도로 보고한다. Robot 사이의 reward scale 차이를 통제하는 normalized score는 보조 자료로만 사용하며 raw return과 raw-return 기반 비교를 대체하지 않는다.

### 실험 흐름

Round 1은 다음 흐름으로 진행한다.

1. Dataset interface, robot별 episode/reward 특성과 canonical morphology distance를 확인한다.
2. 네 robot의 target-only IQL reference를 확보한다.
3. A1 source checkpoint에서 세 target으로 분기하여 pairwise sequential learning을 실행한다.
4. Target 학습 과정에서 A1과 target을 같은 평가 protocol로 반복 측정한다.
5. Backbone-matched comparison과 cross-architecture reference를 구분하여 결과를 비교한다.
6. Morphology와 dataset difficulty를 함께 고려하여 pair별 차이를 해석한다.

### 공정성 원칙

- 같은 method와 seed에서 생성한 하나의 A1 source checkpoint를 세 target branch에 재사용한다.
- Target 학습 중 과거 A1 transition 또는 trajectory를 replay하지 않는다.
- Embodiment stage boundary와 evaluation embodiment는 알려진 것으로 가정한다.
- Robot별 mask와 adapter 선택에 필요한 embodiment identity는 허용하되, 각 method가 실제로 사용한 정보를 기록한다.
- EWC Fisher, L2M modulator, VQ codebook과 mask 같은 persistent state는 허용하되 저장량을 보고한다.
- 같은 pair에서는 동일한 dataset regime, training seed와 evaluation seed를 사용한다.
- 서로 다른 architecture는 data exposure, parameter 수, auxiliary memory와 계산 시간을 함께 보고한다.
- Dataset quality 차이가 morphology 효과로 해석되지 않도록 robot별 dataset 통계와 single-task 성능을 함께 기록한다.

### Round 1 종료 시 필요한 산출물

1. 네 robot의 target-only IQL reference
2. 각 pair와 method의 source/target embodiment 평가 결과
3. A1 forgetting/retention, target acquisition과 forward-transfer 결과
4. 세 pair의 canonical morphology distance와 robot별 dataset difficulty 정보
5. Backbone-matched 결과와 cross-architecture reference를 구분한 비교
6. 각 방법의 data exposure, 계산 시간과 persistent state 사용량
7. 실패 원인이 구현, 학습 budget 또는 hyperparameter에 있는지에 대한 추가 tuning 보고

Round 1은 이후 topology-aware method를 동일한 protocol에서 비교할 수 있을 때 종료한다.
