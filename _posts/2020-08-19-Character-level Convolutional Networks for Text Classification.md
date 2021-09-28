---
title: "[Review] (2015) Character-level Convolutional Networks for Text Classification"
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

- 일반적으로 text classification은 word 기반으로 수행됨(N-grams)
- CNN은 computer vision 분야에 특화되었지만 sequential data에도 간간히 사용되어옴
- 제안하는 방법은 character level에서의 raw signal을 CNN을 사용하여 분석하는 방법
- character 기반의 학습은 중간에 비정상적인 character가 나타나더라도(이모션같은) 자연스럽게 학습해서 괜찮음

# 2. Methods

### 2.1. Key Modules

- 1-D Convolution
    - 일반적인 2D conv가 아닌 1-D로 character 각각에 대해 convolution 진행
    - stride를 두어 일정 간격만큼 이동하며 수행
- Temporal Max-pooling
    - 1-D 방향의 일정 범위 내에서 max-pooling
    - stride를 두어 1-D convolution과 같이 수행
- Activation은 ReLU, Optimizer는 SGD를 사용

### 2.2. Character Quantization

- $m$ 크기의 one-hot vector로 character를 정의함
- sequence의 길이는 $l_0$로 제한함. 이를 넘기면 무시
- blank character를 포함한 알파벳이 아닌 character는 all-zero vector로 변환함. 알파벳의 정의는 다음과 같음

![image](/assets/imgs/paper/2015-cnn-nlp2/00.png)

### 2.3. Model Design

![image](/assets/imgs/paper/2015-cnn-nlp2/01.png)

- 모델은 Large/Small의 2가지 형태로 구성함
    - Input feature의 크기는 1014로 고정
    - Large 모델은 first feature 1024, 8th feature 2048
    - Small 모델은 first feature 256, 8th feature 1024
- 총 9개의 layer를 갖음(6개의 convolution과 3개의 FC layer로 구성함)
- dropout은 3개의 FC layer 사이에 2개를 넣음(정규화를 위해)

### 2.4. Data Augmentation using Thesaurus

- text에서는 기존의 signal transformation은 적합하지 않은 방식임
- LibreOffice, Word-Net 등을 통해 유의어 데이터셋 구성
- input이 있으면 대체 가능한 word들 중 $r$개를 선택하여 대체하여 augmentation 진행 확률 $p^r$
- 대체하기로 한 word에 대한 동의어 목록 중 $s$번째에 해당하는 단어(동의어 출현 빈도수)로의 변환 확률 $q^s$

# 3. Experimental Results & Conclusion

다양한 조건들의 방법들로 테스트 진행

### 3.1. Traditional Methods

- BoW + TFIDF : BoW 생성 후 이에 대한 TFIDF feature를 생성하여 분류기 학습
- BoN-grams + TFIDF : N-gram(up to 5-grams)의 Bag of grams를 만들어서 이를 TFIDF feature 추출 후 학습
- BoMeans on word embedding : word2vec을 k-means로 clustering하여 학습

### 3.2. Deep Learning Methods

- Word2vec을 ConvNet 기반 학습
- Word2vec을 LSTM 기반 학습

### 3.3. Choice of Alphabet

- 대/소문자를 구별하는 것은 의미도출에 큰 의미를 갖지 않으므로 안하는 것이 성능이 더 좋았음

### 3.4. Datasets

![image](/assets/imgs/paper/2015-cnn-nlp2/02.png)

### 3.5. Testing Errors

![image](/assets/imgs/paper/2015-cnn-nlp2/03.png)

- Bag-of-means가 가장 성능이 안좋음.
- 전통적인 방법이라고 무조건 안좋지 않음(ngrams TFIDF의 경우 딥러닝 기반보다 상당히 좋음)
    - small dataset에서는 오히려 전통적인 방식이 효과가 있음
- 유의어를 통한 augmentation이 생각보다 효과가 있음
- 생각보다 Character based CNN도 text classification에 잘 동작함
    - ConvNet은 사용자가 만든 임의의 데이터셋에서도 잘 동작(오타에 강건함)
- 모든 데이터셋에 최적화된 모델은 없음



# References

- [https://arxiv.org/pdf/1509.01626.pdf](https://arxiv.org/pdf/1509.01626.pdf)