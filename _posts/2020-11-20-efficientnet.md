---
title: "[Review] (2019) EfficientNet"
categories:
  - Deep Learning
tags:
  - Deep Learning
  - Model Search
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---


# 1. Introduction

- Convolutional Network는 성능이 좋음
    - 성능을 높이기 위해 scale 조절을 하는 방식에는 다음과 같은 방식들이 있음
        - Layer 수 조절 (ResNet)
        - Channel 수 조절(WideResNet, MobileNet)
        - Input Image Size 조절
    - 성능을 높이기 위해서 위의 3가지를 키우면 됨. 하지만 그만큼 efficiency가 떨어짐
- 제안하는 방법은 성능을 높이는게 입증된 3가지의 parameter들을 어떻게 효율적으로 잘 조절하여 modeling하는지를 말해줌

# 2. Methods

- 일반적으로는 Input X에 대해 Output Y가 잘나오게 하는 Layer 구조(Model 구조)를 최적화하는 것에 초점을 맞춘다면, 모델 확장(Model Scaling)은 baseline model 구조는 그대로 차용하되, Depth, Width, Resolution 등 scale 조절이 가능한 부분을 활용함.

### 2.1. Compound Model Scaling

![image](/assets/imgs/paper/2019-efficientnet/00.png)

**Depth**

- Layer가 깊어질수록, 복잡한 feature를 더 많이 추출 가능함
- 대표적인 예로, skip connection, batch normalization이 있음

**Width**

- 각 Layer당 channel 수를 늘려 fine-grained feature를 더 많이 확보하는 것을 목표로 함
- 하지만, 너무 shallow한 모델에 extreamly wider하면 학습이 더 어려워짐

**Resolution**

- Input Image 크기가 커질수록 더 fine-grained feature를 추출할 수 있음
- 너무 큰 resolution에서는 정확도 증가의 효율성이 많이 떨어짐

**Compound Scaling**

![image](/assets/imgs/paper/2019-efficientnet/01.png)

- Depth와 Resolution 조합 변화를 지켜보면 둘 다 커질수록 성능이 높아지는 것을 알 수 있음
- 그래서 다음과 같이 compound scaling method를 정의함
    - $\phi$ 는 유저가 선택하는 hyperparameter, 총 FLOPS를 표현하는 3가지 조건을 아래와 같이 제약하고 small grid search range에서 적절한 값을 선택함
    - depth : $d=\alpha^{\phi}$
    - width : $w=\beta^{\phi}$
    - resolution : $r=\gamma^{\phi}$
        - $\alpha \cdot \beta^2 \cdot \gamma^2 
        \approx 2$
        - $\alpha \ge 1, \beta \ge 1,\gamma \ge 1$
        

### 2.2. EfficientNet Architecture

![image](/assets/imgs/paper/2019-efficientnet/02.png)

- Model Scaling은 baseline network를 변경하는 것이 아님
- 효율적인 network 구성을 위해 다음과 같은 공식을 통해 network를 평가함
    - $ACC(m) \times [FLOPS(m)/T]^w$
        - $m$ : Model
        - $T$ : Target FLOPS
        - $w$ : trade-off hyperparameter
    - latency가 아닌 FLOPS로 평가하는 것은 어느 특정 device에 맞춰서 하는 것이 아니기 때문
- Parameter $\alpha, \beta, \gamma, \phi$ 는 다음과 같이 정함
    - STEP 1 : $\phi=1$로 고정하고  $\alpha \cdot \beta^2 \cdot \gamma^2 
    \approx 2$ 를 만족하는 값을 찾는다
    - STEP 2 : 위의 조건을 만족하는 파라미터들에서 $\phi$를 1~7로 변화하여 모델을 생성함

# 3. Experimental Results & Conclusion

![image](/assets/imgs/paper/2019-efficientnet/03.png)

- Single scaling model들과의 비교. 유사 정확도의 모델들과 비교했을 때, model의 크기, FLOPS 등 모두 뛰어남.
- 정확도도 모델 크기 대비 상당히 효율적이기 때문에 우수함을 입증

![image](/assets/imgs/paper/2019-efficientnet/04.png)

- MobileNet과 ResNet에 Compound scaling을 적용했을 때의 변화를 비교함
- Width, Depth, Resolution만 변화를 주어도 상당량 올라가지만 종합으로 하였을 때 최대 5%정도 정확도의 향상이 있었음

![image](/assets/imgs/paper/2019-efficientnet/05.png)

- Latency 비교. 동등한 성능의 single scaling model들과 비교하여 5~6배 가량 속도 향상이 있음

![image](/assets/imgs/paper/2019-efficientnet/06.png)

- ImageNet으로 pretrain된 모델들과 비교하여 target dataset에 transfer learning하였을 떄 비교 성능
- Gpipe, DAT와 같이 매우 큰 모델들과 비교하여도 거의 성능차이가 없음

![image](/assets/imgs/paper/2019-efficientnet/07.png)

- Compound scaling이 CAM에 미치는 효용을 보여주는 결과
- baseline 모델에 비해 상당히 정확한 activation을 보여줌

- 기존에 나와있는 이론 + 실험을 적절하게 잘 이용한 것으로 보임 (Resolution, Depth, Width 변화에 따른 성능 차이)
- Mobile Inverted Bottleneck Conv라는 layer구성을 통해 baseline을 최대한 효율적으로 만듦 (추후 scaling에 더 도움이 되는 것으로 보임)
- baseline도 중요하지만 어떻게 scaling하느냐에 따라 성능차이가 큼을 알 수 있었음
- 논문에서 제안된 이론들을 이용하여 AutoML을 사용하면 더 효율적인 모델이 나오지 않을까 하는 생각?