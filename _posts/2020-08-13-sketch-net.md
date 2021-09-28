---
title: "[Review] (2017) A Neural Representation of Sketch Drawings"
categories:
  - Reconstruction
tags:
  - DeepLearning
  - Sketch
  - RNN
  - Reconstruction
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 1. Introduction

- GAN, VAE, RNN+MDN 등 다양한 reconstruction 방법들이 있었음
- 특히, sketch reconstruction에서는 dataset의 크기가 부족하였음
    - 그래서, 저자들이 70K개의 QuickDraw 데이터셋 제안함
    

# 2. Methods

- Dataset의 구성은 다음과 같음
    - 345 class에 대한 20초내 분량의 드로잉
    - 각 데이터는 5개의 element로 구성됨
        - $\Delta x$: 이전 포인트로부터 x방향으로 움직인 거리
        - $\Delta y$: 이전 포인트로부터 y방향으로 움직인 거리
        - $p_1$: 현재 드로잉 중인 상태
        - $p_2$: 그림을 그리지않는 일시정지 상태. 펜이 종이에 붙어있지 않음(드로잉 종료가 아님)
        - $p_3$: 드로잉을 끝마침

![image](/assets/imgs/paper/2017-sketch-net/00.png)

- Encoder : LSTM으로 구성 (Sequence-to-Sequence VAE)
    - VAE의 Encoder처럼 평균, 표준편차에 Gaussian noise를 추가한 variational latent vector를 생성함 (표준편차는 exponential 연산을 하여 음수가 없도록 함)
    - 정/역방향 그리기에 대한 LSTM으로 구성됨
- Decoder : HyperLSTM으로 구성 (autoregressive RNN)
    - latent vector와 이전 layer의 출력값을 입력으로 받아 GMM을 이용하여 다음 상태를 예측함
    - Initial Hidden State : $[h_0,c_0]=\tanh(W_z z+b_z)$
    - Decoder Input : $x_i=[S_{i-1},z]$
    - 움직인 거리는 GMM의 M normal distribution으로, 상태는 categorical distribution으로 모델링함.
    - Encoder와 마찬가지로 표준편차는 exponential을 하여 양수로, correlation에 대해서는 tanh를 하여 -1에서 1사이의 값으로 만듦

### 2.1. Key challenging

- 언제 그리는 것을 멈춰야하는지 알기 어려움 (드로잉 종료 state는 그림 당 1개만 존재하기 때문)
- $N_{\max}$는 모든 sequence 중 가장 길이가 긴 것을 의미하며, 모든 sequence에 대해 이 길이로 맞춰서 데이터 생성 (만약 드로잉 종료 후에도 $N_{\max}-N_{current}$만큼의 길이가 남으면 그 기간동안 드로잉 종료 state로 봄

### 2.2. Temperature

- temperature $\tau$는 생성되는 latent vector 및 Decoder의 parameter 생성의 randomness를 결정함
- 범위는 0에서 1사이로, 0에 가까울 수록 randomness가 줄어듦

### 2.3. Unconditional Generation

- Encoder RNN 없이 Decoder만 학습한 경우
- 초기 Hidden sate를 0으로 모두 초기화하여 학습함

### 2.4. Loss Function

- Reconstruction Loss
    - $L_s$ : 포인트 간 이동거리에 대한 Reconstruction loss
    - $L_p$ : state를 정확히 예측했는지에 대한 loss
    - $L_R=L_s+L_p$
- Kullback-Leibler (KL) divergence Loss
- $Loss=L_R+w_{KL}L_{KL}$
    - $w_{KL}$ : KL Loss에 대한 가중치
- Recon Loss와 KL Loss는 tradeoff 관계임

![image](/assets/imgs/paper/2017-sketch-net/01.png)

- KL에 대한 가중치가 작을수록 분포 일치도가 낮아지기 때문에 latent vector에 drawing point에 대한 prior정보가 적어짐

# 3. Results & Conclusion

### 3.1. Conditional Reconstruction

![image](/assets/imgs/paper/2017-sketch-net/02.png)

- $\tau$ 값에 따라 비정상적인 Input에도 Reconstruction하고자 하는 카테고리를 잘 살려내는지 확인함
- 0.01(Blue)에서 1(Red)로 갈수록 Randomness가 커져 비정상 입력에 대해서 명확한 형태를 재구성하지는 못함
- 그래도 전반적으로 정상적이지 않은 입력에 대해서도 최대한 target에 대해 재구성하려고함(칫솔 형태의 고양이, 트럭형태의 돼지 등)

### 3.2. Latent Space Interpolation

![image](/assets/imgs/paper/2017-sketch-net/03.png)

- $w_{KL}$ 변화에 따른 Latent vector interpolation 실험
- 가중치가 클수록(1에 가까울수록) 변화가 자연스럽고, category에 맞는 결과도 완성도가 있음

### 3.3. Predicting Different Endings of Incomplete Sketches

![image](/assets/imgs/paper/2017-sketch-net/04.png)

- 미완성 sketch를 입력으로 주어 Decoder Only Model에서 나머지 부분을 이어 그리는 실험
- trainset의 데이터를 $\tau=0.8$로 하여 randomness를 어느정도 준 상태로 학습하여 다양한 형태의 예측 스케치를 보여줌

- RNN을 통해 sketch 궤적과 상태를 예측한다는 점이 흥미롭다
- GAN에서 자주 보이는 실험처럼 latent space간 interpolation, 산술 연산을 통해 결과를 변화시킬 수 있는 점도 흥미로움
- 모델 및 dataset이 더 완성도가 있으면 다양한 application에서 사용이 가능할 것 같음

# References

- [https://magenta.tensorflow.org/sketch-rnn-demo](https://magenta.tensorflow.org/sketch-rnn-demo)
- [https://arxiv.org/abs/1704.03477](https://arxiv.org/abs/1704.03477)