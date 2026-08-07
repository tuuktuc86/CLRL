# Morphology-Aware / Cross-Embodiment Taxonomy Note

## Integration decision

최종 [[overview/related_works|Related Works]]는 **Cross-Embodiment Learning**을 상위 범위로 사용하고, 그 아래를 reinforcement learning, policy transfer/adaptation, embodiment–controller co-design으로 나눈다. **Cross-Embodiment Reinforcement Learning**은 이 상위 범위 안에서 RL objective를 사용하는 연구를 가리키는 하위 갈래로 유지한다.

## Rationale for the umbrella

Morphology-aware control, RL transfer, offline RL, co-design을 하나의 넓은 범위에서 연결하려면 **Cross-Embodiment Learning**이 **Cross-Embodiment Reinforcement Learning**보다 적합하다.

Reason:

- The relevant literature in this cluster is not limited to online RL.
- The papers span morphology-aware control, morphology-conditioned pre-training, transfer to new embodiments, offline RL on heterogeneous robot datasets, and robot co-design.
- `Cross-Embodiment Reinforcement Learning` is too narrow for TE-RoboNet-style transfer-enabled co-design and for the pre-training / transfer papers that are central to the narrative.

Recommended structure for the related-works draft:

1. Morphology-aware multi-embodiment control
2. Cross-embodiment pre-training and transfer
3. Cross-embodiment learning in offline / pooled-data settings
4. Adjacent boundary cases

## Paper classification

### Morphology-aware multi-embodiment control

- **One Policy to Control Them All: Shared Modular Policies for Agent-Agnostic Control**  
  **Authors:** Wenlong Huang, Igor Mordatch, Deepak Pathak  
  **Venue / year:** ICML 2020  
  **Problem:** Learn a single controller that handles different morphologies with different observation/action dimensions.  
  **Contribution / result:** Shared Modular Policies (SMP) uses identical actuator-level modules plus message passing to control multiple planar morphologies and generalize to unseen variants.  
  **Why it differs from sequential cross-embodiment continual RL:** joint training over multiple morphologies, not sequential arrival; no forgetting objective.  
  **Primary source:** https://proceedings.mlr.press/v119/huang20d.html

- **MetaMorph: Learning Universal Controllers with Transformers**  
  **Authors:** Agrim Gupta, Linxi "Jim" Fan, Surya Ganguli, Li Fei-Fei  
  **Venue / year:** ICLR 2022  
  **Problem:** Learn a universal controller over a modular robot design space with morphology variation as an input modality.  
  **Contribution / result:** Transformer-based controller over morphology tokens; large-scale pre-training gives zero-shot generalization to unseen morphologies and sample-efficient transfer.  
  **Why it differs from sequential cross-embodiment continual RL:** pre-training and transfer, not a continual stream with preserved performance across earlier embodiments.  
  **Primary source:** https://metamorph-iclr.github.io/site/

- **AnyMorph: Learning Transferable Polices By Inferring Agent Morphology**  
  **Authors:** Brandon Trabucco, Mariano Phielipp, Glen Berseth  
  **Venue / year:** ICML 2022  
  **Problem:** Transfer a policy to new morphologies without requiring an explicit hand-designed morphology description.  
  **Contribution / result:** Learns a morphology embedding directly from the RL objective and improves zero-shot generalization to new agents.  
  **Why it differs from sequential cross-embodiment continual RL:** target is morphology inference for transfer, not learning under a sequential embodiment stream.  
  **Primary source:** https://proceedings.mlr.press/v162/trabucco22b.html

- **Universal Morphology Control via Contextual Modulation**  
  **Authors:** Zheng Xiong, Jacob Beck, Shimon Whiteson  
  **Venue / year:** ICML 2023  
  **Problem:** Model how morphology context changes the control policy across robots.  
  **Contribution / result:** Introduces morphology-conditioned hypernetworks plus morphology-only attention modulation; improves training performance and zero-shot generalization.  
  **Why it differs from sequential cross-embodiment continual RL:** multi-task morphology control, not a continual learning benchmark with repeated embodiment additions.  
  **Primary source:** https://proceedings.mlr.press/v202/xiong23a.html

- **One Policy to Run Them All: an End-to-end Learning Approach to Multi-Embodiment Locomotion**  
  **Authors:** Nico Bohlinger, Grzegorz Czechmanowski, Maciej Krupka, Piotr Kicki, Krzysztof Walas, Jan Peters, Davide Tateo  
  **Venue / year:** CoRL 2024  
  **Problem:** Learn one locomotion policy that can handle many legged embodiments and transfer to unseen robots.  
  **Contribution / result:** URMA provides morphology-agnostic encoders/decoders, trains on 16 robots, and transfers zero-/few-shot to simulated and real robots.  
  **Why it differs from sequential cross-embodiment continual RL:** end-to-end multi-embodiment training and transfer; no explicit sequential embodiment retention test.  
  **Primary source:** https://www.ias.informatik.tu-darmstadt.de/uploads/Team/NicoBohlinger/one_policy_to_run_them_all.pdf

### Cross-embodiment pre-training and transfer

- **PEAC: Unsupervised Pre-training for Cross-Embodiment Reinforcement Learning**  
  **Authors:** Chengyang Ying, Zhongkai Hao, Xinning Zhou, Xuezhou Xu, Hang Su, Xingxing Zhang, Jun Zhu  
  **Venue / year:** NeurIPS 2024 Main Conference Track  
  **Problem:** Learn embodiment-aware, task-agnostic knowledge in reward-free environments.  
  **Contribution / result:** Defines Cross-Embodiment Unsupervised RL and PEAC, which adds an intrinsic reward for cross-embodiment pre-training and improves adaptation / generalization.  
  **Why it differs from sequential cross-embodiment continual RL:** pre-training in reward-free settings, not repeated incremental embodiment learning with forgetting evaluation.  
  **Primary source:** https://papers.nips.cc/paper_files/paper/2024/hash/62203a74e233e933b160711e791e1a02-Abstract-Conference.html

- **Efficient Morphology-Aware Policy Transfer to New Embodiments**  
  **Authors:** Michael Przystupa, Hongyao Tang, Glen Berseth, Mariano Phielipp, Santiago Miret, Martin Jägersand, Matthew E. Taylor  
  **Venue / year:** Reinforcement Learning Journal, vol. 6, 2025; presented at RLC 2025  
  **Problem:** Adapt a pretrained morphology-aware policy to a new embodiment with fewer parameters and fewer samples.  
  **Contribution / result:** Compares direct layer tuning, adapters, and prefix tuning; shows PEFT can improve target performance with less than 1% of parameters.  
  **Why it differs from sequential cross-embodiment continual RL:** single-target adaptation after pre-training, not sequential transfer across multiple new embodiments.  
  **Primary source:** https://rlj.cs.umass.edu/2025/papers/Paper172.html

### Cross-embodiment learning in pooled / offline settings

- **Cross-Embodiment Offline Reinforcement Learning for Heterogeneous Robot Datasets**  
  **Authors:** Haruki Abe, Takayuki Osa, Yusuke Mukuta, Tatsuya Harada  
  **Venue / year:** ICLR 2026 Poster  
  **Problem:** Use heterogeneous robot datasets, including suboptimal trajectories, to learn a universal control prior.  
  **Contribution / result:** Builds a 16-platform locomotion suite, shows offline RL can outperform behavior cloning on suboptimal data, and reduces inter-robot gradient conflict with embodiment-based grouping.  
  **Why it differs from sequential cross-embodiment continual RL:** pooled offline training over all robot data, not a sequential stream with access constraints or forgetting measurement.  
  **Primary source:** https://iclr.cc/virtual/2026/poster/10010454

## Adjacent / boundary case

- **TE-RoboNet: Transfer Enhanced RoboNet for Sample-Efficient Generation of Robot Co-Designs**  
  **Authors:** Kishan Reddy Nagiredla, Arun Kumar A V, Kevin Sebastian Luck, Thommen George Karimpanal, Santu Rana  
  **Venue / year:** EWRL 2025 Poster  
  **Why it is adjacent:** the paper is about robot co-design, not a direct continual-RL benchmark, but it is still relevant because it transfers a shared core policy plus morphology-specific adapters across changing morphologies and DoFs.  
  **Why it differs from sequential cross-embodiment continual RL:** co-design is the primary objective, and the paper is about sample-efficient generation of designs rather than preserving prior embodiment returns under a sequential learning protocol.  
  **Primary source:** https://openreview.net/forum?id=sbjbD8ftCH

## Bottom-line interpretation for the current umbrella

The current paper set supports a **broad cross-embodiment umbrella** rather than a strict RL-only section.

- If the section is titled `Cross-Embodiment Reinforcement Learning`, TE-RoboNet becomes awkward to place and the narrative under-reads the transfer / pre-training papers.
- If the section is titled `Cross-Embodiment Learning`, the draft can cleanly include morphology-aware control, pre-training, transfer, pooled offline RL, and the co-design adjacency without forcing an artificial boundary.

## Optional next additions, if the draft needs a stronger historical bridge

No extra paper is strictly required for the section-title decision above. If a later rewrite wants a denser lineage for morphology-aware control, the next papers to consider are older morphology-aware control / transfer anchors that sit just outside the current requested set.
