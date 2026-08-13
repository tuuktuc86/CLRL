# Round 1 Baseline 구현 근거

이 문서는 Round 1에서 사용할 IQL, EWC, L2M과 VQ-CD의 구현 계획을 일차 자료와 공식 구현을 기준으로 정리한다. 실제 실행 규칙과 축소된 hyperparameter 범위는 [Round 1 Experiments Guideline](../experiments/experiments_1/experiments1_guideline.md)을 따른다.

## 명칭 확인

확인한 일차 자료에서는 continual RL baseline으로서 `L2C`라는 명칭을 찾지 못했다. 따라서 현재 계획에서는 이를 앞서 논의한 `L2M (Learning to Modulate)`의 오기로 해석한다. 별도의 L2C 논문을 의미한 경우 해당 논문을 확인한 뒤 교체해야 한다.

## IQL

### 일차 자료

- [Official implementation](https://github.com/ikostrikov/implicit_q_learning)
- [Official MuJoCo configuration](https://raw.githubusercontent.com/ikostrikov/implicit_q_learning/master/configs/mujoco_config.py)
- [Paper](https://arxiv.org/abs/2110.06169)
- [ICLR 2022 page](https://openreview.net/forum?id=68n2s9ZJWF8)
- [TD3+BC official implementation](https://github.com/sfujim/TD3_BC/blob/main/TD3_BC.py)

### 구현에 반영할 내용

- 공식 구현은 JAXRL을 기반으로 한 offline RL 구현이며 offline training과 fine-tuning entry point를 제공한다.
- IQL은 expectile value regression, dataset action만 사용하는 Q update와 advantage-weighted behavior cloning으로 policy를 추출한다.
- 공개 locomotion configuration의 주요 값은 learning rate `3e-4`, batch size `256`, hidden dims `(256, 256)`, discount `0.99`, expectile `0.7`, advantage temperature `3.0`, target update coefficient `0.005`다.
- 공식 offline training entry point의 기본 budget은 `1M` gradient updates다. Round 1의 `50k` budget은 재현 설정이 아니라 최소 screening 설정이므로, curve가 계속 상승하면 budget 부족을 먼저 보고해야 한다.
- 대표적인 다른 D4RL offline RL 방법인 TD3+BC의 공식 actor와 twin critic도 각각 두 개의 256-unit hidden layer를 사용한다. 따라서 `2 x 256`은 IQL에만 특이한 작은 설정이라기보다 당시 vector-state offline RL에서 널리 쓰인 출발점으로 볼 수 있다.
- 공식 D4RL locomotion training script는 dataset return 범위를 이용해 reward를 rescale한다. 본 프로젝트의 raw reward 사용은 dataset 자체를 그대로 평가하기 위한 별도 결정이므로, critic scale이 불안정하면 이 차이를 원인 후보로 보고해야 한다.
- 따라서 Round 1의 IQL 기본형은 literature-grounded `2 x 256` MLP에서 시작한다. 다만 본 dataset의 최대 668차원 observation에 충분하다고 가정하지 않고, `4 x 256`과 `4 x 512`를 포함한 짧은 capacity gate로 최종 backbone을 고른다. 이 비교는 baseline이 명백한 capacity bottleneck으로 약해지는 것을 방지하기 위한 screening 절차다.

따라서 Round 1에서는 Sequential IQL과 actor-only EWC가 동일한 IQL trainer를 공유하도록 구현한다.

## EWC

### 일차 자료

- [Overcoming catastrophic forgetting in neural networks](https://doi.org/10.1073/pnas.1611835114)

### 구현에 반영할 내용

- EWC는 이전 parameter 주변의 quadratic penalty에 parameter importance를 가중한다.
- 원 논문의 RL 실험은 Atari 계열 agent에 diagonal Fisher와 별도의 EWC scaling factor를 사용한다.
- EWC는 과거 transition 대신 이전 parameter와 importance를 저장하므로 replay보다 memory 요구가 작다.
- 원 논문은 IQL actor에 대한 Fisher 정의를 제시하지 않는다. 따라서 stochastic actor의 log-likelihood로 Fisher를 추정하는 것은 Round 1의 adaptation이며, 원 논문의 그대로인 구현으로 표현하면 안 된다.

Round 1에서는 IQL actor 전체에 diagonal Fisher penalty를 적용하고 Q/V network는 제약하지 않는다.

## L2M

### 일차 자료

- [NeurIPS 2023 paper](https://proceedings.neurips.cc/paper_files/paper/2023/hash/77e59fafe99e94f822e79bf9308ec377-Abstract-Conference.html)
- [Official implementation](https://github.com/ml-jku/L2M)

### 구현에 반영할 내용

- 공식 구현은 Meta-World/Continual-World, DMControl, Atari, Gym-MuJoCo와 ProcGen의 offline dataset 또는 online interaction에서 Decision Transformer를 학습한다.
- 원 실험은 약 40M-parameter Multi-Domain Decision Transformer를 여러 task에서 사전학습한다.
- L2M은 사전학습 backbone을 동결하고, learnable key와 LoRA modulator로 이루어진 modulation pool만 학습한다.
- 논문 설정은 context length `5`, action bin `64`, LoRA rank `8`, pool size `30`, key regularization `0.5`를 사용한다.
- L2M learning rate는 DMControl에서 `1e-4`, Continual World에서 `5e-5`다.
- 원 backbone은 1M updates로 사전학습되고 continual fine-tuning은 task당 100k updates를 사용한다.

따라서 A1만 사전학습하고 compact backbone과 작은 budget을 사용하는 Round 1 설정은 `L2M (A1-pretrained adaptation)`으로 표기한다. 이는 핵심 mechanism은 유지하지만 paper-faithful scale reproduction은 아니다.

## VQ-CD

### 일차 자료

- [Paper](https://arxiv.org/abs/2410.15698)
- [Paper HTML and appendix](https://arxiv.org/html/2410.15698v1)

### 구현에 반영할 내용

- VQ-CD는 task별 다른 state/action space를 정렬하는 Quantized Spaces Alignment (`QSA`)와 task-related sparse mask를 사용하는 Selective Weights Activation (`SWA`) diffuser로 구성된다.
- Diffusion policy와 함께 inverse dynamics model을 사용한다.
- Appendix 기본값은 QSA/SWA hidden size `256`, state/action codebook `512`, state latent `10 x 2`, action latent `5 x 2`다.
- QSA learning rate는 `1e-3`에서 `1e-4`로 감소하며, sequence length는 `8`, diffusion learning rate는 `3e-4`, batch size는 `32`다.
- Mask rate는 $1/I$, diffusion step은 `200`, DDIM stride는 `20`이다.
- QSA pretraining, sequential SWA training과 weights assembling을 분리해서 실행한다.

현재 확인 범위에서는 공식 code repository를 찾지 못했다. 따라서 Round 1 구현은 official reproduction이 아니라 paper-faithful reimplementation으로 표기하고, 논문에 명시되지 않은 adapter와 preprocessing 결정을 모두 기록해야 한다.

## Round 1에 적용되는 공통 제약

- Sequential IQL과 actor-only EWC는 같은 offline RL trainer를 공유하고 EWC penalty와 auxiliary state만 다르게 한다.
- L2M과 VQ-CD는 architecture-level baseline이므로 IQL에 작은 loss를 추가한 형태로 축소하지 않는다.
- Transition method와 sequence method의 비교에는 return뿐 아니라 data exposure, parameter 수, auxiliary memory와 wall-clock time을 함께 보고한다.
- Round 1 main comparison은 과거 transition replay를 허용하지 않는다.
- 축소된 screening budget에서 실패한 L2M/VQ-CD 결과만으로 원 방법의 성능을 결론내리지 않고, 추가 budget 또는 tuning 필요 여부를 먼저 보고한다.
