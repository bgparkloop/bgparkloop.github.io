---
title: "[Review] (2020) EfficientDet"
categories:
  - Detection
tags:
  - Deep Learning
  - Detection
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---


# 1. Introduction

- 기존에 나온 Detection Model들(AmoebaNet, NAS-FPN, 등등)은 너무 overhead가 커서 robotics나 self-driving에 사용이 어려움
- mobile과 같은 low-cost 환경에서도 사용 가능한 모델들(one-stage, anchor free, compress를 적용한 모델들)도 있지만 효율성 증대를 위해 정확도를 희생함(아주 크진 않지만 의외로 차이남)
- 기존 모델들이 정확도/효율성 증대를 위해 사용한 방법들
    - 모델의 크기를 키움(parameter가 많아짐)
    - input을 키움(memory 많이 필요)
    - layer당 channel을 많이 늘림
    - one-stage method
        - Free RoI proposal step
        - Predefined anchors
    - feature fusion
        - one-way path (FPN) : Top-down 방식의 feature fusion (5x5의 resolution → 80x80의 resolution 순으로 큰 단위에서 작은 단위로)
        - bidirectional path (PANet) : Top-down에 Bottom-up방식을 합침(이어짐). parameter수가 많아진다
        - irregular bidirectional path (NAS-FPN) : PANet과 FPN의 중간쯤인데 최적화된 bidirectional 구조를 찾기위해 반복적인 search를 필요로 함. irregular한 모양을 가지기 때문에 변경이나 해석이 어려움
- Object detection에서 중요한 것은 효율적인 multi-scale의 표현
- 이를 위해 논문에서 제안하는 방법
    - BiFPN의 사용
    - Weighted Feature Fusion

# 2. Methods

### 2.1. BiFPN

![image](/assets/imgs/paper/2020-efficientdet/00.png)

- 기존의 FPN의 방식은 one-way path이고 PANet은 bi-path이지만 이 둘의 장점을 합침.
- Top-down의 node를 Bottom-up으로 이으면서 original node를 output node에 합쳐 풍부한 feature를 갖게 함
    - 아래는 구체적인 예시 (td는 중간 결과, in은 입력, out은 출력 값)
    - $P^{td}_6 = Conv(\frac{w_1 \cdot P^{in}_6 + w_2 \cdot Resize(P^{in}_7) }{w_1+w_2+\epsilon})$
    - $P^{out}_6 = Conv(\frac{w'_1 \cdot P^{in}_6 + w'_2 \cdot P^{td}_6 + w'_3 \cdot Resize(P^{out}_5) }{w'_1+w'_2+w'_3+\epsilon})$

### 2.2. Weighted Feature Fusion

- 서로 다른 resolution의 feature들은 fusion시 기여도가 다름
- 이를 해결하기 위해서는 weighting을 통해 resolution feature마다 다른 기여도를 주어야 함
- 3가지를 제안하고 최종 선택
    - **Unbounded fusion** : $O=\sum_i w_i \cdot I_i$
        - learnable weight를 input에 곱하여 weighted sum함. weight의 값이 range가 없어 학습 시 불안정할 수 있음
    - **Softmax-based fusion** : $O=\sum_i \frac{e^w_i}{\sum_j e^w_j} \cdot I_i$
        - softmax function 특성 상 0에서 1사이로 normalization되어 효율적이지만, GPU 사용에 있어 softmax가 비효율적임.
    - **Fast normalization fusion** : $O=\sum_i \frac{w_i}{\epsilon + \sum_j w_j} \cdot I_i$
        - softmax의 효율성을 갖고 GPU 가속도 받을 수 있도록 만듦.
        - softmax-based fusion 대비 GPU 효율 30% 이상
        

### 2.3. EfficientDet

![image](/assets/imgs/paper/2020-efficientdet/01.png)

- 기존 모델들은 single 혹은 제한된 scale들에 대해서만 학습이 가능하였음
- 하지만, efficientnet을 backbone으로 사용하여 다양한 scale에 대해 학습이 가능하도록 함
- BiFPN, Class, Box layer의 수, weight channel 수들을 backbone의 크기에 맞춰 유동적으로 조절함

![image](/assets/imgs/paper/2020-efficientdet/02.png)

# 3. Experimental Results & Conclusion

![image](/assets/imgs/paper/2020-efficientdet/03.png)

- COCO 2017 dataset에 대해 학습하고 평가함
- 기존 모델들 대비 4~9배 더 작고, 13~42배 FLOPs가 더 작음. (효율성, latency가 좋음)
- 파라미터 수, 모델 크기가 작아 빠른 속도로 연산이 가능하여 많은 곳에 사용이 가능

![image](/assets/imgs/paper/2020-efficientdet/04.png)

### 3.1. EfficientDet for Semantic Segmentation

![image](/assets/imgs/paper/2020-efficientdet/05.png)

- EfficientDet-D4을 그대로 사용하되, P2 level만을 per-pixel classification에 사용
- DeepLabV3+와 Pascal VOC 2012 dataset에대해 비교 진행
    - 성능도 더 좋고, FLOPs도 9.8배 적어 매우 효율적임

### 3.2. Ablation Study

- BiFPN 성능 비교
- ResNet50에 FPN 쓴거랑 비교 시 FLOPs, 파라미터 수도 줄어들고 성능은 오히려 오름.
- BiFPN에서는 depthwise separable convolution를 이용하여 parameter수도 획기적으로 줄임

![image](/assets/imgs/paper/2020-efficientdet/06.png)

- Softmax VS Fast Fusion
    - Softmax와 Fast Fusion은 거의 유사하게 동작하는데 실제 아래 도표처럼 학습 후 성능을 비교하면 Fast Fusion이 약간 정확도가 낮음
    - 하지만, GPU 속도 측면에서 1.26~1.31배로 차이가 나기 때문에 정확도가 낮아지는 것 대비 효율이 좋음(trade-off)

![image](/assets/imgs/paper/2020-efficientdet/07.png)

- Compound Scaling
    - BiFPN, resolution, channel, depth에 대해 각각 scale up 하는 것보다 한 번에 하는 것이 효율이 훨씬 좋았음
    - 개별적으로는 BiFPN과 channel이 가장 효율이 좋음
    
    ![image](/assets/imgs/paper/2020-efficientdet/08.png)
    
- Efficientnet 자체도 기존에 연구되었던 depth, channel, image size에 대해서 종합적으로 고려한 모델로 효율을 극대화시켰는데, EfficientDet에서도 이러한 특성과 더불어 BiFPN이라는 효율적인 feature fusion 방식을 도입함으로써 Detection 뿐 아니라 Multi-scale이 고려되는 segmentation에서도 효과적이라는 것을 입증하여 상당히 인상적임
- BiFPN이라는 layer 구조는 EfficientDet이 아니더라도 응용하여 다른 network에도 충분히 사용 가능할 것으로 보이고 매우 효율적임.