# Continual RL Taxonomy Notes

Last updated: 2026-08-07

## Bottom line

The replay / regularization / architecture-parameter-isolation taxonomy is defensible as a coarse, mechanism-first way to organize continual reinforcement learning. It is not cleanly mutually exclusive, though: many strong papers are hybrids, and benchmark / evaluation papers are better treated as a separate non-method layer.

That split matches the current repository framing:
- method families explain how forgetting is reduced or transfer is preserved;
- benchmark papers explain what counts as transfer, forgetting, and task difficulty;
- conceptual papers explain the definition and scope of continual RL itself.

## Mechanism taxonomy

### 1) Replay / rehearsal

These papers support replay as a primary continual-RL mechanism, but they also show that replay is usually strengthened by a second ingredient such as distillation, recurrent state, or world-model learning.

- [Experience Replay for Continual Learning](https://proceedings.neurips.cc/paper_files/paper/2019/hash/fa7cdfad1a5aaf8370ebeda47a1ff1c3-Abstract.html)  
  Authors: David Rolnick, Arun Ahuja, Jonathan Schwarz, Timothy P. Lillicrap, Gregory Wayne  
  Venue / year: NeurIPS 2019  
  Problem: sequential multi-task RL without task identity, where new learning overwrites older skills.  
  Contribution / result: CLEAR combines on-policy learning from new data with off-policy replay from memory, then adds behavioral cloning on replayed experience to better preserve the old policy. The paper shows strong forgetting reduction on multi-task RL benchmarks and demonstrates that task labels are not required.  
  Difference for this project: the setting is still a fixed agent interface across tasks. It is a stability baseline, not a cross-embodiment method.

- [The Effectiveness of World Models for Continual Reinforcement Learning](https://proceedings.mlr.press/v232/kessler23a.html)  
  Authors: Samuel Kessler, Mateusz Ostaszewski, Michał Paweł Bortkiewicz, Mateusz Żarski, Maciej Wolczyk, Jack Parker-Holder, Stephen J. Roberts, Piotr Miłoś  
  Venue / year: The 2nd Conference on Lifelong Learning Agents (CoLLAs 2023), PMLR 232  
  Problem: how to keep a world model useful as the environment stream changes.  
  Contribution / result: the paper studies selective experience replay for world-model agents and proposes Continual-Dreamer, a task-agnostic method that uses the world model for continual exploration. It reports strong sample efficiency and better performance than prior task-agnostic continual-RL methods on MiniGrid and MiniHack.  
  Difference for this project: it is about changing environments, not changing robot embodiments or observation/action interfaces.

- [Task-Agnostic Continual Reinforcement Learning: Gaining Insights and Overcoming Challenges](https://proceedings.mlr.press/v232/caccia23a.html)  
  Authors: Massimo Caccia, Jonas Mueller, Taesup Kim, Laurent Charlin, Rasool Fakoor  
  Venue / year: The 2nd Conference on Lifelong Learning Agents (CoLLAs 2023), PMLR 232  
  Problem: task-agnostic continual RL versus multi-task RL under limited data, compute, and high-dimensional observations.  
  Contribution / result: the paper introduces replay-based recurrent reinforcement learning (3RL) and shows it can outperform baselines on synthetic tasks and Meta-World.  
  Difference for this project: again, it is a single embodied task stream, not sequential transfer across distinct bodies.

### 2) Regularization / consolidation

These papers support the regularization bucket, but they also show that regularization often becomes a hybrid once it is made practical for RL.

- [Overcoming catastrophic forgetting in neural networks](https://pubmed.ncbi.nlm.nih.gov/28292907/)  
  Authors: James Kirkpatrick, Razvan Pascanu, Neil Rabinowitz, Joel Veness, Guillaume Desjardins, Andrei A. Rusu, Kieran Milan, John Quan, Tiago Ramalho, Agnieszka Grabska-Barwinska, Demis Hassabis, Claudia Clopath, Dharshan Kumaran, Raia Hadsell  
  Venue / year: PNAS 2017  
  Problem: sequential learning without catastrophic forgetting.  
  Contribution / result: EWC estimates parameter importance with the Fisher information and penalizes changes to important weights. The paper shows it works on sequential classification and sequential Atari.  
  Difference for this project: it is a generic weight-consolidation method, not morphology-aware and not designed for heterogeneous robot interfaces.

- [Continual Reinforcement Learning with Complex Synapses](https://proceedings.mlr.press/v80/kaplanis18a.html)  
  Authors: Christos Kaplanis, Murray Shanahan, Claudia Clopath  
  Venue / year: ICML 2018, PMLR 80  
  Problem: continual RL with forgetting at multiple timescales.  
  Contribution / result: complex synapses model the parameter state with multiple time constants, which mitigates forgetting and can reduce the need for an experience replay database.  
  Difference for this project: it is biologically motivated consolidation, but it still assumes a fixed agent interface and does not use structure of different robot bodies.

- [Progress & Compress: A scalable framework for continual learning](https://proceedings.mlr.press/v80/schwarz18a.html)  
  Authors: Jonathan Schwarz, Jelena Luketina, Wojciech M. Czarnecki, Agnieszka Grabska-Barwinska, Yee Whye Teh, Razvan Pascanu, Raia Hadsell  
  Venue / year: ICML 2018, PMLR 80  
  Problem: sequential tasks with bounded capacity.  
  Contribution / result: P&C alternates an active column for the current task with a knowledge base for consolidated skills, using distillation to protect earlier tasks.  
  Difference for this project: this is a hybrid of regularization and architecture growth control, not a pure penalty-based method, and it still assumes a stable interface.

### 3) Architecture / parameter-isolation

These papers isolate task-specific capacity by adding heads, prompts, masks, or task-specific modules. They are the closest bucket to the project’s cross-embodiment direction, but most of them still stop short of explicit morphology-aware transfer.

- [Same State, Different Task: Continual Reinforcement Learning without Interference](https://ojs.aaai.org/index.php/AAAI/article/view/20674)  
  Authors: Samuel Kessler, Jack Parker-Holder, Philip Ball, Stefan Zohren, Stephen J. Roberts  
  Venue / year: AAAI 2022  
  Problem: RL tasks can be incompatible even when they share the same observation space, so a single shared policy can interfere rather than merely forget.  
  Contribution / result: OWL factorizes the policy into shared features plus separate task heads, and uses bandit-based policy selection at test time.  
  Difference for this project: it is about task interference in one embodiment, not about transferring across different robot morphologies.

- [P2DT: Mitigating Forgetting in task-incremental Learning with progressive prompt Decision Transformer](https://arxiv.org/abs/2401.11666)  
  Authors: Zhiyuan Wang, Xiaoyang Qu, Jing Xiao, Bokui Chen, Jianzong Wang  
  Venue / year: ICASSP 2024  
  Problem: forgetting in task-incremental offline RL with transformer policies.  
  Contribution / result: P2DT appends new prompt / decision tokens as tasks arrive, so each task can grow its own prompt-based policy context. The paper reports reduced forgetting and scalability as task count increases.  
  Difference for this project: prompt growth is a parameter-isolation mechanism for task increments, but it is not morphology-aware and does not use robot structure as the transfer signal.

- [Tackling Continual Offline RL through Selective Weights Activation on Aligned Spaces](https://proceedings.neurips.cc/paper_files/paper/2025/hash/4d65fc9de1051c382fd258dbafd8cde9-Abstract-Conference.html)  
  Authors: Jifeng Hu, Sili Huang, Li Shen, Zhejian Yang, Shengchao Hu, Shisong Tang, Hechang Chen, Lichao Sun, Yi Chang, Dacheng Tao  
  Venue / year: NeurIPS 2025  
  Problem: continual offline RL with heterogeneous observation and action spaces.  
  Contribution / result: VQ-CD first aligns different state/action spaces with vector quantization, then uses sparse task masks to selectively activate weights in a unified diffusion policy. It reports SOTA on 15 continual-learning tasks across both same-space and different-space settings.  
  Difference for this project: this is the closest match to heterogeneous embodied continual RL, but the alignment is learned in latent space rather than derived from explicit morphology or kinematic correspondence.

## Why benchmark / evaluation should be separate

The benchmark layer is not a learning mechanism, and treating it as one blurs the taxonomy. In continual RL, benchmark papers define the task stream, the evaluation target, and the metrics that determine whether transfer or forgetting is actually improving.

- [Continual World: A Robotic Benchmark For Continual Reinforcement Learning](https://papers.nips.cc/paper/2021/hash/ef8446f35513a8d6aa2308357a268a7e-Abstract.html)  
  Authors: Maciej Wołczyk, Michał Zając, Razvan Pascanu, Łukasz Kuciński, Piotr Miłoś  
  Venue / year: NeurIPS 2021  
  Problem: existing continual-RL evaluation focused too much on forgetting and not enough on forward transfer.  
  Contribution / result: Continual World introduces a robotic benchmark on top of Meta-World and argues that forward transfer must be prioritized alongside forgetting.  
  Difference for this project: it is a benchmark for a single robot embodiment, so it does not address cross-embodiment transfer directly.

- [Disentangling Transfer in Continual Reinforcement Learning](https://arxiv.org/abs/2209.13900)  
  Authors: Maciej Wołczyk, Michał Zając, Razvan Pascanu, Łukasz Kuciński, Piotr Miłoś  
  Venue / year: NeurIPS 2022  
  Problem: what drives transfer in continual RL.  
  Contribution / result: the paper studies actor, critic, exploration, and data choices in SAC and evaluates ClonEx-SAC on Continual World, reporting better transfer and final success than PackNet.  
  Difference for this project: it sharpens evaluation and transfer analysis, but it is still not a morphology-aware method.

- [A Definition of Continual Reinforcement Learning](https://arxiv.org/abs/2307.11046)  
  Authors: David Abel, André Barreto, Benjamin Van Roy, Doina Precup, Hado van Hasselt, Satinder Singh  
  Venue / year: NeurIPS 2023  
  Problem: the field lacked a precise definition of continual RL.  
  Contribution / result: the paper formalizes continual RL as endless adaptation and provides a language for analyzing and cataloging agents.  
  Difference for this project: conceptual, not algorithmic; useful for scoping the benchmark and evaluation section, not for mechanism taxonomy.

## Taxonomy verdict

The three mechanism buckets are a good first-order organization, but they should be described as overlapping families, not as hard partitions.

Practical overlaps worth calling out explicitly:
- CLEAR = replay + behavioral cloning regularization.
- SYNERgy = replay + synaptic consolidation.
- Progress & Compress = regularization + architecture growth control.
- OWL = shared trunk + separate heads + policy selection.
- P2DT = prompt expansion / task-specific parameter isolation.
- VQ-CD = latent-space alignment + selective weight activation.

That means the write-up should say:
1. replay / rehearsal is one mechanism family;
2. regularization / consolidation is another;
3. architecture / parameter-isolation is another;
4. benchmark / evaluation is a separate non-method layer;
5. real papers often sit on the boundary between two families.

## Short reusable takeaway

Yes, the replay / regularization / architecture-parameter-isolation taxonomy is defensible for continual RL, but only if the note explicitly allows hybrids and moves benchmark / evaluation into its own subsection. For this project, the most relevant boundary case is VQ-CD: it is architecture-heavy and heterogeneous-space aware, but it still learns latent alignment rather than using explicit morphology as the transfer prior.
