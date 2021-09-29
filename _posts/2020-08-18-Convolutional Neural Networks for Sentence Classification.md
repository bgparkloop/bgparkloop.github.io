---
title: "[Review] (2014) Convolutional Neural Networks for Sentence Classification"
categories:
  - NLP
tags:
  - CNN
  - Deep Learning
  - NLP
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 1. Introduction

- 일반적으로 문장 분류는 단어를 임베딩(1 to V mapping)으로 일정 길이의 vector로 변환하고 이를 통해 문장의 의미나 감정분석 등 다양한 분류를 진행함
- 임베딩끼리의 비교는 Euclidean이나 Cosine distance 등을 주로 사용함
- 특히, 문장의 분류와 같은 NLP task는 인간의 언어 사용과 유사한 RNN을 사용하였는데, CNN을 통한 NLP task를 풀고자 연구함
    - Word를 Word2Vec방식을 이용하여 임베딩
    - senetence에 대해 simple CNN구조를 이용하여 word embedding사이의 관계를 분석함

# 2. Methods

![image](/assets/imgs/paper/2014-cnn-nlp2/00.png)

- 한 문장에 나오는 단어들(N개)에 대해 임베딩(K sized vector)로 만들어 concat함
- NxK region을 입력으로 받고, Convolution filter 크기는 한 번에 보고자 하는 단어 수 x 임베딩 크기로 지정함 (i x k, i=2, 3, 4, 5,...)
- Activation은 Tanh을 이용함
- Conv filter를 거쳐 나온 output은 1-max-pooling(global max pooling)하여 concat함
- dataset에 맞는 output size로 fully-connected layer를 통해 결과 도출함
- FC-layer에는 dropout을 통한 Regularization을, weight vector들은 L2 Regularization을 도입

# 3. Experimental Results & Conclusion

### 3.1. Dataset

- MR : 영화 리뷰 당 한 문장으로 이루어짐. 긍정/부정적인 리뷰인지 분류하는 데이터셋
- SST-1 : MR의 확장 버전으로 5개의 클래스(매우긍정, 긍정, 중립, 부정, 매우 부정)으로 이루어짐
- SST-2 : SST-1과 같은데 중립 클래스를 제거하여 binary class로 만든 것
- Subj : 문장에 대해 주관적, 객관적으로 분류하는 데이터셋
- TREC : 6개의 질문 종류로 분류하는 데이터셋
- CR : 제품 고객리뷰의 긍정/부정 예측 데이터셋
- MPQA : 의견의 polarity 검출 데이터셋

### 3.2. Pre-trained Word Vectors

- Google News로부터 얻어진 100 bilion words로 학습한 Word2Vec이용
- pre-trained word에 없으면 무작위 초기화하여 사용

### 3.3. Model Variations

- CNN-rand : 기본 무작위 초기화한 CNN 모델
- CNN-static : pre-trained word2vec으로 학습한 모델
- CNN-non-static : CNN-static과 같지만 각 task에 대해 fine-tuning한 것
- CNN-multichannel : 2개의 다른 dataset을 같이 입력으로 한 모델

![image](/assets/imgs/paper/2014-cnn-nlp2/01.png)

### 3.4. Multichannel vs Sing Channel

- multi channel을 이용하면 정규화가 잘되고 overfitting을 방지할 줄 알았지만, 그에 따른 fine-tuning이 별도로 필요하여 더 복잡해짐

### 3.5. Static vs Non-static

![image](/assets/imgs/paper/2014-cnn-nlp2/02.png)

- Non-static model의 경우 pre-trained에 없는 word에 대해 fine-tuning을 하여 static model보다 더 정밀한 결과를 얻을 수 있음

- 상당히 간단한 CNN 모델인데도 기존의 RNN 모델들과 유사하거나 좋은 성능을 보이는 점이 놀라움
- CNN의 convolution filter 구조 상 인접한 word 간의 연결성을 파악에 용이하기 때문인듯 싶음



# References

- [https://www.aclweb.org/anthology/D14-1181/](https://www.aclweb.org/anthology/D14-1181/)