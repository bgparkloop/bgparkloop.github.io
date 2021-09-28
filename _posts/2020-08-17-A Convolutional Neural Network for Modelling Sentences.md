---
title: "[Review] (2014) A Convolutional Neural Network for Modelling Sentences"
categories:
  - NLP
tags:
  - CNN
  - DeepLearning
  - NLP
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# Introduction

- 입력 받는 문장의 길이는 다양함
- Dynamic K-Max Pooling을 이용함
    - Max pooling의 일반화 버전
    - non-linear subsampling을 수행하기 때문에 일반화에 좋다
- 4가지 실험으로 구성함
    - 영화 리뷰 감정 분석 (binary / multi-class)
    - 6개 질문 종류에 대해 categorization
    - 트위터 게시물의 감정 예측

# Methods

![image](/assets/imgs/2014-cnn-nlp/00.png)

- Wide convolution + Dynamic K-max pooling으로 network 구성
    - Wide convolution : filter 범위가 feature map보다 크면 나머지는 0으로 채워서(zero padding) 연산하는 기법
    - K-max pooling : local range K에서 가장 큰 값만을 취하는 방식(일반적인 max pooling)
    - Dynamic K-max Pooling : K-max pooling과 같지만 모델의 깊이나 sentence의 길이에 따라 가변적인 K 값을 갖는 pooling
        - $k_l=\max(k_{top},\lceil \frac{L-l}{L}s \rceil)$
        - $l$ : 현재 conv layer의 번호
        - $L$ : 전체 conv layer의 수
        - $s$ : 입력된 문자열 길이
        - $k_{top}$ : topmost conv layer의 고정 pooling 크기
        - ex) 전체 conv layer가 3개고, $s$가 18, $k_{top}$이 3인 첫 번째 conv layer의 $k_l$은 12이다.
- Folding : Feature map의 각 행과 다음 행을 component-wise로 더해서 $d$개의 행에서 $d/2$개의 행으로 만드는 작업
- N-gram과 같은 특성을 위해 첫 번째 Covolutional layer에서 N=10의 큰 단위의 N을 사용함
- Convolutional layer와 Pooling layer를 통해 Input sentence에 대해 내부적으로 word 간 연관성을 갖도록 feature graph가 생성됨(induced feature graph)

# Experimental Results & Conclusion

![image](/assets/imgs/2014-cnn-nlp/01.png)

![image](/assets/imgs/2014-cnn-nlp/02.png)

- 영화 리뷰, 트위터 리뷰의 감정 분류 문제에서 기존의 Naive bayesian, SVM, BoW 모델들보다 성능이 뛰어남

![image](/assets/imgs/2014-cnn-nlp/03.png)

- 6개의 질문 형식으로 분류하는 문제에 있어서도 외부 조건(추가 특징 정보)를 추가하여 비교할 때 SVM다음으로 가장 좋은 성능을 보임

![image](/assets/imgs/2014-cnn-nlp/04.png)

- Feature detector를 시각화하여 보니 단순히 N-gram의 인식뿐 아니라 N-gram 내의 패턴도 분석되어 의미적으로 유사한 것끼리 묶이는 것을 확인할 수 있음

- 생각외로 CNN구조가 NLP task에 굉장히 유용함을 볼 수 있었음
- 특히, feature detector의 visualization으로 유사 패턴끼리 모이는 것은 이미지에서 CAM을 통해 정답을 맞추기 위해 활성화 되는 영역을 보는 것과 같은 느낌(또는 filter의 모양)

# References

- [http://mirror.aclweb.org/acl2014/P14-1/pdf/P14-1062.pdf](http://mirror.aclweb.org/acl2014/P14-1/pdf/P14-1062.pdf)