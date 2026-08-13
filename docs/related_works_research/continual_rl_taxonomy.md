# Continual RL 분류 조사 메모

최종 수정: 2026-08-10

## 결론

Continual Reinforcement Learning을 `replay`, `regularization`, `architecture/parameter isolation`로 나누는 방식은 주된 메커니즘을 기준으로 문헌을 정리하기에 타당하다. 다만 세 범주는 상호 배타적이지 않으며, 주요 방법 중에는 둘 이상의 메커니즘을 결합한 hybrid가 많다. Benchmark와 evaluation 연구는 학습 방법과 구분하여 별도의 비방법론 계층으로 두는 것이 적절하다.

이 구분은 현재 repository의 서술 방향과도 일치한다.

- 방법론 연구는 forgetting을 완화하거나 transfer를 보존하는 방법을 설명한다.
- Benchmark 연구는 transfer, forgetting, task difficulty를 무엇으로 측정할지 정의한다.
- 개념 연구는 Continual RL의 정의와 연구 범위를 설명한다.

## 메커니즘 기준 분류

### 1. Replay / Rehearsal

다음 논문들은 replay가 Continual RL의 주요 메커니즘임을 보여준다. 동시에 실제 방법에서는 distillation, recurrent state, world-model learning과 같은 추가 요소가 replay와 함께 사용되는 경우가 많다.

- [Experience Replay for Continual Learning](https://proceedings.neurips.cc/paper_files/paper/2019/hash/fa7cdfad1a5aaf8370ebeda47a1ff1c3-Abstract.html)
  - **저자:** David Rolnick, Arun Ahuja, Jonathan Schwarz, Timothy P. Lillicrap, Gregory Wayne
  - **학회 / 연도:** NeurIPS 2019
  - **문제:** task identity가 주어지지 않는 sequential multi-task RL에서 새로운 학습이 이전 skill을 덮어쓰는 문제를 다룬다.
  - **기여 / 결과:** CLEAR는 새로운 data를 이용한 on-policy learning과 memory의 과거 experience를 이용한 off-policy replay를 결합하고, replay experience에 behavioral cloning을 적용해 이전 policy를 보존한다. Multi-task RL benchmark에서 forgetting을 크게 줄였으며 task label이 필요하지 않음을 보였다.
  - **본 연구와의 차이:** task 사이에 동일한 agent interface를 가정한다. Cross-embodiment 방법이라기보다 stability baseline에 가깝다.

- [The Effectiveness of World Models for Continual Reinforcement Learning](https://proceedings.mlr.press/v232/kessler23a.html)
  - **저자:** Samuel Kessler, Mateusz Ostaszewski, Michał Paweł Bortkiewicz, Mateusz Żarski, Maciej Wolczyk, Jack Parker-Holder, Stephen J. Roberts, Piotr Miłoś
  - **학회 / 연도:** CoLLAs 2023, PMLR 232
  - **문제:** environment stream이 변할 때 world model의 유용성을 계속 유지하는 문제를 다룬다.
  - **기여 / 결과:** World-model agent를 위한 selective experience replay를 분석하고, world model을 continual exploration에 활용하는 task-agnostic 방법인 Continual-Dreamer를 제안한다. MiniGrid와 MiniHack에서 기존 task-agnostic Continual RL 방법보다 높은 sample efficiency와 성능을 보고했다.
  - **본 연구와의 차이:** 변화하는 environment를 다루며 robot embodiment나 observation/action interface의 변화는 고려하지 않는다.

- [Task-Agnostic Continual Reinforcement Learning: Gaining Insights and Overcoming Challenges](https://proceedings.mlr.press/v232/caccia23a.html)
  - **저자:** Massimo Caccia, Jonas Mueller, Taesup Kim, Laurent Charlin, Rasool Fakoor
  - **학회 / 연도:** CoLLAs 2023, PMLR 232
  - **문제:** 제한된 data와 compute, high-dimensional observation 환경에서 task-agnostic Continual RL과 multi-task RL의 차이를 다룬다.
  - **기여 / 결과:** Replay-based Recurrent Reinforcement Learning인 3RL을 제안하고 synthetic task와 Meta-World에서 baseline보다 높은 성능을 보였다.
  - **본 연구와의 차이:** 하나의 고정된 embodiment에서 task가 변하는 stream을 다루며, 서로 다른 robot body 사이의 sequential transfer는 고려하지 않는다.

### 2. Regularization / Consolidation

다음 논문들은 regularization을 독립적인 분류로 둘 수 있음을 보여준다. 다만 RL에 실제로 적용할 때는 architecture나 distillation과 결합된 hybrid가 되는 경우가 많다.

- [Overcoming catastrophic forgetting in neural networks](https://pubmed.ncbi.nlm.nih.gov/28292907/)
  - **저자:** James Kirkpatrick, Razvan Pascanu, Neil Rabinowitz, Joel Veness, Guillaume Desjardins, Andrei A. Rusu, Kieran Milan, John Quan, Tiago Ramalho, Agnieszka Grabska-Barwinska, Demis Hassabis, Claudia Clopath, Dharshan Kumaran, Raia Hadsell
  - **학회 / 연도:** PNAS 2017
  - **문제:** Sequential learning에서 발생하는 catastrophic forgetting을 다룬다.
  - **기여 / 결과:** EWC는 Fisher information으로 parameter importance를 추정하고 중요한 weight의 변화를 제한한다. Sequential classification과 sequential Atari에서 forgetting 완화 효과를 보였다.
  - **본 연구와의 차이:** 범용적인 weight consolidation 방법이며 morphology-aware하지 않고 heterogeneous robot interface를 위해 설계되지 않았다.

- [Continual Reinforcement Learning with Complex Synapses](https://proceedings.mlr.press/v80/kaplanis18a.html)
  - **저자:** Christos Kaplanis, Murray Shanahan, Claudia Clopath
  - **학회 / 연도:** ICML 2018, PMLR 80
  - **문제:** 여러 시간 규모에서 forgetting이 발생하는 Continual RL을 다룬다.
  - **기여 / 결과:** 여러 time constant를 가진 complex synapse로 parameter state를 표현하여 forgetting을 완화하고 experience replay database에 대한 의존도를 낮출 수 있음을 보였다.
  - **본 연구와의 차이:** 생물학적으로 동기화된 consolidation 방법이지만 고정된 agent interface를 가정하며 서로 다른 robot body의 구조를 사용하지 않는다.

- [Progress & Compress: A scalable framework for continual learning](https://proceedings.mlr.press/v80/schwarz18a.html)
  - **저자:** Jonathan Schwarz, Jelena Luketina, Wojciech M. Czarnecki, Agnieszka Grabska-Barwinska, Yee Whye Teh, Razvan Pascanu, Raia Hadsell
  - **학회 / 연도:** ICML 2018, PMLR 80
  - **문제:** 제한된 model capacity에서 task를 순차적으로 학습하는 문제를 다룬다.
  - **기여 / 결과:** 현재 task를 학습하는 active column과 skill을 통합하는 knowledge base를 번갈아 사용하고, distillation으로 이전 task를 보호한다.
  - **본 연구와의 차이:** Regularization과 architecture growth control을 결합한 hybrid이며 안정적인 observation/action interface를 가정한다.

### 3. Architecture / Parameter Isolation

다음 논문들은 head, prompt, mask 또는 task-specific module을 추가하여 task별 capacity를 분리한다. 이 범주는 본 연구의 cross-embodiment 방향과 비교적 가깝지만, 대부분 명시적인 morphology를 transfer에 사용하지 않는다.

- [Same State, Different Task: Continual Reinforcement Learning without Interference](https://ojs.aaai.org/index.php/AAAI/article/view/20674)
  - **저자:** Samuel Kessler, Jack Parker-Holder, Philip Ball, Stefan Zohren, Stephen J. Roberts
  - **학회 / 연도:** AAAI 2022
  - **문제:** 같은 observation space를 공유하더라도 RL task가 서로 양립할 수 없어 하나의 shared policy에서 forgetting 이상의 interference가 발생하는 문제를 다룬다.
  - **기여 / 결과:** OWL은 policy를 shared feature와 task별 head로 분리하고 evaluation 시 bandit 기반 policy selection을 사용한다.
  - **본 연구와의 차이:** 하나의 embodiment 안에서 발생하는 task interference를 다루며 서로 다른 robot morphology 사이의 transfer는 고려하지 않는다.

- [P2DT: Mitigating Forgetting in task-incremental Learning with progressive prompt Decision Transformer](https://arxiv.org/abs/2401.11666)
  - **저자:** Zhiyuan Wang, Xiaoyang Qu, Jing Xiao, Bokui Chen, Jianzong Wang
  - **학회 / 연도:** ICASSP 2024
  - **문제:** Transformer policy를 사용하는 task-incremental offline RL의 forgetting을 다룬다.
  - **기여 / 결과:** 새로운 task가 등장할 때 prompt와 decision token을 확장하여 task별 policy context를 분리한다. Task 수가 증가할 때 forgetting 완화와 scalability를 보고했다.
  - **본 연구와의 차이:** Prompt growth를 이용한 task-specific parameter isolation 방법이지만 morphology-aware하지 않으며 robot 구조를 transfer signal로 사용하지 않는다.

- [Tackling Continual Offline RL through Selective Weights Activation on Aligned Spaces](https://proceedings.neurips.cc/paper_files/paper/2025/hash/4d65fc9de1051c382fd258dbafd8cde9-Abstract-Conference.html)
  - **저자:** Jifeng Hu, Sili Huang, Li Shen, Zhejian Yang, Shengchao Hu, Shisong Tang, Hechang Chen, Lichao Sun, Yi Chang, Dacheng Tao
  - **학회 / 연도:** NeurIPS 2025
  - **문제:** Observation/action space가 서로 다른 heterogeneous continual offline RL을 다룬다.
  - **기여 / 결과:** VQ-CD는 vector quantization으로 서로 다른 state/action space를 정렬한 뒤 sparse task mask를 이용해 unified diffusion policy의 일부 weight만 선택적으로 활성화한다. 동일하거나 서로 다른 space를 포함한 15개 continual-learning task에서 높은 성능을 보고했다.
  - **본 연구와의 차이:** Heterogeneous embodied Continual RL과 가장 가까운 방법이지만, explicit morphology나 kinematic correspondence가 아니라 학습된 latent space에서 정렬한다.

## Benchmark와 Evaluation을 분리해야 하는 이유

Benchmark는 학습 메커니즘이 아니다. Continual RL benchmark는 task stream, evaluation target, metric을 정의하여 transfer와 forgetting이 실제로 개선되었는지를 판단하는 기준을 제공한다. 따라서 방법론 taxonomy와 같은 계층에 두면 분류의 의미가 흐려진다.

- [Continual World: A Robotic Benchmark For Continual Reinforcement Learning](https://papers.nips.cc/paper/2021/hash/ef8446f35513a8d6aa2308357a268a7e-Abstract.html)
  - **저자:** Maciej Wołczyk, Michał Zając, Razvan Pascanu, Łukasz Kuciński, Piotr Miłoś
  - **학회 / 연도:** NeurIPS 2021
  - **문제:** 기존 Continual RL 평가가 forgetting에 치우치고 forward transfer를 충분히 평가하지 않는 문제를 다룬다.
  - **기여 / 결과:** Meta-World 기반 robotic benchmark인 Continual World를 제안하고 forgetting과 함께 forward transfer를 우선적으로 평가해야 한다고 주장한다.
  - **본 연구와의 차이:** 하나의 robot embodiment에서 task가 변하는 benchmark이며 cross-embodiment transfer를 직접 다루지 않는다.

- [Disentangling Transfer in Continual Reinforcement Learning](https://arxiv.org/abs/2209.13900)
  - **저자:** Maciej Wołczyk, Michał Zając, Razvan Pascanu, Łukasz Kuciński, Piotr Miłoś
  - **학회 / 연도:** NeurIPS 2022
  - **문제:** Continual RL에서 transfer를 결정하는 요인이 무엇인지 분석한다.
  - **기여 / 결과:** SAC에서 actor, critic, exploration, data 선택의 영향을 분석하고 Continual World에서 ClonEx-SAC을 평가하여 PackNet보다 높은 transfer와 final success를 보고한다.
  - **본 연구와의 차이:** Transfer evaluation을 구체화하지만 morphology-aware 방법은 아니다.

- [A Definition of Continual Reinforcement Learning](https://arxiv.org/abs/2307.11046)
  - **저자:** David Abel, André Barreto, Benjamin Van Roy, Doina Precup, Hado van Hasselt, Satinder Singh
  - **학회 / 연도:** NeurIPS 2023
  - **문제:** Continual RL 분야에 정밀하고 합의된 정의가 부족한 문제를 다룬다.
  - **기여 / 결과:** Continual RL을 endless adaptation으로 형식화하고 agent를 분석하고 분류하기 위한 언어를 제공한다.
  - **본 연구와의 차이:** Algorithm이 아닌 개념 연구다. 메커니즘 taxonomy보다는 benchmark와 evaluation 범위를 정할 때 유용하다.

## 분류 판단

세 가지 메커니즘 분류는 1차적인 문헌 구조로 적합하지만 명확히 분리된 범주가 아니라 서로 겹치는 방법군으로 설명해야 한다.

명시해야 할 주요 중첩 사례는 다음과 같다.

- CLEAR: replay + behavioral cloning regularization
- SYNERgy: replay + synaptic consolidation
- Progress & Compress: regularization + architecture growth control
- OWL: shared trunk + separate heads + policy selection
- P2DT: prompt expansion + task-specific parameter isolation
- VQ-CD: latent-space alignment + selective weight activation

따라서 Related Works에서는 다음 원칙을 사용한다.

1. Replay / rehearsal을 하나의 메커니즘 family로 둔다.
2. Regularization / consolidation을 별도의 family로 둔다.
3. Architecture / parameter isolation을 별도의 family로 둔다.
4. Benchmark / evaluation은 비방법론 계층으로 분리한다.
5. 실제 논문은 두 family의 경계에 놓일 수 있음을 명시한다.

## 재사용 가능한 요약

Continual RL을 `replay`, `regularization`, `architecture/parameter isolation`로 분류하는 것은 타당하다. 다만 hybrid를 허용하고 benchmark/evaluation을 별도 subsection으로 분리해야 한다. 본 연구와 가장 가까운 경계 사례는 VQ-CD다. VQ-CD는 heterogeneous space와 architecture를 함께 다루지만 explicit morphology를 transfer prior로 사용하지 않고 latent alignment를 학습한다.
