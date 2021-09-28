---
title: "[Review] (2015) A Sensitivity Analysis of (and Practitioners’ Guide to) Convolutional Neural Networks for Sentence Classification"
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

# 1. Introduction

- 기존 연구된 CNN을 통한 sentence classification들은 생각보다 좋은 성능을 냄
- 정확도를 최적화하기 위해 다양한 architecture 구조, embedding 방식 등을 고안함
- 제안하는 연구에서는 같은 형식의 CNN이라도 인풋이나 hyperparameter를 바꾸면 생기는 성능 차이를 고려해서 실제 학습 시 좋은 hyperparameter 값을 선정하기 위해 테스트한 결과를 보여줌
    - input word(embedding)
    - filter의 region size
    - feature map 수
    - activation function
    - pooling strategy
    - regularization

# 2. Methods

![image](assets/imgs/2015-cnn-nlp/00.png)

- 실험을 위해 사용된 sentence classification CNN architecture
    - 몇 단어를 볼지(n-gram)를 위한 filter region size 변화(2, 3, 4)
    - pooling 전략
    - activation function 종류
    - L2, dropout 등 정규화 유/무

# 3. Experimental Results & Conclusion

![image](assets/imgs/2015-cnn-nlp/01.png)

- 비교를 위한 SVM baseline 성능

### 3.1. Word embedding

![image](assets/imgs/2015-cnn-nlp/02.png)

- Word2Vec과 GloVe는 둘다 각 word 당 300-dims
- 둘을 concat한 embedding도 사용해봄
- 성능은 거의 차이가 없음. 둘을 합친 embedding은 생각보다 성능이 별로임

### 3.2. Region Size

![image](assets/imgs/2015-cnn-nlp/03.png)

- 각 dataset 마다 최적의 region size(N-gram)이 있음
- Senetence의 길이에 따라 최적 길이도 달라짐

![image](assets/imgs/2015-cnn-nlp/04.png)

![image](assets/imgs/2015-cnn-nlp/05.png)

- dataset마다 최적 region size 조합이 다르긴 함. 하지만, 성능차이가 크지 않기 때문에 크게 의미는 없음을 알 수 있음

### 3.3. Number of feature map

![image](assets/imgs/2015-cnn-nlp/06.png)

- region size는 고정인 상태로 feature map 크기만 변경하여 진행
- dataset마다 적정 feature map이 다르지만, 일반적으로 600까지는 성능이 좋아짐

### 3.4. Activation Function

![image](assets/imgs/2015-cnn-nlp/07.png)

- ReLU, Tanh, Sigmoid, softplus, cube, tanh cube, iden의 7개 중 tanh, softplus, iden, ReLU가 가장 좋은 결과를 보임
- dataset마다 좋은 결과를 보인 activation function이 다르지만 일반적으로 tanh와 ReLU가 가장 성능이 좋았음

### 3.5. Pooling strategy

- region size와 pooling size를 모두 변경하며 테스트함
- average pooling도 같이 테스트해보았지만 max pooling에 비해 성능이 좋지 않음
- K-max pooling보다 1-max pooling이 더 성능이 좋음

### 3.6. Regularization

![image](assets/imgs/2015-cnn-nlp/08.png)

![image](assets/imgs/2015-cnn-nlp/09.png)

- dropout과 l2 norm에 대해 실험 진행
- l2 norm은 성능에 영향을 크게 미치지 못함
- dropout의 경우도 크게 성능에 영향을 주지 못했지만, penultimate layer(FC-layer)가 아닌 conv layer에 적용시켰을 때는 성능이 좋아짐

- 결론적으로는 대부분의 hyper parameter들이 결과에 큰 영향을 미치진 않지만, dataset에 맞추어 tuning은 필요함을 얘기함
    - region size, feature map size를 가장 우선적으로 선택하는 것이 좋음
    - max pooling의 경우는 1-max pooling이 가장 적합
    - word embedding은 word2vec, glove 중 원하는 걸로 선택
    - regularization은 거의 영향이 없으니 선택적 사용



# References

- [https://arxiv.org/abs/1510.03820](https://arxiv.org/abs/1510.03820)