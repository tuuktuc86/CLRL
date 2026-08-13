# Heterogeneous observation/action dimensions 관련 일차 자료 메모

최종 수정: 2026-08-13

이 메모는 `VQ-CD`, `L2M`, `URMA / One Policy to Run Them All`이 서로 다른 observation/action dimension을 어떻게 다루는지, 그리고 Round 1 같은 최소 비교 실험에서 어떤 학습 예산 항목이 분리되어야 하는지를 1차 자료 기준으로 정리한 것이다.

## 검증된 사실

### 1) VQ-CD

- VQ-CD는 continual offline RL에서 task 간 state/action space가 달라도 학습할 수 있도록 만든 방법이다.
- 논문은 구조를 두 부분으로 나눈다.
  - `QSA (Quantized Spaces Alignment)`는 vector quantization으로 서로 다른 task의 state/action space를 하나의 공간으로 맞춘다.
  - `SWA (Selective Weights Activation)`는 task-related sparse mask로 diffusion model의 서로 다른 weight를 선택적으로 활성화한다.
- 논문 소개문은 `unified diffusion model attached by the inverse dynamic model`을 사용한다고 명시한다.
- 공개된 arXiv abstract 기준으로는, 평가 시 원래 task space를 복원하기 위한 decoder/복원 단계가 존재한다.
- 확인한 공식 자료 범위에서는 별도의 공식 code repository를 찾지 못했다. 따라서 현재는 paper만 근거로 정리한다.

근거:

- [arXiv abstract](https://arxiv.org/abs/2410.15698)
- [arXiv HTML/abstract 페이지](https://arxiv.org/abs/2410.15698)

### 2) L2M

- L2M은 frozen pre-trained model의 information flow를 `learnable modulation pool`로 조절해 catastrophic forgetting을 줄이는 방법이다.
- 공식 구현은 Decision Transformer 기반이며, Meta-World / Continual-World / DMControl / Atari / Gym-MuJoCo / ProcGen을 지원한다.
- paper의 MDDT 설명에 따르면, 서로 다른 state/action space를 처리하기 위해 unified state space를 만들고, 사용하지 않는 차원은 `0`으로 padding한다.
- 같은 설명에서 action은 각 dimension별로 tokenization하고, `min-max normalization + 64-bin uniform discretization`으로 autoregressive prediction을 수행한다.
- modulation pool은 learnable keys와 각 Transformer block에 대응하는 modulation matrices로 구성된다.
- query는 frozen pre-trained model의 state-token embedding history를 mean-pooling해서 만든다.
- retrieval은 cosine similarity와 key selection count를 함께 사용한다.
- modulation은 self-attention의 `query/value`와 feed-forward block에 적용된다.
- paper는 L2M을 task-agnostic으로 설명하며, 별도 task별 action head를 쓰지 않는다고 명시한다.
- pre-training은 MT40 + DMC10 데이터로 MDDT를 먼저 학습하는 2단계 전제다.
- 본문은 pre-training backbone을 별도로 만든 뒤 frozen model로 유지하는 구조를 전제로 한다.
- paper는 later experiments에서 `40M` parameter model을 사용한다고 밝힌다.
- 공식 repo README는 continual fine-tuning 예시에서 `steps_per_task=100000`을 사용한다.
- 공식 repo README는 pre-training run과 continual fine-tuning run을 분리해서 제공한다.

근거:

- [NeurIPS 2023 paper page](https://proceedings.neurips.cc/paper_files/paper/2023/hash/77e59fafe99e94f822e79bf9308ec377-Abstract-Conference.html)
- [Paper PDF](https://www.proceedings.com/content/075/075280-1660open.pdf)
- [Official repository](https://github.com/ml-jku/L2M)

### 3) URMA / One Policy to Run Them All

- URMA는 robot-specific observations와 general observations를 분리해 처리한다.
- robot-specific observations는 joint와 foot observations이며, robot마다 개수가 달라질 수 있다.
- 이를 위해 URMA는 joint description vectors를 만든 뒤 attention encoder에 넣는다.
  - joint description vectors는 key 역할
  - joint observations는 value 역할
- foot observations도 같은 attention encoding으로 처리한다.
- 이후 joint latent와 foot latent를 general observations와 concat해서 core network에 넣는다.
- 마지막으로 universal morphology decoder가 core output과 joint descriptions, single joint latents를 짝지어 각 joint에 대한 최종 action을 만든다.
- project page 기준으로 URMA는 16개 robot을 동시에 학습하며, PPO로 robot당 100M simulation steps를 사용한다.
- project page는 unseen robot에 대해 zero-shot generalization을 주장한다.
- 공식 README는 새 robot을 추가할 때 `environment.py`의 robot-specific 변수들, XML/mesh, reward, controller gains, domain randomization을 수정하라고 안내한다.

근거:

- [Project page](https://nico-bohlinger.github.io/one_policy_to_run_them_all_website/)
- [Official repository README](https://github.com/nico-bohlinger/one_policy_to_run_them_all)
- [Policy implementation](https://github.com/nico-bohlinger/one_policy_to_run_them_all/blob/main/one_policy_to_run_them_all/algorithms/uni_ppo/ppo/policy.py)

## 추론

- `L2M`은 heterogeneous dimension 문제를 `zero padding + action tokenization + frozen backbone + task-agnostic modulation`으로 푼다. 즉, 새 task의 차원 자체에 대한 전용 head를 늘리기보다, 기존 backbone을 유지하고 modulation 파라미터만 추가하는 쪽이다.
- `URMA`는 `description-conditioned routing + universal decoder` 구조이므로, 새 robot의 joint 수가 달라져도 shared encoder/decoder를 그대로 쓰는 방향에 가깝다. 다만 이 inference는 project page와 code를 바탕으로 한 해석이며, “unseen dimension용 별도 cold-start head”가 없다고 논문이 직접 문장으로 못 박는 것은 아니다.
- `VQ-CD`는 `QSA + SWA + weight assembling`의 3단계로 보는 것이 가장 안전하다. 실제 Round 1 최소 프로토콜에서는 `QSA 정렬 비용`, `SWA 순차 학습 비용`, `최종 assembly 비용`을 분리해서 기록해야 한다.
- `L2M`의 최소 비교 예산은 `pretraining backbone cost`와 `per-task adaptation cost`를 반드시 분리해야 한다. pretraining 없이 L2M을 비교하면 method 자체의 전제가 깨진다.
- `URMA`는 paper/project page 기준으로는 “jointly train one policy over all robots”가 핵심이므로, Round 1에서 pairwise 비교를 하려면 per-pair adaptation baseline보다 shared multi-embodiment training cost를 별도 컬럼으로 둬야 한다.

## Round 1 최소 예산 해석

### VQ-CD

- 최소한 `QSA pretraining` 한 번과 `SWA sequential training` 한 번이 필요하다.
- 그 뒤 `weights assembling`이 들어간다.
- 따라서 최소 프로토콜에서는 `alignment cost`, `sequential diffusion cost`, `assembly cost`를 합산하되, 세 항을 따로 기록하는 것이 맞다.
- 다만 공개 1차 자료에서 각 단계의 step 수를 명시적으로 확인하지 못했으므로, 현재 문서에서는 `exact step budget`을 단정하지 않는다.

### L2M

- 전제는 `MDDT pretraining on MT40 + DMC10`이다.
- 본문은 pretraining backbone을 별도로 만든 뒤 frozen model로 유지하는 구조를 전제로 한다.
- paper는 later experiments에서 `40M` parameter model을 사용한다고 밝힌다.
- 공식 repo README는 continual fine-tuning 예시에서 `steps_per_task=100000`을 사용한다.
- 따라서 Round 1에서 L2M을 pairwise baseline으로 둘 경우, `backbone pretrain`과 `target adaptation`을 분리해서 집계해야 한다.

### URMA

- project page 기준 학습 예산은 `16 robots × 100M sim steps`다.
- zero-shot transfer가 중심이므로, round1 비교에서는 target-only adaptation보다 `joint training cost` 자체를 baseline 행으로 두는 편이 더 정직하다.

## 이 프로젝트에 대한 함의

- `VQ-CD`와 `L2M`은 둘 다 “새 task에 맞춰 기존 backbone을 조금만 바꾸는” 계열이 아니라, 전제되는 입출력 정렬 방식이 다르다.
- `L2M`은 padding/tokenization으로 heterogeneous dimensions를 공통 DT 입력에 맞춘 뒤 modulation pool로 적응한다.
- `VQ-CD`는 space alignment를 별도 모듈로 두고, 그 위에 task-specific mask를 얹는다.
- `URMA`는 robot description-based routing과 universal decoder로, dimension mismatch를 architecture 안에서 직접 흡수한다.
- 따라서 최소 Round 1 pairwise protocol에서는 세 방법을 같은 축으로 취급하지 말고 다음처럼 분리하는 것이 맞다.
  - `backbone pretrain cost`
  - `dimension alignment / routing cost`
  - `per-task adaptation cost`
  - `assembly / freeze / mask management cost`

## 한 줄 결론

세 방법 모두 heterogeneous dimensions를 다루지만, 처리 방식은 다르다.

- `VQ-CD`는 `alignment + mask + assembly`
- `L2M`은 `zero padding + action tokenization + frozen backbone + modulation pool`
- `URMA`는 `description-conditioned routing + universal decoder`

그래서 Round 1 비교에서는 “성능”만 보지 말고, 각 방법의 전제 학습 비용을 분해해서 기록해야 한다.
