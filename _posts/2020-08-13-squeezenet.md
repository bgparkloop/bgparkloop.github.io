---
title: "[Review] (2017) SQUEEZENET"
categories:
  - Model Compression
tags:
  - Deep Learning
  - Model Compression
  - CNN
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 1. Introduction

- 모델의 크기가 작으면 유리한 점
    - 병렬 분산 학습 등에서 효율적일 수 있도록 모델의 크기가 작으면 좋음
    - client에게 새로운 모델을 다시 제공할 때 overhead를 줄일 수 있음
    - FPGA와 같은 임베디드 시스템에 deploy하기 용이함
- Model Compression을 위한 노력들이 있었음
    - SVD와 같은 전통적인 방식
    - CNN micro-architecture의 변형 : 3x3를 1x1과 같은 filter로 tricky하게 많이 사용해서 성능은 같지만 파라미터 수는 줄어들도록 설계
    - CNN macro-architecture의 변형 : Layer나 Module 단위에서의 병합. Residual network나 Highway network와 같은 전체 구조의 변형
- 그래서 SqueezeNet이라는 새로운 모델을 제안함
    - 잘 만들어진 모델들을 압축시키는 방법

# 2. Methods

- 정확도를 유지하며 적은 파라미터로 모델을 구성하기 위해 3가지 전략을 사용
    - 1) 3x3 filter를 1x1 filter로 변환
    - 2) input layer의 3x3 filter 수를 줄이기 (해당 레이어에서 3x3 filter 수가 많을수록 막대하게 파라미터 수가 늘어나버림)
    - 3) Conv layer가 큰 activation map을 갖도록 늦게 downsampling
        - downsample을 늦게 할 수록 뒷단의 layer들이 더 넓은 영역을 참조하기 때문에 더 높은 classification 성능을 가짐

### 2.1. Fire Module

![image](/assets/imgs/paper/2017-squeezenet/00.png)

- Squeeze : 1x1 filter로 차원 수를 줄이는 작업 진행
- Expand : 1x1, 3x3 filter로 activation map 추출 (width, height는 유지되도록 padding)

![image](/assets/imgs/paper/2017-squeezenet/01.png)

- 3가지의 SqueezeNet 종류들
- 오른쪽으로 갈수록 좀 더 풍부한 feature 추출이 가능하도록 residual과 유사하게 feature를 더해줌

# 3. Results & Conclusion

![image](/assets/imgs/paper/2017-squeezenet/02.png)

- Fire module에서 squeeze, expand 각 부분에서 sparse하게 파라미터를 줄임
- 전체적으로 3분의 1가량으로 줄어든 것을 확인가능

![image](/assets/imgs/paper/2017-squeezenet/03.png)

- 기존의 Compression 방법들과 비교
- Deep Compression정도가 감소율이 비슷. SqueezeNet + Deep Compression 시 최대 510배의 큰 감소율을 보이며 정확도도 유지하거나 약간 높은 수준
- 비교를 통해 small model들은 모든 파라미터를 다 사용하지도 않고(압축 가능), float 값의 모든 자릿수를 이용할 필요도 없음(자릿수 표현을 하는 bit수 줄임)

### 3.1. Exploring Microarchitecture Design

![image](/assets/imgs/paper/2017-squeezenet/04.png)

- Fire Module의 filter 수를 정하는 meta parameter를 두어 실험을 수행함
    - $s_{1x1},e_{1x1},e_{3x3}$은 각각 squeeze, expand의 1x1, 1x1, 3x3의 필터 수
    - $base_e$ :  첫번째 fire module의 expand 필터 수
    - $freq$ : 해당 주기마다 $incr_e$만큼 expand 필터 수를 증가시킴
    - $e_i=base_e+(incr_e \times [\frac{i}{freq}])$ : i번째 fire module의 expand 필터 수
        - $e_i=e_{i,1x1}+e_{i,3x3}$
        - $e_{i,3x3}=e_i \times pct_{3x3}$ : 3x3 필터 수 감소 파라미터
        - $s_{i}=SR \times e_i$ : Squeeze Ratio로 감소시킨 필터 수
- SR의 경우 0.5정도까지는 크게 정확도가 감소하지 않음(필터 수 거의 절반으로 줄어듦)
- SR을 고정하고, pct를 감소하며 실행했을 때 일정 이상이 지나면 큰 영향이 있음
    - 50%까지는 정확도의 변화가 없는 것으로 보아 너무 많은 spatial filter도 의미가 없음 (어느정도 양은 있어야함)
    

### 3.2. Exploring Macroarchitecture Design

- Resnet 구조처럼 중간 중간 결과를 다음 layer에게 넘겨주어 feature를 풍부하게 함
- 이런 형태의 단점은 input과 output의 크기가 같아야 하므로 SqueezeNet의 Fire Module에서는 모든 layer에 적용이 힘듦

![image](/assets/imgs/paper/2017-squeezenet/05.png)

- 3가지 타입의 SqueezeNet의 결과를 비교해보니 의외로 Simple bypass가 가장 성능이 좋음
- Bypass를 통해 representational bottleneck을 어느정도 완화할 수 있음

- 유사 연구로는 Network Pruning과 DSD(Dense-Sparse-Dense)와 같은 연구가 있는데, 이 연구들은 기존 parameter 중 쓸모없는 부분은 0으로 없애거나 학습 과정에서만 sparse하게 변경하는 것인 반면, SqueezeNet은 설계 구조 자체부터 Compression에 적합한 모델링이기 때문에 더 효율적으로 보임

# References

- [https://arxiv.org/abs/1602.07360](https://arxiv.org/abs/1602.07360)