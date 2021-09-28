---
title: "[Review] (2016) Deep Neural Networks for YouTube Recommendations"
categories:
  - Recommendation
tags:
  - DeepLearning
  - Recommendation
  - Youtube
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 1. Introduction

- Youtube는 매우 많은 유저와 video가 있기 때문에 3가지를 고려해야 함
    - Scale: 많은 추천 알고리즘들이 작은 문제들에 대해 잘 동작하는데 Youtube와 같은 거대한 데이터에 대해 잘 동작하지 않음. 그래서 많은 유저, video를 핸들링할 수 있는 효율적인 serving 시스템과 특별한 알고리즘이 필요함
    - Freshness: 계속해서 수많은 데이터가 업로드되므로 이에 대해서 뿐만 아니라 유저가 최근 한 행동(action)에 대해서도 빠르게 반영이 되야함.
    - Noise: 유저들의 행동 흐름은 sparse하고 관측할 수 없는 외부적인 요인들이 있기 때문에 예측하기 어려움. 그래서 feedback에 대해서는 noise가 섞인 정답만 얻으므로 완전한 Ground-truth를 얻기 어렵다. content들의 metadata들은 잘 정의가 안되있으므로 이런 특징을 고려하여 알고리즘을 만들어야 함.

# 2. Methods

![image](/assets/imgs/2016-youtube-net/00.png)

- 전체 framework. Candidate generation, ranking의 2단계로 구성됨

## 2.1. Candidate Generation Network

- 유저의 YouTube activity history를 입력으로 받고 small subset video를 검색해줌 with high precision
- provides broad personalization via collaborative filtering
- 유사도는 비디오를 본 ID들, query tokens, demographic과 같은 coarse feature들로 표현함

### 2.1.1. Recommendation as Classification

- 수많은 비디오 중 유저가 본 video일 확률
    - $P(w_t=i|U,C)=\frac{e^{v_i u}}{\sum_{j\in V} e^{v_j u}}$
    - $w_t$ : 시간 t에 본 특정 비디오 w
    - $i$ : Video (or Class)
    - $u$ : User와 Context pair의 high-dimension embedding
    - $v$ : Candidate video embedding
    - $V$ : Corpus
    - $U$ : User
    - $C$ : Context
- YouTube 자체 explicit feedback 메커니즘(좋아요, 싫어요 등)이 있지만, implicit feedback을 모델 학습에 사용함

### 2.1.2. Efficient Extreme Multiclass

- 수 백만개의 video class중에 일부를 선택하는 process는 serving time에 행하기가 매우 어려움
- 이전 알고리즘은 hash 기반으로 하였지만 지금은 negative sampling 기반의 알고리즘 사용
- Background distribution에서 수 천개의 negative sample을 추출하고 importance weighting으로 sampling을 보정하는 방식을 이용함
- 이 방식은 기존 softmax 방법보다 학습속도를 100배 이상 빠르게함
- 학습된 likelihood는 nearest neighboring으로 class 분류 수행함

### 2.1.3. Model Architecture

![image](/assets/imgs/2016-youtube-net/01.png)

- Input은 고정된 크기의 concatenated vector로 구성됨
    - video embedding, user's information, context, etc
- 전체 hidden layer들은 fully-connected layer와 ReLU로 구성됨

### Heterogeneous Signals

- Neural network가 matrix factorization의 일반화로써 갖는 장점은 임의의 연속적이고 범주화된 특징들을 쉽게 추가할 수 있다는 점임
- Search history는 watch history와 유사하게 다뤄지며, 각 query는 token화하여 unigram 또는 bigram으로써 embedding함
- embedding vector를 평균내면, 요약된 dense search history가 됨
- Demographic feature는 새로운 유저에 대해 prior를 제공하여 중요함
- 유저의 geographic region과 사용하는 device, gender, logged-in state, age 등의 정보도 결합
- 모든 정보는 0-1로 normalize하여 input으로 들어감

### Example Age Feature

![image](/assets/imgs/2016-youtube-net/02.png)

- 유저가 관련성이 있으면서 fresh한 content(new video)를 선호한다는 것을 관측함
- ML 모델은 과거를 통해 미래를 예측하지만 bias가 있고, 실제 video의 인기 분포는 매우 유동적임.
- 그래서 training example의 age(신선도-freshness)를 feature로써 학습시킴. serving time에서 처음에는 0또는 음수로 시작하다가 점점 증가함

### Label and Context Selection

![image](/assets/imgs/2016-youtube-net/03.png)

- 추천 시스템에서 surrogate problem(대리 모델을 통한 문제 해결)이 많은데, 이런 경우는 A/B testing의 성능 측정에서 매우 중요하지만 offline 테스트 환경에서 정확한 측정이 어렵다.
    - 학습 예제는 모든 YouTube 비디오(심지어 다른 사이트 비디오들)를 사용함. 이렇게 안하면 새로운 content가 노출되기 어렵고, 추천자가 편향된 정보를 사용할 수 있음
    - 유저마다 고정된 수의 학습예제를 생성하여 동등한 가중치로 loss 함수에 영향을 끼치게 함. 이는 소수의 활동이 많은 유저들이 loss 함수에 많은 영향을 끼치지 않게 하기 위함

### Experiments with Features and Depth

![image](/assets/imgs/2016-youtube-net/04.png)

- Feature들의 종류와 Network depth와 관련된 실험을 수행
    - Feature 종류(시청한 목록, 검색 목록, 등록된지 얼마나 지났는지(example age), 등등) 많은 feature를 사용할 수록 더 높은 성능을 보임
    - Depth의 경우 tower처럼 2048→1024→512 등으로 절반으로 차원이 떨어지는 구조로 늘려갔으며, 성능은 더 좋아지긴 하지만 수렴이 어려워짐

## Ranking

- 높은 확률로 추천해준 비디오라도 썸네일 등이 맘에 안들어서 선택 안될 수 있음
- 유저의 관계성(선호도?)는 수백만개의 비디오가 아닌 수백개의 비디오를 추천해주는 것이기 때문에 이런 비디오들과 유저 사이의 관계를 잘 설명해줄 feature들이 많이 필요함
- Best 추천은 high recall을 갖는 candidate 사이를 구별 할 수 있는 fine-level representation이 필요함
- video와 user 사이 관계를 표현해줄 rich set of features로 score를 매겨 순위를 부여함

### Feature Representation

- 다양한 방식으로 feature는 정의됨
    - binary형태(logged-in), multi-variant(유저가 마지막으로 본 영상들)

### Feature Engineering

- DNN이 직접 feature를 만드는 것을 많이 줄여주었지만, 실제로 raw data를 그냥 쓰지 않고 어느정도 가공해야함
    - 특히, 시간순으로 유저의 행동들이 video가 노출되는 것과의 관계성처럼
- 중요한 signal들은 item과 연관된 유저들의 지난 상호작용들임
    - 특정 채널에서 업로드된 영상들이 지난 시간동안 얼마나 시청되었는가
    - 유저가 이와 유사한 토픽의 비디오를 본적이 있는가
- 영상들이 과거 노출된 빈도를 묘사하는 특징도 중요함
    - 특정 topic이나 사람의 비디오를 추천하였는데 이를 안본다면 자연스럽게 그 다음에는 랭킹 순위가 내려가서 추천을 안해줌

### Embedding Categorical Features

- sparse한 categorical feature를 dense representation으로 바꾸기 위해 embedding을 사용함
    - vocabulary와 같은 unique ID space는 look up table등을 이용하고 근사치로 학습
    - Very large cardinality ID space(search query terms)는 빈도수에 따른 top N개를 빼고 폐기처분함
    - 같은 ID space의 categorical feature들은 같은 embedding을 공유함
    - Embedding sharing은 memory 소모 감소, 일반화, 학습속도 증가 등 다양한 이점이 있음

### Normalizing Continuous Features

- NN은 입력값의 분포나 크기 단위에 굉장히 민감함
- 반면, Decision tree와 같은 방식은 강건함
- Input으로 일반적인 normalization $\tilde{x}$ 말고도, $\tilde{x}^2$나 $\sqrt{\tilde{x}}$을 입력으로 넣으면 다양한 표현이 가능해져 성능 향상을 돕는다

### Modeling Expected Watch Time

![image](/assets/imgs/2016-youtube-net/05.png)

- 얼마나 video를 시청할지 시간을 예측하기 위해 weighted logistic regression도입

### Experiments with Hidden Layers

![image](/assets/imgs/2016-youtube-net/06.png)

- 1024→512→256 연속 모델이 가장 성능이 좋음
    - input을 normalization 안해봤더니 loss가 0.2% 상승
    - weight를 positive, negative에 동등하게 주었더니 loss가 4.1% 상승

# Conclusion

- 제안한 Deep collaborative filtering 모델은 기존에 적용하던 Matrix Factorization보다 훨씬 효과적임
- 미래 정보 유출을 예방하고 실시간 측정 기준에서 잘 동작하기 위해 비대칭적인 동시 시청 동작에 대해 잘 캐치하고 이를 대리 문제 해결에 잘 도입함
- 특히, Example age의 도입은 모델이 인기있는 동영상을 잘 추천하도록 해줌
- Weighted logistic regression은 시청을 한(positive) 경우 시청 시간으로 가중치를 부여하였고, 아닌 경우는 동일한 가중치를 두어 예상 시청 시간을 잘 모델링함
- YouTube가 적용한 추천 알고리즘 방식이 대략적으로 어떤지 알 수 있었고, 많은 고민의 흔적이 보인다. 아직 CF, MF 등의 방법이나 추천 시스템 고유의 방법론들을 잘 몰라 이해가 부족한 것이 아쉬운 점.

# References

- [https://research.google/pubs/pub45530/](https://research.google/pubs/pub45530/)