# Morphology-Aware / Cross-Embodiment 분류 조사 메모

최종 수정: 2026-08-10

## 통합 결정

최종 [[overview/related_works|Related Works]]는 [[overview/glossary#Cross-Embodiment Learning|Cross-Embodiment Learning]]을 상위 범위로 사용하고, 그 아래를 reinforcement learning, policy transfer/adaptation, embodiment–controller co-design으로 나눈다. [[overview/glossary#Cross-Embodiment Reinforcement Learning|Cross-Embodiment Reinforcement Learning]]은 이 상위 범위 안에서 RL objective를 사용하는 연구를 가리키는 하위 갈래로 유지한다.

## 상위 범위 명칭의 근거

Morphology-aware control, RL transfer, offline RL, co-design을 하나의 넓은 범위에서 연결하려면 `Cross-Embodiment Learning`이 `Cross-Embodiment Reinforcement Learning`보다 적합하다.

근거는 다음과 같다.

- 관련 문헌은 online RL에 한정되지 않는다.
- 현재 논문들은 morphology-aware control, morphology-conditioned pre-training, 새로운 embodiment에 대한 transfer, heterogeneous robot dataset을 이용한 offline RL, robot co-design에 걸쳐 있다.
- `Cross-Embodiment Reinforcement Learning`만 상위 제목으로 사용하면 TE-RoboNet과 같은 transfer 기반 co-design 연구와 pre-training/transfer 중심 연구를 자연스럽게 포괄하기 어렵다.

Related Works의 권장 구조는 다음과 같다.

1. Morphology-aware multi-embodiment control
2. Cross-embodiment pre-training and transfer
3. Offline / pooled-data setting의 cross-embodiment learning
4. 인접하거나 경계에 있는 연구

## 논문 분류

### Morphology-Aware Multi-Embodiment Control

- **One Policy to Control Them All: Shared Modular Policies for Agent-Agnostic Control**
  - **저자:** Wenlong Huang, Igor Mordatch, Deepak Pathak
  - **학회 / 연도:** ICML 2020
  - **문제:** Observation/action dimension과 morphology가 서로 다른 agent를 하나의 controller로 제어하는 문제를 다룬다.
  - **기여 / 결과:** Shared Modular Policies는 동일한 actuator-level module과 message passing을 사용하여 여러 planar morphology를 제어하고 unseen variant로 일반화한다.
  - **본 연구와의 차이:** 여러 morphology를 동시에 사용하는 joint training이며 sequential arrival이나 forgetting objective를 다루지 않는다.
  - **원문:** https://proceedings.mlr.press/v119/huang20d.html

- **MetaMorph: Learning Universal Controllers with Transformers**
  - **저자:** Agrim Gupta, Linxi "Jim" Fan, Surya Ganguli, Li Fei-Fei
  - **학회 / 연도:** ICLR 2022
  - **문제:** Morphology variation을 입력으로 사용하는 modular robot design space에서 universal controller를 학습하는 문제를 다룬다.
  - **기여 / 결과:** Morphology token을 처리하는 Transformer controller를 제안한다. Large-scale pre-training을 통해 unseen morphology에 대한 zero-shot generalization과 sample-efficient transfer를 보였다.
  - **본 연구와의 차이:** Pre-training과 transfer가 중심이며 이전 embodiment의 성능을 유지해야 하는 continual stream은 다루지 않는다.
  - **원문:** https://metamorph-iclr.github.io/site/

- **AnyMorph: Learning Transferable Polices By Inferring Agent Morphology**
  - **저자:** Brandon Trabucco, Mariano Phielipp, Glen Berseth
  - **학회 / 연도:** ICML 2022
  - **문제:** 명시적으로 설계한 morphology description 없이 새로운 morphology로 policy를 이전하는 문제를 다룬다.
  - **기여 / 결과:** RL objective로부터 morphology embedding을 직접 학습하여 새로운 agent에 대한 zero-shot generalization을 개선한다.
  - **본 연구와의 차이:** Morphology inference를 이용한 transfer가 목적이며 embodiment가 순차적으로 등장하는 학습 설정은 다루지 않는다.
  - **원문:** https://proceedings.mlr.press/v162/trabucco22b.html

- **Universal Morphology Control via Contextual Modulation**
  - **저자:** Zheng Xiong, Jacob Beck, Shimon Whiteson
  - **학회 / 연도:** ICML 2023
  - **문제:** Robot마다 다른 morphology context가 control policy를 어떻게 변화시켜야 하는지 model하는 문제를 다룬다.
  - **기여 / 결과:** Morphology-conditioned hypernetwork와 morphology-only attention modulation을 제안하여 training performance와 zero-shot generalization을 개선한다.
  - **본 연구와의 차이:** Multi-task morphology control이 중심이며 embodiment가 반복적으로 추가되는 Continual Learning benchmark는 다루지 않는다.
  - **원문:** https://proceedings.mlr.press/v202/xiong23a.html

- **One Policy to Run Them All: an End-to-end Learning Approach to Multi-Embodiment Locomotion**
  - **저자:** Nico Bohlinger, Grzegorz Czechmanowski, Maciej Krupka, Piotr Kicki, Krzysztof Walas, Jan Peters, Davide Tateo
  - **학회 / 연도:** CoRL 2024
  - **문제:** 여러 legged embodiment를 하나의 locomotion policy로 제어하고 unseen robot으로 이전하는 문제를 다룬다.
  - **기여 / 결과:** URMA는 morphology-agnostic encoder/decoder를 사용하고 16개 robot에서 학습한다. Simulation과 real robot에 대한 zero-shot/few-shot transfer를 보였다.
  - **본 연구와의 차이:** End-to-end multi-embodiment training과 transfer가 중심이며 sequential learning 이후 이전 embodiment의 성능 보존을 명시적으로 평가하지 않는다.
  - **원문:** https://www.ias.informatik.tu-darmstadt.de/uploads/Team/NicoBohlinger/one_policy_to_run_them_all.pdf

### Cross-Embodiment Pre-Training and Transfer

- **PEAC: Unsupervised Pre-training for Cross-Embodiment Reinforcement Learning**
  - **저자:** Chengyang Ying, Zhongkai Hao, Xinning Zhou, Xuezhou Xu, Hang Su, Xingxing Zhang, Jun Zhu
  - **학회 / 연도:** NeurIPS 2024 Main Conference Track
  - **문제:** Reward-free environment에서 embodiment-aware하고 task-agnostic한 지식을 학습하는 문제를 다룬다.
  - **기여 / 결과:** Cross-Embodiment Unsupervised RL을 정의하고 intrinsic reward를 이용한 PEAC를 제안하여 downstream adaptation과 generalization을 개선한다.
  - **본 연구와의 차이:** Reward-free pre-training 이후 adaptation을 평가하며, embodiment가 반복적으로 추가되면서 forgetting이 발생하는 설정은 다루지 않는다.
  - **원문:** https://papers.nips.cc/paper_files/paper/2024/hash/62203a74e233e933b160711e791e1a02-Abstract-Conference.html

- **Efficient Morphology-Aware Policy Transfer to New Embodiments**
  - **저자:** Michael Przystupa, Hongyao Tang, Glen Berseth, Mariano Phielipp, Santiago Miret, Martin Jägersand, Matthew E. Taylor
  - **학회 / 연도:** Reinforcement Learning Journal 6, RLC 2025
  - **문제:** Pre-trained morphology-aware policy를 적은 parameter와 sample만 사용하여 새로운 embodiment로 이전하는 문제를 다룬다.
  - **기여 / 결과:** Direct layer tuning, adapter, prefix tuning을 비교하고, parameter의 1% 미만을 갱신하는 PEFT로 target performance를 개선할 수 있음을 보였다.
  - **본 연구와의 차이:** `pre-training → single target adaptation` 설정이며 여러 target embodiment가 순차적으로 등장하는 transfer는 다루지 않는다.
  - **원문:** https://rlj.cs.umass.edu/2025/papers/Paper172.html

### Offline / Pooled-Data Setting의 Cross-Embodiment Learning

- **Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets**
  - **저자:** Haruki Abe, Takayuki Osa, Yusuke Mukuta, Tatsuya Harada
  - **학회 / 연도:** ICLR 2026 Poster
  - **문제:** Suboptimal trajectory를 포함한 heterogeneous robot dataset에서 universal control prior를 학습하는 문제를 다룬다.
  - **기여 / 결과:** 16개 legged robot dataset을 구축하고 suboptimal data에서는 offline RL이 behavior cloning보다 높은 성능을 낼 수 있음을 보였다. Embodiment 기반 grouping으로 robot 사이의 gradient conflict를 줄인다.
  - **본 연구와의 차이:** 모든 robot data를 함께 사용하는 pooled offline training이며 순차적 data stream, 과거 data 접근 제한, forgetting 측정을 고려하지 않는다.
  - **원문:** https://iclr.cc/virtual/2026/poster/10010454

## 인접하거나 경계에 있는 연구

- **TE-RoboNet: Transfer Enhanced RoboNet for Sample-Efficient Generation of Robot Co-Designs**
  - **저자:** Kishan Reddy Nagiredla, Arun Kumar A V, Kevin Sebastian Luck, Thommen George Karimpanal, Santu Rana
  - **학회 / 연도:** EWRL 2025 Poster
  - **연관성:** 직접적인 Continual RL benchmark가 아니라 robot co-design 연구다. 그러나 morphology와 DoF가 달라질 때 shared core policy와 morphology-specific adapter를 이용해 지식을 이전한다는 점에서 관련된다.
  - **본 연구와의 차이:** 주된 목적이 과거 embodiment의 return 보존이 아니라 sample-efficient robot design 생성이다.
  - **원문:** https://openreview.net/forum?id=sbjbD8ftCH

## 현재 상위 범위에 대한 판단

현재 논문 집합은 RL에 한정된 section보다 넓은 cross-embodiment 상위 범위를 지지한다.

- Section 제목을 `Cross-Embodiment Reinforcement Learning`으로 한정하면 TE-RoboNet을 배치하기 어렵고 pre-training/transfer 연구의 범위를 충분히 드러내지 못한다.
- `Cross-Embodiment Learning`을 사용하면 morphology-aware control, pre-training, transfer, pooled offline RL, co-design 인접 연구를 인위적인 경계 없이 함께 설명할 수 있다.

## 추후 보강할 수 있는 문헌

현재 section 명칭과 분류를 결정하기 위해 추가 논문이 반드시 필요한 것은 아니다. 이후 morphology-aware control의 역사적 흐름을 더 자세히 설명해야 한다면, 현재 논문보다 앞선 morphology-aware control 또는 transfer 연구를 추가로 조사한다.
