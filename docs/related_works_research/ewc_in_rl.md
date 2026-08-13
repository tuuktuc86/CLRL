# EWC 적용 위치에 관한 Continual RL 문헌 조사

## 조사 질문

Actor, critic, value network가 분리된 reinforcement learning algorithm에서 Elastic Weight Consolidation(EWC)을 어느 parameter에 적용해 왔으며, IQL 위에 EWC를 구현하는 것이 합리적인지 조사한다.

## 문헌별 적용 방식

| 연구 | 기반 RL algorithm | EWC 적용 대상 | 해석 |
| --- | --- | --- | --- |
| *Overcoming catastrophic forgetting in neural networks* (Kirkpatrick et al., PNAS 2017) | Atari DQN | Q-network | Actor가 없는 value-based algorithm이므로 EWC가 Q-network 자체를 보호한다. 이 결과를 actor–critic의 모든 network에 그대로 일반화할 수는 없다. |
| *Progress & Compress* (Schwarz et al., ICML 2018) | Distributed actor–critic | Knowledge base의 shared representation과 policy/value parameter | Policy와 value function이 convolutional encoder를 공유하는 구조에 online EWC를 사용한다. Shared encoder 때문에 actor-only와 critic-only 효과를 명확히 분리하기 어렵다. |
| *Continual World* (Wołczyk et al., NeurIPS 2021) | SAC | Main setting은 actor 중심 regularization; critic regularization은 별도 분석 | Critic을 자유롭게 적응시키는 편이 유리했고, EWC로 critic까지 regularize한 설정은 성능이 저하되었다. |
| *Disentangling Transfer in Continual Reinforcement Learning* (Wołczyk et al., NeurIPS 2022) | SAC | Actor와 critic의 전달·재초기화를 분리해 분석 | Critic은 forward transfer에 중요한 역할을 하므로, 이를 지나치게 고정하는 것이 새 task의 acquisition을 방해할 수 있음을 보여준다. |

## IQL 위 EWC에 대한 판단

EWC는 독립된 RL algorithm이라기보다 기존 학습 objective에 parameter-consolidation penalty를 추가하는 continual learning mechanism이다. 따라서 IQL 위에 구현하는 것 자체는 합리적이다. 다만 `EWC-IQL`이라는 표준 구성이 정해진 것은 아니므로 적용 대상을 명시해야 한다.

Round 1의 주 baseline으로는 `IQL + actor-only EWC`를 권장한다.

1. IQL actor는 advantage-weighted log-likelihood로 학습되는 명시적 stochastic policy이므로 policy Fisher를 정의하기 쉽다.
2. 현재 설계에서는 actor, Q, V encoder가 parameter를 공유하지 않으므로 actor만 보호하는 효과를 분리할 수 있다.
3. Q와 V는 학습 중 target robot의 value structure에 빠르게 적응할 필요가 있으며, Continual World의 결과는 critic regularization이 plasticity를 크게 낮출 수 있음을 보여준다.
4. 실행 시 사용하는 것은 actor이므로, 이전 robot의 행동 능력 보존이라는 EWC의 목적과 actor protection이 직접 연결된다.

첫 robot A1 학습 후 actor parameter \(\theta^*\)와 diagonal policy Fisher를 저장하고, target robot의 actor loss에 다음 penalty를 추가한다.

$$
L_{actor}^{target}
+
\frac{\lambda}{2}
\sum_j F_j(\theta_j-\theta_j^*)^2.
$$

Main Fisher는 이전 policy의 log-likelihood로 계산하는 vanilla policy Fisher를 사용한다.

$$
F_j =
\mathbb{E}_{s\sim D_{A1},\,a\sim\pi_{A1}(\cdot\mid s)}
\left[
\left(
\frac{\partial \log \pi_{A1}(a\mid s)}{\partial \theta_j}
\right)^2
\right].
$$

IQL advantage weight를 Fisher에 다시 곱하는 방식은 가능한 변형이지만 vanilla EWC가 아니므로 main baseline에는 사용하지 않는다.

## Pairwise Round 1에 미치는 영향

`A1 -> target`의 두-task 실험에서는 보호할 과거 task가 A1 하나뿐이다. 따라서 online Fisher decay는 식별할 수 없고 tuning할 필요가 없다. Round 1에서는 \(\lambda\)만 tuning하는 편이 적절하다. 여러 과거 robot의 Fisher를 누적하는 full sequence 실험에서만 online EWC decay를 도입한다.

Actor-only EWC가 유일하게 옳은 선택이라는 뜻은 아니다. 이후 하나의 대표 전환에서 다음 ablation을 수행할 수 있다.

- actor-only EWC
- actor + Q/V EWC
- actor encoder-only EWC

단, critic의 regression loss gradient를 Fisher라고 부르려면 likelihood model을 별도로 가정해야 한다. Continual World는 critic output을 unit-variance Gaussian으로 간주한 실험을 수행했으나, 이를 actor의 policy Fisher와 동일한 의미로 해석해서는 안 된다.

## 실험 설계 권고

- Baseline 이름을 `IQL + actor-only EWC`로 명시한다.
- A1 Fisher는 seed별로 한 번 계산하여 Go1, H1, Hexapod branch에서 재사용한다.
- Pairwise Round 1에서는 online Fisher decay를 제거하고 \(\lambda\)만 탐색한다.
- Fisher sample 수는 우선 약 10k transitions로 시작하고 추정 안정성을 확인한다. 현재 계획의 `100 x 1024`는 pilot 단계에는 과도할 가능성이 있다.
- Critic regularization은 main configuration이 아니라 후속 ablation으로 둔다.

## Primary Sources

- Kirkpatrick et al., [Overcoming catastrophic forgetting in neural networks](https://www.pnas.org/doi/10.1073/pnas.1611835114), PNAS 2017.
- Schwarz et al., [Progress & Compress: A Scalable Framework for Continual Learning](https://proceedings.mlr.press/v80/schwarz18a.html), ICML 2018.
- Wołczyk et al., [Continual World: A Robotic Benchmark for Continual Reinforcement Learning](https://proceedings.neurips.cc/paper/2021/hash/ef8446f35513a8d6aa2308357a268a7e-Abstract.html), NeurIPS 2021; [Supplement](https://proceedings.neurips.cc/paper_files/paper/2021/file/ef8446f35513a8d6aa2308357a268a7e-Supplemental.pdf).
- Wołczyk et al., [Disentangling Transfer in Continual Reinforcement Learning](https://proceedings.neurips.cc/paper/2022/hash/2938ad0434a6506b125d8adaff084a4a-Abstract-Conference.html), NeurIPS 2022.
