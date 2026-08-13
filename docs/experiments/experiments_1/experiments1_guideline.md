# Round 1 Experiments Guideline

이 문서는 서버에서 Round 1 실험을 구현하고 실행할 수 있도록 필요한 설정을 정의한다. 직접적인 code structure나 framework는 강제하지 않으며, data contract, model behavior, budget, evaluation과 logging contract를 고정한다.

관련 문서:

- [Experiment Guideline](../guideline.md)
- [Dataset](../dataset.md)
- [Baseline 구현 근거](../../related_works_research/round1_baseline_implementation_sources.md)
- [Heterogeneous observation/action dimension 처리 조사](../../related_works_research/heterogeneous_dimensions_vqcd_l2m_urma.md)
- [Round 1 protocol 근거](../../related_works_research/round1_protocol_evidence.md)
- [Glossary](../../overview/glossary.md)

## 1. Round 1의 질문과 범위

Round 1은 다음 세 전환을 독립적으로 평가한다.

| Pair ID | Source Embodiment | Target Embodiment | 구조적 구분 |
| --- | --- | --- | --- |
| `a1_go1` | `unitree_a1` | `unitree_go1` | 유사한 quadruped morphology |
| `a1_h1` | `unitree_a1` | `unitree_h1` | quadruped에서 biped로 전환 |
| `a1_hexapod` | `unitree_a1` | `hexapod` | leg 수와 topology가 크게 달라지는 전환 |

각 pair에서 답할 질문은 세 가지다.

1. Target embodiment를 학습한 뒤 source embodiment의 control capability가 얼마나 유지되는가?
2. Source embodiment의 학습 결과를 이어받은 방법이 target embodiment를 얼마나 빠르고 충분하게 학습하는가?
3. Canonical morphology distance가 다른 세 전환에서 acquisition과 forgetting 양상이 어떻게 달라지는가?

세 pair는 별도 run이다. `A1 -> Go1 -> H1 -> Hexapod`과 같은 4-stage sequence는 Round 1 범위에서 제외한다.

Canonical morphology distance는 세 pair의 구조적 관계를 설명하는 보조 자료다. 세 pair는 통계적 상관관계를 검정하기에는 부족하므로 Round 1에서는 관찰된 경향만 기술하며, `distance가 증가할수록 transfer가 감소한다`는 일반적 결론을 주장하지 않는다.

## 2. 비교군

### 2.1 Backbone-matched methods

| Method ID | Method | 과거 transition replay | 비고 |
| --- | --- | ---: | --- |
| `iql_seq` | Sequential IQL with URMA-style I/O | 없음 | Topology를 사용하지 않는 component-wise 기준선 |
| `iql_ewc` | URMA-I/O IQL + actor-only EWC | 없음 | Sequential IQL과 같은 backbone 사용 |

Sequential IQL과 EWC는 architecture, source checkpoint와 target budget이 같은 직접 비교군이다. 여기서 `URMA-style I/O`는 variable joint/foot dimension을 처리하는 shared component encoder와 universal joint decoder를 뜻한다. PPO 기반 URMA 전체를 재현하는 것이 아니며, graph adjacency나 parent–child edge는 사용하지 않는다.

### 2.2 Cross-architecture references

| Method ID | Method | 과거 transition replay | 비고 |
| --- | --- | ---: | --- |
| `l2m_a1_adapt` | L2M (A1-pretrained adaptation) | 없음 | A1-only pretraining을 사용하는 축소 adaptation baseline |
| `vq_cd` | VQ-CD | 없음 | Paper-faithful reimplementation |

L2M과 VQ-CD는 IQL/EWC와 architecture 및 사전학습 과정이 다르다. 따라서 cross-architecture reference로 분리하고, parameter 수만이 아니라 representation pretraining, transition-token exposure, auxiliary memory와 wall-clock time을 함께 보고한다.

### 2.3 Single-task reference runs

| Reference ID | 목적 | 필수 여부 |
| --- | --- | ---: |
| `iql_scratch_a1` | A1 source checkpoint, A1 normalization reference와 capacity 확인 | 필수 |
| `iql_scratch_go1` | Go1-from-scratch curve와 IQL forward transfer 계산 | 필수 |
| `iql_scratch_h1` | H1-from-scratch curve와 IQL forward transfer 계산 | 필수 |
| `iql_scratch_hexapod` | Hexapod-from-scratch curve와 IQL forward transfer 계산 | 필수 |
| method-specific target scratch | L2M/VQ-CD의 strict forward transfer 계산 | Round 1에서는 선택 |

네 IQL single-task run은 새로운 CL method가 아니라 normalization과 forward-transfer 해석을 위한 reference다. Target-only curve 없이 측정한 target AUC는 acquisition speed일 뿐 forward transfer라고 부르지 않는다. L2M/VQ-CD도 method-specific target-only reference가 없으면 해당 방법의 AUC를 strict forward transfer라고 부르지 않는다.

## 3. 공통 data contract

### 3.1 입력과 mask

- Stored observation interface: 최대 `668`차원
- Stored action interface: 최대 `24`차원
- Robot별 native observation/action dimension은 [Dataset](../dataset.md)의 roster를 따른다.
- IQL/EWC loader는 native dimension을 이용해 raw observation을 `joint[J,26]`, `foot[F,12]`, `global[20]`으로 파싱한다.
- IQL/EWC encoder에는 뒤쪽 zero padding을 token으로 전달하지 않는다. 실제 joint/foot만 처리하고 batch padding이 필요할 때는 component mask를 사용한다.
- Actor는 active joint마다 action 하나를 생성한다. Logging과 공통 environment interface에서만 최대 24차원으로 padding한다.
- Actor loss, action sampling과 environment step에는 active action mask를 적용한다.
- Q-network는 active joint의 action만 joint token과 결합하며 inactive action dimension은 representation에 포함하지 않는다.
- L2M과 VQ-CD는 각 원 방법의 padding/tokenization 또는 QSA alignment를 사용하고 그 preprocessing을 별도 기록한다.
- Robot identity는 dataset slot index가 아니라 명시적인 robot name 또는 stable integer ID로 관리한다.

### 3.2 Transition 구성

- IQL/EWC는 robot별 transition을 uniform sampling한다.
- L2M/VQ-CD는 episode boundary를 넘지 않는 연속 sequence를 sampling한다.
- `terminated`와 `truncated`를 분리한다.
- Q target의 bootstrap mask는 `1 - terminated`로 계산한다. Time-limit truncation에는 bootstrap을 유지한다.
- Target 학습 중 A1 transition을 sampling하지 않는다.

### 3.3 Observation, action과 reward 처리

- Dataset에 이미 적용된 observation scaling을 사용한다.
- Robot별 observation z-normalization은 기본적으로 적용하지 않는다. Morphology descriptor의 물리적 차이를 지우지 않기 위함이다.
- Active action dimension은 robot별 environment action bound를 이용해 `[-1, 1]`로 변환하고, evaluation에서 원래 범위로 복원한다.
- 학습과 평가에는 raw reward를 사용한다.
- L2M의 return-to-go는 robot별 offline episode return의 95th percentile로 나눈다. Evaluation desired return은 normalized `1.0`으로 고정하고, 사용한 percentile 값을 run metadata에 기록한다.

### 3.4 Stage boundary, embodiment identity와 허용 정보

Round 1은 embodiment-incremental continual RL protocol로 정의한다. Stage boundary와 현재 학습/evaluation 대상 embodiment는 알려진다. 동일한 forward-locomotion objective를 유지하므로 source와 target을 서로 다른 locomotion task라고 표현하지 않는다. Embodiment identity를 사용하는 범위는 다음처럼 고정한다.

| 정보 또는 상태 | 허용 여부 | 사용 범위 |
| --- | ---: | --- |
| Stage boundary | 허용 | Source/target embodiment stage 전환과 optimizer reset |
| Robot identity | 허용 | Dataset routing, native dimension, action mask/bound와 adapter 선택 |
| Robot ID embedding | IQL/EWC에서 금지 | 명시적 one-hot 또는 learned ID를 policy/Q/V 입력에 concatenate하지 않음 |
| Observation에 저장된 morphology descriptor | 모든 방법에 허용 | Dataset의 공통 observation 정보 |
| Explicit graph adjacency/topology | Round 1 baseline에는 제공하지 않음 | 이후 topology-aware method의 추가 structural input |
| 과거 A1 transition/trajectory | Target stage에서 금지 | Replay 또는 재구성 학습에 사용하지 않음 |
| Method-specific persistent state | 허용 | EWC Fisher, L2M key/modulator, VQ codebook/mask 등 |

L2M의 task key와 VQ-CD의 robot-specific QSA/mask처럼 원 방법이 요구하는 routing state는 허용한다. 사용한 persistent state는 종류, byte 크기와 task 수에 따른 증가량을 기록한다. Fixed preprocessing statistics도 persistent state로 기록하며, raw transition을 통계량이라는 이름으로 보존하지 않는다.

본 dataset의 observation에는 morphology descriptor가 이미 포함되므로 IQL/EWC도 morphology-conditioned input을 받는다. 이후 제안 방법의 차이는 morphology의 존재 여부가 아니라 explicit topology를 학습 또는 knowledge consolidation에 사용하는지로 정의한다.

### 3.5 Canonical morphology distance contract

Pair 선택의 구조적 차이를 보조적으로 정량화하기 위해 16개 robot 전체에 대해 각 robot의 nominal configuration을 나타내는 torso–joint–foot graph를 만들고 Fused Gromov–Wasserstein (`FGW`) distance matrix를 한 번 계산한다. 이를 `canonical morphology distance`라고 부른다.

- Node: torso, active joint, foot
- Edge: torso–adjacent joint, kinematic joint–joint, terminal joint–foot
- Node feature: nominal relative position과 control-related local descriptor
- Feature cost: 전체 robot에서 표준화한 descriptor 사이의 Euclidean distance
- Structure cost: graph shortest-path distance
- Node weight: uniform
- FGW trade-off: `alpha=0.5`
- Numerical regularization: `epsilon=1e-3`

Graph adjacency는 flattened observation만으로 추정하지 않고 simulator/MJCF 또는 검증된 topology metadata에서 가져온다. Dataset descriptor는 physics randomization에 따라 episode 중 갱신될 수 있으므로 임의 transition 하나의 descriptor를 canonical morphology distance에 사용하지 않는다. 각 robot의 nominal simulator configuration에서 고정된 graph와 descriptor를 사용하고 그 source/version을 기록한다.

다음 두 값을 저장한다.

1. Raw FGW distance $d_{FGW}(i,j)$
2. 16-robot 전체 matrix에서 min–max normalization한 $d_{morph}(i,j)$와 $sim_{morph}=1-d_{morph}$

네 robot만 추려 다시 normalization하지 않는다. `A1-Go1`, `A1-H1`, `A1-Hexapod`의 near/far 표현은 실제 distance를 계산한 뒤 확정한다. Canonical morphology distance는 주 성능 지표가 아니며, 이 세 점만으로 distance와 transfer/forgetting의 correlation coefficient 또는 유의성을 보고하지 않는다.

### 3.6 Dataset difficulty와 confound 기록

세 pair는 모두 `all_robots_replay_forward_1m`을 사용한다. 모든 robot이 1,000,000 transitions를 가지더라도 episode 구성, termination과 behavior quality는 다르므로 다음 값을 robot별로 고정 보고한다.

- Transition 수와 episode 수
- Episode length의 mean, standard deviation과 quantile
- Termination/truncation rate
- Per-step reward의 mean, standard deviation과 quantile
- Episodic return의 `p10, p25, p50, p75, p90`
- Action saturation/coverage diagnostic
- Single-task IQL의 final return과 target AUC

Pair별 차이는 canonical morphology distance만으로 설명하지 않는다. 특히 target-only IQL curve를 해당 robot의 경험적 학습 난이도 reference로 사용하고, dataset return/termination 및 randomized morphology variation 통계와 함께 해석한다.

## 4. Evaluation protocol

### 4.1 모든 checkpoint에서 두 embodiment를 평가한다

Target 학습 budget을 0–100%로 놓고 다음 checkpoint를 사용한다.

```text
0%, 20%, 40%, 60%, 80%, 100%
```

각 checkpoint에서 **source embodiment(A1)와 target embodiment를 모두 평가**한다.

| Target progress | Source: A1 | Target Embodiment | 해석 |
| ---: | ---: | ---: | --- |
| `0%` | 필수 | 필수 | source 학습 직후 retention 기준과 target 초기 성능 |
| `20%` | 필수 | 필수 | 초기 forgetting/acquisition |
| `40%` | 필수 | 필수 | 중간 learning dynamics |
| `60%` | 필수 | 필수 | 중간 learning dynamics |
| `80%` | 필수 | 필수 | budget 충분성 점검 |
| `100%` | 필수 | 필수 | 최종 retention과 target 성능 |

`0%`의 target return은 zero-shot diagnostic일 뿐 Round 1의 독립적인 주 지표로 사용하지 않는다.

### 4.2 Evaluation episode와 seed

- Development training seed: `{42}`
- Final training seeds: `{43, 44, 45}`
- 기본 evaluation episodes: robot/checkpoint당 `20`
- Evaluation simulator seeds: `{1000, ..., 1019}`
- Policy evaluation: deterministic mean action 또는 각 방법이 정의한 deterministic inference
- 통계 표본 단위: evaluation episode가 아니라 training seed
- 보고: training seed 3개의 평균과 표준편차, 같은 seed끼리 paired difference

Development seed `42`는 architecture, hyperparameter와 구현 상태를 선택하는 데만 사용하고 최종 3-seed 통계에는 포함하지 않는다. Final seed `{43, 44, 45}`는 선택된 configuration을 평가하는 데만 사용한다. Training seed와 evaluation seed는 서로 다른 무작위성의 원천이므로 숫자가 겹쳐도 통계적 문제는 없지만, run metadata와 사람이 읽는 결과에서 역할을 분명히 하기 위해 서로 다른 범위를 사용한다.

한 robot에서 episode return의 변동계수가 `0.2`보다 크거나 method 순위가 evaluation seed에 따라 자주 바뀌면 evaluation episode만 `50`으로 늘린다. Final training seed는 Round 1에서 3개로 유지한다.

### 4.3 Main metrics

Robot마다 raw undiscounted episodic return을 별도로 보고한다. Raw return은 원 결과를 해석하는 주 지표이며, 서로 다른 robot의 raw return을 그대로 합산하거나 평균하지 않는다.

Source embodiment retention은 다음 값으로 기록한다.

$$
\Delta R_{source}=R_{A1}^{after}-R_{A1}^{before},
$$

$$
Retention_{source}=100\times\frac{R_{A1}^{after}}{R_{A1}^{before}}.
$$

`R_before`가 0에 가깝거나 reward offset의 영향을 크게 받으면 retention ratio는 보고하지 않고 raw difference와 4.5절의 normalized forgetting을 사용한다. Forgetting은 양수가 직관적이도록 다음과 같이 병기한다.

$$
Forgetting_{source}=R_{A1}^{before}-R_{A1}^{after}.
$$

Target embodiment는 `100%` checkpoint의 raw target return을 주 지표로 사용한다. Normalized score는 보조 자료로 병기하고, episode length와 termination rate도 diagnostic으로 기록한다.

### 4.4 Learning curve와 AUC

Learning curve는 target progress $x_k\in[0,1]$에 따른 return $R_k$를 기록한다. Target AUC는 trapezoidal rule로 계산한다.

$$
AUC_{target}=\sum_{k=1}^{K}(x_k-x_{k-1})\frac{R_k+R_{k-1}}{2}.
$$

Progress 축을 0–1로 두므로 AUC는 동일 target budget에서 얻은 평균적인 target return으로 해석할 수 있다.

AUC의 목적은 다음과 같다.

- 최종 return이 같을 때 어느 방법이 더 빨리 유용한 성능에 도달했는지 구분
- 중간 collapse나 불안정성을 최종 checkpoint가 숨기는지 확인
- target budget이 아직 부족한지 확인

고정 offline dataset을 반복해서 사용하므로 AUC를 environment sample efficiency로 표현하지 않는다. `update efficiency` 또는 `data-exposure efficiency`로 표현한다.

Sequential IQL의 forward transfer는 동일 architecture, target dataset, update budget, training seed와 evaluation checkpoint를 사용한 target-from-scratch IQL과 비교한다.

$$
\Delta AUC_{IQL}=AUC_{target}^{iql\_seq}-AUC_{target}^{iql\_scratch}.
$$

Raw-return AUC 차이는 같은 target robot 안에서 해석하는 Round 1의 주 forward-transfer 지표다. 4.5절의 normalized score curve로 계산한 AUC 차이는 robot 사이 scale을 맞춰 보는 보조 자료로만 사용한다. Continual World의 ceiling-normalized 식은 success가 `[0,1]`로 제한된 setting을 전제로 하므로 본 score에 그대로 적용하지 않는다.

L2M/VQ-CD는 method-specific target-from-scratch reference가 있을 때만 동일한 방식으로 forward transfer를 계산한다. 해당 reference가 없으면 target AUC는 acquisition speed로만 보고한다.

### 4.5 보조 지표: Single-task-relative normalized score

Robot별 reward scale 차이를 보조적으로 확인하기 위해 raw return과 별도로 다음 single-task-relative normalized score를 사용한다. Method 선택과 각 robot 내부의 주 결론은 raw return 및 raw-return AUC를 기준으로 한다.

$$
S_i(R)
=
100\frac{R-R_{random,i}}
{R_{STL,i}-R_{random,i}}.
$$

- $R_{random,i}$: robot $i$의 valid action bound에서 uniform random action을 사용하는 policy의 평균 return
- $R_{STL,i}$: 선택된 IQL architecture와 full Round 1 budget으로 robot $i$를 scratch부터 학습한 세 training seed의 final return 평균

Random reference는 robot당 고정된 100 evaluation seeds에서 한 번 측정하고 모든 method가 공유한다. $R_{STL,i}-R_{random,i}$가 0에 가깝거나 그 차이의 confidence interval이 0을 포함하면 해당 robot의 normalized score를 사용하지 않고 raw metric만 보고한다.

$S=100$은 Round 1의 single-task IQL reference와 같은 성능을 뜻한다. 100보다 큰 값과 0보다 작은 값을 허용하며 metric 계산에서 clipping하지 않는다. 이는 expert reference를 사용하는 D4RL normalized score가 아니므로 결과에서 `STL-relative normalized score`라고 명시한다. Round 1 진행에 expert return은 필수적이지 않다.

Normalized source forgetting은 다음처럼 계산한다.

$$
F_{source}^{S}=S_{A1}^{before}-S_{A1}^{after}.
$$

보조 normalized forward transfer는 target progress에 따른 $S_i$ curve의 AUC 차이로 계산한다.

$$
FT_{IQL}^{AUC}
=
AUC(S_{target}^{iql\_seq})-AUC(S_{target}^{iql\_scratch}).
$$

Pair 전체를 한눈에 보는 보조 summary가 필요하면 normalized forgetting, normalized target final score와 normalized forward transfer를 각각 제시한다. 이 aggregate로 robot별 raw 결과를 대체하거나 method 순위를 단정하지 않으며, stability와 plasticity를 하나의 임의 가중합으로 합치지 않는다.

## 5. URMA-style IQL backbone capacity gate

### 5.1 왜 별도 점검이 필요한가

공식 IQL D4RL locomotion configuration과 대표적인 TD3+BC 구현은 `2 x 256` MLP를 사용한다. 따라서 policy/critic core의 `2 x 256`은 임의로 작은 설정이 아니라 문헌에 근거한 출발점이다.

그러나 본 setting은 최대 24개 joint action과 가변적인 joint/foot token을 하나의 shared encoder/decoder로 처리한다. D4RL에서 flat `2 x 256` MLP가 작동했다는 사실만으로 component encoder, attention pooling과 universal decoder를 포함한 전체 capacity가 충분하다고 결론낼 수 없다.

### 5.2 Candidate와 budget

다음 세 candidate를 `unitree_a1`과 `unitree_h1` single-task IQL에서만 비교한다.

| Candidate | Component latent | Policy/critic core | 목적 |
| --- | ---: | --- | --- |
| `small` | `128` | `256, 256` | 공식 IQL core 규모를 사용한 최소 구조 |
| `deep` | `128` | `256, 256, 256, 256` | Core depth 증가 효과 점검 |
| `large` | `256` | `512, 512, 512, 512` | Component latent와 core width 부족 여부 점검 |

- Seed: `42` 한 개
- Robot: `unitree_a1`, `unitree_h1`
- Budget: robot/configuration당 `30k` gradient updates
- Evaluation: `0, 10k, 20k, 30k`
- 그 외 IQL hyperparameter는 6절 기본값으로 고정

총 6개의 short run이다.

### 5.3 선택 규칙

1. 각 robot에서 마지막 두 checkpoint 평균 return을 계산한다.
2. 두 robot 모두에서 최고 candidate의 `95%` 이상을 달성한 가장 작은 candidate를 선택한다.
3. NaN, critic divergence 또는 seed 내 급격한 collapse가 있는 candidate는 제외한다.
4. 모든 candidate가 마지막 10k 구간에서 계속 뚜렷하게 상승하면 상위 두 candidate만 `60k`까지 연장한다.
5. 선택한 component latent와 core architecture는 Sequential IQL과 EWC의 모든 pair와 seed에 고정한다.

이 gate는 “4-layer가 항상 더 좋다”를 보이는 실험이 아니다. 본 baseline이 명백한 capacity bottleneck 때문에 약해지는 것을 방지하는 일회성 선택 절차다. `4 x 256`이 `4 x 512`와 비슷하면 사용자가 사용하던 depth 4가 충분하다고 판단할 수 있다.

## 6. Sequential IQL with URMA-style I/O

### 6.1 Model behavior

- Actor, twin Q와 V는 같은 **구조**를 사용하지만 parameter는 공유하지 않는다.
- Joint descriptor 23차원과 joint dynamic state 3차원은 각각 shared MLP로 encode한다. 같은 MLP weight를 해당 robot의 모든 joint에 적용한다.
- Foot descriptor 10차원과 foot dynamic state 2차원도 각각 shared MLP로 encode한다.
- Joint와 foot token은 descriptor-conditioned masked attention으로 각각 하나의 latent로 pooling한다.
- Pooled joint latent, pooled foot latent와 global state 20차원을 concatenate하여 capacity gate에서 선택한 core에 전달한다.
- Actor의 universal joint decoder는 core latent, 해당 joint latent와 joint descriptor latent를 입력받아 active joint마다 Gaussian mean과 log standard deviation 하나를 출력한다. 모든 joint가 같은 decoder weight를 사용한다.
- Q-network는 각 active joint의 action scalar를 해당 joint token에 concatenate한 뒤 shared action-conditioned encoder와 pooling을 거쳐 scalar Q를 출력한다.
- V-network는 action을 받지 않는 state encoder와 pooling을 거쳐 scalar V를 출력한다.
- Explicit graph adjacency, `parent_idx`와 message passing은 사용하지 않는다. Joint relative position, axis, limit과 같은 observation descriptor만 사용한다.
- Hidden activation은 ReLU, weight initialization은 orthogonal initialization을 사용하고 dropout은 적용하지 않는다.
- Batch padding token과 inactive action은 attention, loss와 environment action에서 제외한다.

기본 component module은 다음과 같이 둔다.

| Module | 기본 구조 |
| --- | --- |
| Joint descriptor/dynamic encoder | 각각 2-layer MLP, output은 candidate latent dimension |
| Foot descriptor/dynamic encoder | 각각 2-layer MLP, output은 candidate latent dimension |
| Joint/foot pooling | 1-layer descriptor-conditioned attention |
| Policy/Q/V core | Capacity gate에서 선택 |
| Universal joint decoder | 2-layer shared MLP, joint당 Gaussian parameter 출력 |

### 6.2 A1에서 H1로 전환하는 예시

Flat padded MLP에서는 A1 학습 중 observation `380:667`과 action output `12:23`에 대응하는 weight가 거의 학습되지 않는다. H1으로 전환하면 19개 joint를 사용하므로, A1에서 사용되지 않던 일곱 action output과 추가 observation position을 처음부터 학습해야 한다. 또한 flat vector의 같은 index가 두 robot에서 의미적으로 비슷한 joint라는 보장도 없다.

URMA-style I/O에서는 A1의 12개 joint가 모두 하나의 shared joint encoder와 universal decoder를 반복해서 사용한다. H1의 19개 joint도 같은 encoder/decoder를 사용한다. 예를 들어 H1에만 존재하는 추가 joint는 새 output neuron을 만드는 대신, A1의 여러 joint를 처리하며 이미 학습된 decoder에 자신의 relative position, axis, range, torque limit과 현재 joint state를 입력한다.

따라서 H1의 추가 joint는 **새로운 dimension-specific parameter를 cold start하는 대신 기존의 joint-control function을 재사용**한다. 다만 joint 사이의 parent–child connectivity는 주어지지 않으므로, topology-aware method가 추가할 structural information은 여전히 분리되어 있다.

### 6.3 기본 hyperparameter

| Hyperparameter | 기본값 | 최소 점검 범위 |
| --- | ---: | --- |
| Actor/Q/V learning rate | `3e-4` | `{1e-4, 3e-4}` |
| Batch size | `256 transitions` | 고정 |
| Discount | `0.99` | 고정 |
| IQL expectile | `0.7` | `{0.7, 0.8}` |
| Advantage temperature | `3.0` | `{1.0, 3.0}` |
| Target EMA | `0.005` | 고정 |
| Gradient clipping | `1.0` | 고정 |

전체 Cartesian product를 실행하지 않는다. 먼저 공식 기본값을 사용한다. A1 또는 H1 source curve가 발산하거나 30k 이후에도 거의 개선되지 않을 때만 한 번에 한 hyperparameter를 바꾸며, 최대 4개 configuration까지만 점검한다.

### 6.4 학습 budget과 checkpoint 전달

- A1 source: `50k` updates, batch `256`
- Target: `50k` updates, batch `256`
- 각 stage exposure: `12.8M transition draws`
- Single-task reference: A1, Go1, H1과 Hexapod 각각 `50k` updates를 final training seed `{43, 44, 45}`로 실행
- Single-task evaluation checkpoint: `0, 10k, 20k, 30k, 40k, 50k`
- A1 학습 후 actor, Q, V와 target Q parameter를 target branch로 전달한다.
- Target 시작 시 optimizer state는 초기화한다.
- Model parameter는 초기화하거나 freeze하지 않는다.
- Target 학습에는 target robot dataset만 사용한다.

각 training seed의 A1 checkpoint는 같은 seed의 세 target branch에 재사용한다. Target-from-scratch run도 sequential target stage와 같은 selected architecture, IQL hyperparameter, dataset sampler, update budget과 checkpoint를 사용한다.

공식 IQL offline training script의 기본 budget은 `1M` updates다. 여기의 `50k`는 원 논문 재현 budget이 아니라 Round 1의 최소 screening budget이다. 따라서 낮은 성능이나 계속 상승하는 curve를 곧바로 IQL의 실패로 해석하지 않는다.

Target curve가 마지막 `20%` 구간에서도 상승하고 `80% -> 100%` return 증가가 전체 증가량의 `10%`보다 크면, 선택 configuration을 `100k`까지 연장해야 한다고 보고한다. Round 1의 모든 method를 자동으로 연장하지 않는다.

## 7. URMA-I/O IQL + actor-only EWC

### 7.1 적용 범위

EWC는 Sequential IQL과 같은 source checkpoint, architecture, optimizer와 target budget을 사용한다. 차이는 target stage의 actor objective에 EWC penalty가 추가되는 것뿐이다.

$$
L_{actor}^{EWC}=L_{actor}^{IQL}+\frac{\lambda}{2}\sum_j F_j(\theta_j-\theta_j^*)^2.
$$

- 적용 parameter: actor 전체
- 적용하지 않는 parameter: Q, V와 target Q
- Fisher: A1 dataset에서 추정한 diagonal empirical Fisher
- Fisher gradient: stochastic actor의 masked log-likelihood gradient
- 저장 상태: source actor $\theta^*$와 diagonal $F$

IQL actor에 이 Fisher 정의를 적용하는 것은 EWC 원 논문의 그대로인 설정이 아니라 본 Round 1 adaptation임을 결과에 명시한다.

### 7.2 고정값과 tuning

| Hyperparameter | 값 |
| --- | --- |
| Fisher minibatches | `50` |
| Fisher batch size | `256` |
| Fisher aggregation | minibatch 평균 후 전체 평균 |
| Fisher damping | `1e-8` |
| EWC coefficient $\lambda$ | `{0.1, 1, 10}` |

세 $\lambda$를 development seed `42`에서 세 pair에 실행한다. Target final return이 Sequential IQL의 `80%` 이상인 설정 중 평균 source retention이 가장 높은 $\lambda$ 하나를 선택하고, 그 값만 final seed `{43, 44, 45}`로 실행한다. Pair별로 다른 $\lambda$를 고르지 않는다.

다음이면 추가 tuning 필요로 기록한다.

- 세 값 모두 Seq-IQL과 동일한 forgetting을 보임: Fisher scale/parameter scope 점검
- 세 값 모두 target acquisition이 Seq-IQL의 80% 미만: 더 작은 $\lambda$ 필요
- EWC loss가 actor loss보다 지속적으로 수십 배 큼: Fisher normalization 또는 $\lambda$ scale 점검

## 8. L2M (A1-pretrained adaptation)

### 8.1 Round 1에서의 위치

L2M은 IQL 위에 추가하는 regularizer가 아니다. Frozen pretrained backbone, key와 LoRA modulator pool이라는 원 방법의 mechanism을 유지하는 별도 architecture baseline이다.

A1만으로 compact backbone을 사전학습하므로 `L2M (A1-pretrained adaptation)`으로 표기한다. 원 논문의 multi-domain pretraining을 직접 재현한 것으로 주장하지 않는다.

### 8.2 기본 구조와 hyperparameter

| 항목 | 기본값 | 최소 점검 |
| --- | ---: | --- |
| Backbone | compact MDDT | 고정 |
| Layers / heads / embedding | `4 / 4 / 256` | 고정 |
| Context length | `5` | 고정 |
| Action bins | `64` | 고정 |
| LoRA rank | `8` | 고정 |
| Modulation pool size | `30` | 고정 |
| Key regularization | `0.5` | `{0.1, 0.5}` |
| Pretrain learning rate | `1e-4` | 고정 |
| Modulator learning rate | `1e-4` | `{5e-5, 1e-4}` |
| Dropout / weight decay | `0.2 / 0.01` | 고정 |
| Batch size | `256 sequences` | 고정 |

### 8.3 학습 흐름과 budget

1. A1 sequence로 backbone을 사전학습한다.
2. A1 학습이 끝나면 backbone을 동결한다.
3. 각 target branch에서 modulation pool과 task key만 학습한다.
4. A1 evaluation에는 source modulation state를, target evaluation에는 target에서 선택된 modulation state를 사용한다.

- Source pretraining: `10k` updates × `256` sequences × context `5` = `12.8M transition tokens`
- Target modulation: `10k` updates × `256` sequences × context `5` = `12.8M transition tokens`
- Target evaluation checkpoints: `0, 2k, 4k, 6k, 8k, 10k`

A1 source performance가 selected IQL의 `80%` 미만이면 L2M mechanism 비교 전에 source backbone budget이 부족한 것으로 기록한다. Target curve가 마지막 20%에서도 상승하면 `25k` target update 확장이 필요하다고 보고한다.

Development seed `42`에서 기본값과 learning rate 대체값을 먼저 비교한다. Routing collapse가 발생할 때만 key regularization `0.1`을 추가하며, 선택된 하나의 configuration만 final seed `{43, 44, 45}`로 실행한다.

## 9. VQ-CD

### 9.1 Round 1에서의 위치

VQ-CD는 QSA로 heterogeneous state/action space를 정렬하고, SWA mask를 사용하는 diffusion policy를 학습하는 별도 architecture baseline이다. 현재 official repository가 확인되지 않았으므로 paper-faithful reimplementation으로 기록한다.

### 9.2 기본 구조와 hyperparameter

| 항목 | 기본값 | 최소 점검 |
| --- | ---: | --- |
| QSA/SWA hidden size | `256` | 고정 |
| State/action codebook | `512 / 512` | collapse 시 `{256, 512}` |
| Commitment coefficient | `0.25` | 고정 |
| Codebook limit $\rho$ | `3.0` | 고정 |
| State latent | `10 x 2` | 고정 |
| Action latent | `5 x 2` | 고정 |
| QSA learning rate | `1e-3 -> 1e-4` | 고정 |
| Diffusion learning rate | `3e-4` | 고정 |
| Sequence length / batch | `8 / 32` | 고정 |
| Mask rate | pairwise setting에서 `0.5` | 고정 |
| Condition dropout | `0.25` | 고정 |
| Diffusion steps / DDIM stride | `200 / 20` | 고정 |
| Guidance value / weight | `0.95 / 1.2` | 고정 |

### 9.3 학습 흐름과 budget

1. A1과 현재 target 각각에 대해 QSA를 학습한다.
2. Reconstruction loss, codebook utilization과 dead-code ratio를 검증한다.
3. A1에서 source SWA diffusion을 학습한다.
4. 같은 SWA state에서 target branch를 시작하며 target mask에 의해 선택된 weight를 학습한다.
5. Evaluation에서는 robot identity로 QSA와 mask를 선택하고 inverse dynamics로 action을 복원한다.

- QSA: robot당 `25k` updates × `32` sequences × length `8` = `6.4M transition tokens`
- Source SWA: `50k` updates × `32` sequences × length `8` = `12.8M transition tokens`
- Target SWA: `50k` updates × `32` sequences × length `8` = `12.8M transition tokens`
- Target evaluation checkpoints: `0, 10k, 20k, 30k, 40k, 50k`

QSA는 policy training 전에 target dataset을 사용하는 method-specific representation step이다. 따라서 policy exposure와 별도로 QSA exposure를 반드시 보고하며, IQL과 엄밀히 같은 총 계산량이라고 주장하지 않는다.

먼저 development seed `42`에서 codebook `512`를 실행한다. Code utilization이 `10%` 미만이거나 dead-code ratio가 `80%`보다 크면 codebook `256`을 한 번 추가한다. 선택된 configuration만 final seed `{43, 44, 45}`로 실행한다.

## 10. Training-sample exposure와 model-size 보고

`Training-sample exposure`는 model size가 아니라 optimizer가 dataset sample을 처리한 누적 횟수다. 다음처럼 계산한다.

$$
\text{exposure}=\text{gradient updates}\times\text{batch size}\times\text{sequence length}.
$$

`M`은 million을 뜻한다. 예를 들어 IQL의 `50k updates x batch 256`은 `12,800,000 = 12.8M` transition draws다. 이 dataset에는 robot별 unique transition이 1M개이므로 `12.8M exposure`는 새로운 transition 12.8M개를 뜻하지 않는다. 고정 dataset에서 replacement sampling으로 transition을 반복해 읽고 gradient를 계산한 총량이다.

Main target adaptation과 method-specific representation 학습의 현재 exposure는 다음과 같다.

| Method | Main target adaptation | Target representation/alignment | 현재 total target-stage exposure |
| --- | ---: | ---: | ---: |
| IQL / EWC | `50k × 256 = 12.8M` | 없음 | `12.8M` |
| L2M (A1-pretrained adaptation) | `10k × 256 × 5 = 12.8M` | 없음 | `12.8M` |
| VQ-CD | SWA `50k × 32 × 8 = 12.8M` | QSA `25k × 32 × 8 = 6.4M` | `19.2M` |

따라서 현재 설정은 main adaptation token만 같고, VQ-CD가 target QSA에서 추가로 `6.4M`을 처리한다. Exposure를 맞추는 이유는 한 방법이 같은 offline dataset을 더 많이 반복 학습해서 얻은 이득을 architecture의 이득으로 오해하지 않기 위해서다.

Exposure가 같아도 optimization difficulty나 실제 계산량이 같다는 뜻은 아니다. 다음 네 종류의 비용을 분리해서 보고한다.

- Actor/policy inference parameter 수
- 전체 trainable parameter 수
- Robot별 activated parameter 수
- EWC Fisher, L2M pool, VQ codebook/mask를 포함한 auxiliary memory
- Peak GPU memory
- Source와 target wall-clock time
- Inference latency
- Method-specific representation pretraining exposure

## 11. 최소 tuning budget과 선택 규칙

Round 1은 exhaustive tuning을 하지 않는다.

| Method | Development seed 42에서 허용되는 최대 점검 | Final seed 43/44/45 실행 |
| --- | ---: | --- |
| IQL common config | 최대 4 configs | 선택된 1개 |
| EWC | 3 lambdas | 선택된 1개 |
| L2M (A1-pretrained adaptation) | 기본 + lr 대체, 필요 시 key reg 1개 | 선택된 1개 |
| VQ-CD | codebook 512, collapse 시 256 | 선택된 1개 |

Hyperparameter는 세 pair의 평균적인 acquisition–retention을 보고 하나의 공통값을 선택한다. Pair마다 별도 최적값을 고르지 않는다.

추가 tuning이나 budget 확장이 필요하다고 판단하는 조건:

- NaN, exploding loss 또는 critic divergence
- Target curve가 마지막 20%에서도 뚜렷하게 상승
- Source model 자체가 낮아 retention을 해석할 수 없음
- EWC penalty scale이 사실상 0이거나 actor update를 완전히 막음
- L2M routing collapse
- VQ-CD reconstruction/codebook collapse
- Seed 3개의 순위가 일관되지 않음

이 경우 실패 결과를 숨기지 않고 `implementation issue`, `insufficient budget`, `hyperparameter sensitivity`, `method failure` 중 어느 가설이 가장 유력한지 기록한다. 추가 run은 자동으로 실행하지 않는다.

### 11.1 Baseline reproduction gate와 VQ-CD 처리

각 baseline은 다음 최소 조건을 통과한 뒤에만 성능 비교표에 포함한다.

- 공통: NaN, exploding loss, 잘못된 mask 또는 episode-boundary 누수가 없음
- 공통: Source policy가 random-policy reference보다 명확히 높고 target 학습 curve가 random reference에서 개선됨
- EWC: Fisher와 penalty가 유한하고, penalty가 항상 0이거나 actor update를 완전히 막지 않음
- L2M: Source backbone과 routing/modulation이 모두 동작하고 하나의 key 또는 modulator로 완전히 collapse하지 않음
- VQ-CD QSA: state/action reconstruction, codebook utilization과 dead-code diagnostic을 보고할 수 있음
- VQ-CD SWA: Source와 target stage 모두에서 유효한 trajectory와 action을 생성하며 source/target evaluation을 완료함

VQ-CD는 필수 비교군이지만 공식 구현이 확인되지 않은 paper-faithful reimplementation이므로 다른 baseline의 진행을 막는 선행 조건으로 두지 않는다. IQL, EWC와 L2M의 검증된 결과를 먼저 생성하고, VQ-CD는 `QSA -> source SWA -> target SWA -> evaluation` 단계별 상태와 현재까지의 성능을 별도로 보고한다.

VQ-CD가 reproduction gate를 통과하지 못하면 collapse한 수치를 정상 baseline 성능처럼 비교표에 넣지 않는다. 대신 마지막으로 통과한 단계, loss/diagnostic, 사용한 budget과 다음에 점검할 가설을 `VQ-CD pending reproduction`으로 보고하고 사용자와 추가 수정 범위를 결정한다. 이 상태에서는 다른 세 방법의 중간 결과를 분석할 수 있지만, 네 방법을 모두 포함하는 Round 1 최종 비교가 완료된 것은 아니다.

## 12. 실행 순서

1. Robot별 native dimension, padding, action mask와 episode boundary 검증
2. 네 robot의 episode/reward/action 통계와 random-policy reference 생성
3. 16-robot nominal graph를 검증하고 FGW canonical morphology distance matrix 생성
4. Capacity gate: A1/H1 × 3 architectures × seed 42
5. 선택 IQL configuration으로 A1/Go1/H1/Hexapod single-task reference 생성
6. Development seed 42에서 세 pair의 Sequential IQL 실행
7. Development seed 42에서 EWC/L2M/VQ-CD의 최소 configuration 선택
8. 선택된 configuration을 final seed 43/44/45로 실행
9. 모든 target checkpoint에서 source와 target embodiment 평가가 존재하는지 검증
10. Raw return, STL-relative score, forgetting, target AUC와 계산 자원 표 생성
11. Canonical morphology distance, randomized morphology variation과 dataset difficulty를 함께 둔 pair별 해석 작성
12. 추가 tuning/budget 필요 보고 작성

## 13. 필수 logging schema

각 training/evaluation record에는 최소한 다음 field가 있어야 한다.

```text
method
pair_id
dataset_id
source_robot
target_robot
train_seed
seed_role             # development | final
eval_seed
stage                 # source | target | qsa | pretrain
update
target_progress
eval_robot            # source 또는 target embodiment
episodic_return
normalized_score      # STL-relative; 사용할 수 없으면 null
episode_length
terminated
model_parameters
activated_parameters
auxiliary_memory_bytes
transition_tokens_seen
wall_clock_seconds
canonical_morphology_distance_raw
canonical_morphology_distance_normalized
config_hash
checkpoint_path
```

Run-level metadata에는 stage-boundary knowledge, robot identity의 사용 위치, topology 접근 여부, persistent-state 종류, $R_{random}$과 $R_{STL}$ reference version을 저장한다. Dataset 통계와 16-robot canonical morphology matrix는 별도 versioned artifact로 저장하고 각 run에서 그 version을 참조한다.

각 target checkpoint에서 `eval_robot=unitree_a1`과 `eval_robot=<target>` record가 모두 없으면 해당 run은 완료로 간주하지 않는다.

## 14. Round 1 완료 조건

다음 조건을 모두 만족해야 한다.

1. A1, Go1, H1과 Hexapod의 single-task IQL reference가 final training seed 3개로 존재한다.
2. 세 pair와 네 method에 final training seed 3개의 결과가 있고 development seed 결과와 분리되어 있다.
3. 모든 target checkpoint에서 source와 target embodiment가 모두 평가되었다.
4. Target 종료 후 raw A1 forgetting과 raw target final performance를 계산할 수 있고, reference가 유효하면 normalized 보조 지표도 병기되어 있다.
5. Sequential IQL은 동일 조건의 target-from-scratch curve와 forward transfer를 비교할 수 있다.
6. 세 pair의 canonical morphology distance, randomized morphology variation과 robot별 dataset difficulty 통계가 기록되었다.
7. 각 method가 사용한 identity/topology 정보, persistent state, target exposure, parameter 수, auxiliary memory와 wall-clock time이 기록되었다.
8. 실패하거나 계속 상승하는 configuration에는 추가 tuning 필요 여부가 기록되었다.
9. Robot 사이 raw return을 평균한 aggregate score를 만들지 않았다.
10. Canonical morphology distance와 normalized score를 보조 자료로만 사용했으며, 세 pair만으로 distance와 transfer/forgetting의 통계적 상관관계를 주장하지 않았다.
11. VQ-CD가 reproduction gate를 통과하지 못한 경우 해당 결과를 정상 baseline과 비교하지 않았으며 Round 1을 최종 완료로 표시하지 않았다.

Round 1은 baseline의 범위와 failure mode를 확보하면 종료한다. Topology-aware method의 우수성 검증은 다음 Round에서 같은 protocol을 사용해 수행한다.
