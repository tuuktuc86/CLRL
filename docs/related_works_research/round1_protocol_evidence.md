# Round 1 protocol evidence: forward transfer, morphology distance, and baseline taxonomy

이 메모는 Round 1 실험 설계에 대해 제기된 의견들을 1차 자료 기준으로 점검한 결과다. 핵심은 두 가지다.

1. Continual World의 forward transfer 정의는 single-task reference curve를 전제로 하므로, target-only scratch baseline은 단순 비교용이 아니라 FT 해석에 사실상 필요하다.
2. Cross-Embodiment Offline Reinforcement Learning 논문은 morphology graph + FGW distance + gradient conflict 상관관계를 직접 보여주므로, morphology distance를 실험 변수로 명시하는 제안은 근거가 있다.

아래에서 `검증됨`은 원문에서 직접 확인한 사실, `추론`은 원문을 바탕으로 한 실험 설계 권고다.

## 1) Continual World: forward transfer와 single-task baseline

### 검증됨

Continual World 논문은 forward transfer를 `single-task, reference experiment`와의 learning curve 면적 차이로 정의한다.

- 논문은 forward transfer를 “reference, single-task experiment”와 비교하는 normalized area로 정의한다.
- 정확한 식은 `FT_i = (AUC_i - AUC_b_i) / (1 - AUC_b_i)` 이다.
- 여기서 `AUC_i`는 task `i` 구간의 learning curve 적분이고, `AUC_b_i`는 그 task를 scratch로 학습한 reference curve의 AUC다.

원문 근거:

- [Continual World: A Robotic Benchmark For Continual Reinforcement Learning](https://arxiv.org/abs/2105.10919)
- PDF: [2105.10919v3](https://arxiv.org/pdf/2105.10919.pdf)

### 해석

이 정의 때문에, `A1 -> H1` 같은 sequential run만 있으면 target을 “빨리 배운 것인지”를 정량적으로 분리하기 어렵다. 같은 architecture, 같은 dataset, 같은 update budget으로 `H1-from-scratch IQL`을 돌려야 `AUC_sequential - AUC_scratch`를 forward transfer로 해석할 수 있다.

즉, 사용자가 제안한 `A1-only`, `Go1-only`, `H1-only`, `Hexapod-only` IQL 확보는 원래 benchmark 정의와 잘 맞는다.

### 추론

- target-only scratch baseline은 “있으면 좋은 보조선”이 아니라, forward transfer를 주장하려면 필요한 reference다.
- learning curve를 evaluation checkpoint마다 저장하는 것도 타당하다. 이건 논문 정의상 필수는 아니지만, FT와 forgetting을 안정적으로 계산하고 구간별 붕괴를 읽어내는 데 유리하다.

## 2) Cross-Embodiment Offline RL: morphology distance를 실험 변수로 둘 근거

### 검증됨

`Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets`는 다음을 직접 주장한다.

- 16개 robot platform으로 구성된 locomotion datasets를 만든다.
- suboptimal data가 많아질수록, 그리고 robot type 수가 많아질수록 cross-embodiment learning에서 gradient conflict가 증가한다.
- 이를 완화하기 위해 embodiment similarity로 robot을 grouping한다.

원문 근거:

- [arXiv 2602.18025](https://arxiv.org/abs/2602.18025)
- PDF: [2602.18025v1](https://arxiv.org/pdf/2602.18025.pdf)

### 검증됨: morphology graph와 FGW distance

논문은 robot embodiment를 graph로 표현한다.

- node: torso, joints, feet
- edge: torso–adjacent joint, adjacent joints, terminal joint–foot
- node features: torso로부터의 상대 위치와 control parameters
- distance: Fused Gromov–Wasserstein (FGW)

또한 논문은 FGW distance matrix를 `1 - min-max-normalized FGW distance` 형태의 similarity로 바꿔 시각화하고, quadruped들이 가깝게 cluster된다고 보고한다. 예시는 `Unitree Go1, A1, Go2, Anymal B/C`다.

원문 근거:

- `1 - min-max-normalized FGW distance between robot pairs`
- `Quadrupedal robots cluster closely`
- [2602.18025 PDF, Fig. 3 / Section 5.2](https://arxiv.org/pdf/2602.18025.pdf)

### 검증됨: 상관관계 수치

이 논문에서 확인된 상관관계는 두 종류다.

1. embodiment similarity vs mean gradient cosine similarity
   - `r = 0.63`, `p = 1.26 × 10^-14`
2. transfer gain vs average gradient cosine similarity
   - `r = 0.815`

중요한 점은 이 둘이 같은 값이 아니라는 것이다.

- `r = 0.63`은 morphology similarity와 gradient similarity의 상관관계다.
- `r = 0.815`는 transfer gain과 gradient similarity의 상관관계다.

따라서 사용자가 말한 “morphology similarity와 IQL gradient similarity 사이의 상관관계”는 `r = 0.63`이 맞고, `0.815`는 별도의 결과다.

원문 근거:

- [2602.18025 PDF, Section 5.1–5.2](https://arxiv.org/pdf/2602.18025.pdf)

### 검증됨: suboptimality / diversity / gradient conflict

논문은 다음을 직접 보고한다.

- suboptimal data 비율이 증가할수록 negative cosine similarity 비율이 증가한다.
- 더 다양한 embodiment를 포함할수록 negative cosine similarity가 증가한다.
- 이 현상은 IQL뿐 아니라 TD3+BC에서도 유사하게 나타난다.

즉, “dataset quality”와 “morphology diversity”가 gradient conflict를 동시에 악화시킬 수 있다는 점은 이 논문에서 명시적으로 지지된다.

원문 근거:

- [2602.18025 PDF, Figure 2 / Section 5.1](https://arxiv.org/pdf/2602.18025.pdf)

### 추론

- `A1 -> Go1`, `A1 -> H1`, `A1 -> Hexapod`를 고르는 것은 단순 직관이 아니라, 논문이 보여준 “quadruped cluster vs morphologically distant robot” 구조와 잘 맞는다.
- 따라서 Round 1에 `d_morph(A1, Go1)`, `d_morph(A1, H1)`, `d_morph(A1, Hexapod)`를 함께 기록하는 제안은 후속 질문인 “distance가 커질수록 transfer가 감소하고 forgetting이 증가하는가?”로 자연스럽게 이어진다.

## 3) source return curve와 normalized score

### 검증됨

Continual World는 forward transfer와 forgetting을 모두 curve 기반으로 정의한다. 따라서 checkpoint마다 source task 성능을 함께 저장하면, 최종값뿐 아니라 학습 중 잊는 패턴을 볼 수 있다.

### 추론

- `R_A1_before`, `R_A1_after`, `R_target_final`만 저장하는 최소안은 가능하지만, evaluation마다 A1도 같이 측정하면 A1 retention curve를 거의 추가 training cost 없이 얻는다.
- raw return은 robot마다 scale이 다를 수 있으므로, 내부 분석용 normalized score를 두는 제안은 합리적이다.
- 다만 normalized score는 Continual World 원 논문이 요구하는 필수 형식은 아니다. 원문 정의의 핵심은 task별 single-task reference와 curve AUC다.

## 4) baseline taxonomy: 같은 종류의 baseline인지 구분해야 한다

### 추론

이 부분은 원문 정의에서 직접 나온 결론은 아니고, 실험 설계 상의 분류 제안이다.

- `Sequential IQL`과 `IQL + actor-only EWC`는 같은 backbone 위에서 retention mechanism만 바꾸는 backbone-matched 비교로 두는 편이 맞다.
- `L2M`과 `VQ-CD`는 architecture가 다른 reference baseline이므로, IQL/EWC와 같은 범주의 “동일 backbone 비교”로 취급하면 안 된다.
- 특히 `L2M` 원 논문은 여러 task로 pre-training한 backbone을 freeze하고 modulation pool을 학습하는 구조이므로, `A1-pretrained L2M` 혹은 `L2M (A1-pretrained adaptation)`처럼 정확히 적는 편이 과학적으로 더 정직하다.
- `VQ-CD`도 별도 backbone이므로 parameter 수, token exposure, auxiliary memory, wall-clock time을 함께 기록하는 것이 맞다.

## 5) dataset quality confound

### 검증됨

Cross-Embodiment Offline RL 논문은 suboptimal data 비율과 robot diversity가 gradient conflict를 악화시킨다고 보고한다.

### 추론

다만 이것이 곧바로 “A1 dataset은 좋고 H1 dataset은 나쁘다”를 뜻하지는 않는다. dataset quality가 morphology 효과와 얽힐 수 있으므로, 가능한 한 모든 pair에 같은 dataset regime을 유지하고 robot별 transition 수, return 분포, suboptimal 비율을 기록하는 편이 좋다.

## 최종 판단

사용자가 제안한 방향은 전반적으로 타당하다.

1. `target-only single-task baseline`은 forward transfer를 해석하기 위해 사실상 필요하다.
2. `morphology distance`를 실험 변수로 명시하는 것은 이 분야 원문과 잘 맞는다.
3. `learning curve` 저장은 단순 최종 성능보다 훨씬 더 많은 정보를 준다.
4. `L2M/VQ-CD`를 IQL/EWC와 같은 종류로 뭉뚱그리지 말고, backbone-matched vs cross-architecture reference로 나누는 것이 맞다.
5. dataset quality confound는 별도로 기록해야 한다.

내가 하나만 조심하라고 하면, `r = 0.63`과 `r = 0.815`를 같은 주장으로 섞지 않는 것이다. 전자는 morphology similarity vs gradient similarity, 후자는 transfer gain vs gradient similarity다.
