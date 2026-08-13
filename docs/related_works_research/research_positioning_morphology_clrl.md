# Topology-aware Cross-Embodiment Continual RL 포지셔닝 메모

최종 수정: 2026-08-13

## 결론

기존 연구가 cross-embodiment / heterogeneous RL의 일부를 이미 풀었다고 보는 편이 맞다. 다만 대부분은 아래 중 하나에 머문다.

- 여러 morphology를 함께 쓰는 joint training 또는 pre-training
- single target embodiment로의 transfer / adaptation
- heterogeneous observation/action space를 latent space로 정렬한 continual RL
- continual RL에서 replay / regularization / prompt / mask로 forgetting을 줄이는 방법

즉, "cross-embodiment continual offline RL" 전체를 끝냈다고 보기는 어렵다. 현재 남는 공간은 **morphology의 구조적 하위 요소인 topology를 학습 과정에 명시적으로 넣었을 때, continual offline RL의 stability–plasticity trade-off가 실제로 개선되는가**이다.

이 연구의 방어 가능한 핵심 novelty는 다음 한 줄이다.

> 서로 다른 robot embodiment가 순차적으로 도착하는 offline continual learning에서, morphology를 구성하는 topology를 inductive bias로 사용하면 이전 embodiment의 제어 지식을 더 잘 보존하면서 새 embodiment의 학습도 유지할 수 있는가?

이 주장만은 아직 선행 연구가 직접적으로 검증하지 않았다.

## 이미 선행 연구가 보여준 것

### 1. Cross-embodiment transfer / pre-training은 가능하다

- [XIRL: Cross-embodiment Inverse Reinforcement Learning](https://proceedings.mlr.press/v164/zakka22a.html)
  - embodiment 차이가 큰 video demonstration에서도 reward를 학습해 transfer할 수 있음을 보였다.
- [Efficient Morphology-Aware Policy Transfer to New Embodiments](https://rlj.cs.umass.edu/2025/papers/Paper172.html)
  - morphology-aware pretraining 후 target embodiment로의 PEFT transfer가 sample-efficient함을 보였다.
- [Embedding Morphology into Transformers for Cross-Robot Policy Learning](https://arxiv.org/abs/2603.00182)
  - Robot morphology를 encoding한 token을 transformer에 넣어 cross-robot policy learning을 강화했다.

이 축의 공통점은 "새 embodiment로 옮겨갈 수 있는가"에 초점이 있고, **이전 embodiment의 성능을 유지해야 하는 continual setting**은 정면으로 다루지 않는다는 점이다.

### 2. Continual RL은 이미 mature한 축이지만, 보통 embodiment는 고정된다

- [Continual World](https://proceedings.neurips.cc/paper_files/paper/2021/hash/ef8446f35513a8d6aa2308357a268a7e-Abstract.html)
- [Disentangling Transfer in Continual Reinforcement Learning](https://proceedings.neurips.cc/paper_files/paper/2022/hash/2938ad0434a6506b125d8adaff084a4a-Abstract-Conference.html)
- [P2DT: Mitigating Forgetting in task-incremental Learning with progressive prompt Decision Transformer](https://arxiv.org/abs/2401.11666)
- [Solving Continual Offline Reinforcement Learning through Selective Weights Activation on Aligned Spaces](https://arxiv.org/abs/2410.15698)

이 축의 공통점은 forgetting / transfer / sparse activation / latent alignment를 다루지만, **robot morphology 자체를 continual learning의 핵심 구조로 사용하지 않는다**는 점이다.

### 3. Heterogeneous / pooled multi-robot offline RL은 있지만 continual retention은 약하다

- [Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets](https://arxiv.org/abs/2602.18025)
  - 16개 robot dataset을 pooled offline RL로 학습하고, morphology similarity에 따른 gradient conflict를 줄이는 grouping을 제안했다.

이 논문은 우리 문제와 가장 가깝지만, 여전히 **순차 도착 + 과거 embodiment 성능 보존**이라는 continual objective는 아니다.

## 우리 연구가 주장할 수 있는 것

다음은 충분히 방어 가능한 주장이다.

1. Morphology와 그 구조적 하위 요소인 topology는 단순 메타데이터가 아니라, cross-embodiment continual learning에서 transfer 가능성과 interference 가능성을 가르는 prior가 될 수 있다.
2. Explicit topology를 continual learning mechanism에 사용하면, 같은 compute/data budget에서 forward transfer와 retention의 균형이 개선될 수 있다.
3. 이 효과는 모든 pair에서 항상 동일하지 않을 수 있고, morphology가 비슷한 pair에서 더 강하게 나타날 가능성이 있다.

즉, 우리의 주장은 "topology를 넣으면 CL이 자동으로 해결된다"가 아니라,

> topology-aware inductive bias가 cross-embodiment continual offline RL의 stability–plasticity trade-off를 실제로 개선하는지 검증한다

가 되어야 한다.

## 과장하면 안 되는 주장

다음은 현재 단계에서 위험하다.

- "기존 cross-embodiment RL은 해결되지 않았다"
  - 틀리다. transfer / pre-training / pooled multi-robot learning은 이미 있다.
- "topology를 넣으면 항상 더 좋다"
  - 아직 검증되지 않았다.
- "모든 robot pair에서 동일한 개선이 나온다"
  - morphology distance에 따라 결과가 달라질 가능성이 크다.
- "VQ-CD/EWC/replay보다 본 방법이 본질적으로 우월하다"
  - baseline 대비 실험 결과가 나와야만 말할 수 있다.

## 실험적으로 성립해야 하는 최소 조건

우리 연구의 장점이 성립하려면 적어도 아래가 보여야 한다.

1. `source → target` pairwise setting에서, topology-aware 방법이 Seq-FT 대비 target acquisition을 유지하거나 개선해야 한다.
2. 같은 조건에서 source embodiment 성능 저하가 EWC / other CL baseline보다 작아야 한다.
3. parameter 수나 data exposure가 비슷한 baseline과 비교해도 이득이 남아야 한다.
4. 적어도 하나의 가까운 morphology pair와 하나의 먼 pair에서 topology-aware inductive bias의 효과를 해석할 수 있어야 한다.

## 포지셔닝 한 줄

이 연구는 "cross-embodiment continual RL을 처음 푸는 것"이 아니라, **heterogeneous robot embodiment가 순차적으로 도착하는 offline continual learning에서 morphology의 topology를 이용한 inductive bias가 실제로 retention과 transfer를 동시에 개선하는지 검증하는 것**이다.
