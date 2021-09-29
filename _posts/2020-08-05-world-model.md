---
title: "[Review] (2018) World Models"
categories:
  - Reinforcement Learning
tags:
  - Deep Learning
  - Reinforcement Learning
  - RNN
  - VAE
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 1. Introduction

- 사람은 상상을 통해서 추상적으로 어떤 주제에 대해 시간적, 공간적 정보들의 관계들을 학습할 수 있다.
- Reinforcement Learning(RL)은 현재와 과거의 이득을 통해 artificial agent 학습
    - Large RNN은 많은 데이터와 자원을 필요. but, model-free RL은 작은 network면 됨
    - credit assignment problem : 최종결과에 대해 취해진 action들에 대해 어느 action에 어느정도의 reward를 할당해야 하는지에 대한 문제
    - Large world model과 small control model로 나누어 학습
- 제안하는 방법은 Generated environment에서 RL모델과 controller 모델 학습 후, 실제 환경으로 transfer
    - 생성된 환경의 불완전함 속에서 학습 관련 문제를 해결하기 위해 temperature라는 변수를 둠

# 2. Methods

### ‌2.1. Agent Model

![(2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled.png]((2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled.png)

- 3가지의 component로 구성됨
    - Visual sensory component : 현재 또는 과거의 상황을 관측함
    - Memory component : 과거에 쌓여있는 정보들을 바탕으로 미래에 대한 부분을 예측함
    - Decision-making component : Vision과 Memory component의 결과를 바탕으로 어떤 action을 취할지 결정함

### 2.2. VAE (V) Model

![(2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%201.png]((2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%201.png)

- Visual 환경 구성을 위해 VAE 모델을 V component로 채택함
- 원본 영상들을 VAE로 학습하여 latent vector z를 추출하고 특정 z가 들어오면 시뮬레이션 가능한 프레임을 생성할 수 있게함
- V 모델은 agent가 frame의 상황을 알 수 있도록 압축된 latent vector 생성에 목적을 둠

### 2.3. MDN-RNN (M) Model

![(2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%202.png]((2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%202.png)

- V에서 환경을 관측한 z vector를 이용해 M에서는 미래를 예측함
    - 실제 환경은 stochastic하기 때문에 M의 RNN output은 deterministic prediction이 아닌 probability density function으로 나옴 (MDN - Mixture Density Network을 통해)
    - 현재 상황을 기준으로 다음상황을 예측 : $P(z_{t+1}|a_t,z_t,h_t)$
    - Temperature $\tau$ 는 모델의 불확실성을 제어하는데 사용 (latent vector의 범위 조절)

### 2.4. Controller (C) Model

- C 모델은 축적된 reward가 최대화되는 방향으로의 action을 결정함
- Agent의 복잡성을 최대한 world model(V + M)에 두기 위해 C 모델은 매우 간단한 형태로 구성
    - $a_t=W_c[z_t \ h_t] + b_c$
    - 모델의 복잡성을 낮춰서 CA problem을 다룰 수 있는 다양한 방법을 수행할 수 있게 도움이 됨. (예 - evolution strategies)
    - C 모델의 파라미터 최적화를 위해 Covariance-Matrix Adaptation Evolution Strategy(CMA-ES)를 사용
    - Multi-CPU의 single machine에서 병렬 처리를 통해 rollout 수행
    

# 3. Results & Conclusion

### ‌3.1. Procedure

- random policy에 대한 10,000개의 rollout을 모음
- V모델 학습
- M모델 학습
- C모델 학습

### 3.2. V Model Only

- M 모델을 제외하고 V모델만을 이용해 상황 판별 및 액션
    - 스코어 편차가 매우 크며, racing simulation에서 가파른 코너 등에 매우 약한 모습을 보임
- C 모델을 추가하면 편차가 줄어들고 스코어가 조금 올라가나 뭔가 부족함

### 3.3. Full World Model

![(2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%203.png]((2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%203.png)

- V 모델만 사용했을 때보다 월등히 좋아짐
- 이를 통해 RNN을 사용한 미래 예측이 큰 영향을 끼침을 알 수 있음

### 3.4. Train on Dreams

- 제안된 World model에서 M 모델의 기능을 활용하여 $z_{t+1}$의 시나리오를 생성할 수 있음
- 생성된 시나리오를 이용하여 가상환경(시나리오)를 만들어 그 안에서 RL의 학습이 가능함

**Procedure**

- 10,000개의 random policy rollout을 수집
- VAE 모델을 학습해서 latent space representation을 확보함
- MDN-RNN 모델을 학습
- C 모델을 V모델을 통해 생성한 가상 환경에서 학습
- 학습된 정책 set을 실제 환경에서 테스트

### 3.5. Weakness

- 실제로 M 모델이 잘못 학습되어 Fool해질 때가 있음 (몬스터가 총알을 안쏜다던지)
- 이러한 약점은 Dynamic model을 RL에 사용한 이전 연구들에서도 등장함
    - Agent가 모델에 관여하기 때문
- 약점 보완을 위해 Bayesian model (PILCO)들을 사용하면 어느정도 불확실성을 해결하지만 완벽하지는 않음
- 최근에는 모델 기반 방법에 Model-free 방식을 결합하여 시도하는데, 이 방법은 실제 환경에서 사용할 때는 policy의 fine-tuning을 model-free방식에 의존해야함

### 3.6. Cheating the World Model

- $\tau=0.1$ 일 경우, deterministic LSTM과 같아서 agent가 뭘 하든간에 mode collapse에 의해 정상적인 환경 구성이 되지 않음.
- Temperature 변화에 따라 큰 차이를 보임. temperature가 커질수록 가상환경에서의 성능은 떨어지는데 agent가 다양한 전략을 발견하여 실제 환경에서의 성능이 좋아지고 편차가 커짐.
- 하지만, 너무 늘리면 오히려 학습하기 어려운 가상환경들이 세팅되어 성능이 떨어지므로 적절한 조절이 필요함

![(2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%204.png]((2018)%20World%20Models%2010ee0b44788748a8b4e27fe3da045aeb/Untitled%204.png)

### 3.7. Iterative Training Procedure

- 합리적인 World model 학습을 위해서 전략적으로 어떻게 세계를 관측할지 학습해야함
- Learning To Think라는 전략을 통해 이러한 학습을 수행함
    - 1) M, C를 random model parameter로 초기화
    - 2) 실제 환경에서 N번 rollout 진행. 이 때, 행해진 모든 액션과 관측된 정보를 기록함
    - 3) M을 $P(x_{t+1},r_{t+1},a_{t+1},d_{t+1}|x_t,a_t,h_t)$로 모델링 하고 학습. C는 M의 기대보상을 통해 최적화.
    - 4) 완벽하게 task가 끝나지 않으면 2번으로 돌아가 다시 진행
- 어려운 task일수록 step2부분에서의 다양한 환경을 탐험하는 것이 중요하다.
- 만약 M 모델이 poor job을 하면, 아직 익숙치않은 환경을 맞닥뜨린것임

# References

- [https://arxiv.org/abs/1803.10122](https://arxiv.org/abs/1803.10122)
- [https://worldmodels.github.io/](https://worldmodels.github.io/)