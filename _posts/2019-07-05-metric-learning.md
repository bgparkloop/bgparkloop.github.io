---
layout: post
title: "[Study] Metric Learning"
categories: [deep learning]
tags: [metric learning]
fullview: true
comments: true
---

### 1. How to measure similarity?
Image든 Character든 다양한 상황에서 Object 간 유사도를 측정하기 위해서는 어떻게 해야 할까? 사람이라면 같은 물체인 공이라고 해도 축구공과 농구공의 차이를 쉽게 판단할 수 있다. 더 나아가면 같은 축구공이라도 다른 브랜드에서 만들어 약간의 차이가 있는 점도 구분할 수 있을 것이다.  
그렇다면 컴퓨터는 이를 어떻게 구분할 수 있을까? 간단하게 생각하면 Machine Learning으로 두 Image들의 차이를 구분해줄 Feature를 만들어서 학습시키면 될 것이다. 하지만, 열심히 데이터를 모아서 학습해도 또 다른 class의 물체가 등장하면 이를 구분하기 어려울 것이다. 이런 문제점을 해결하기 위해 One-shot Learning이라는 방법이 등장했다.

<span style="width: 400px" class="image left"><img src="{{ '/assets/images/study/similarity-img-01.png' | relative_url }}" alt="" /></span> One-shot Learning은 말 그대로 1장의 Image만 있어도 그 object를 인식할 수 있다는 의미다. 구체적으로는 다양한 방법으로 이를 실천할 수 있겠지만, 가장 간단한 모델로는 kNN Clustering을 예로 들 수 있다.
왼쪽의 그림을 예로 One-shot Learning의 학습법을 살펴볼 수 있다. CNN을 기준으로 설명하면, CNN은 널리 알려진 모델 구조로 특히 Image에 대해 뛰어난 Feature 추출 성능을 갖는다. 이런 특성을 이용하여 위의 Verification Tasks에서는 CNN 모델의 Feature 추출 성능을 올리기 위해 대량의 데이터에 대해 parameter들을 학습한다. 학습된 모델은 실제로 test를 하고자 하는 task로 Transfer Learning을 진행한다. 이와 같은 Transfer Knowledge를 통해 다양한 object(또는 instance)를 인식할 수 있도록 한다.

Reference:
- [https://wwiiiii.tistory.com/entry/Pairwise-Triplet-Loss](https://wwiiiii.tistory.com/entry/Pairwise-Triplet-Loss)  
- [https://jayhey.github.io/deep%20learning/2018/02/06/saimese_network/](https://jayhey.github.io/deep%20learning/2018/02/06/saimese_network/)

---
### 2. How to use one-shot learning in Deep learning?</h2>
일반적인 CNN의 학습법(Classification)에서는 vanilla cross entropy loss를 계산하고 softmax를 통해 class를 분류한다. 이런 방식은 one-shot learning을 적용하기에는 수 많은 parameter들을 학습시키며 적은 데이터를 갖는 class에 대해서도 유의미한 분류성능을 얻기 어렵다. 즉, Overfitting이 발생한다.  
해결법은 앞서 설명된 Transfer Learning을 적용하여 유사하거나 다양한 object를 갖는 데이터셋에서 충분히 학습된 모델을 해결하고자 하는 task로 갖고 오는 것이다. 또한, 막단의 분류기로서의 부분을 Distance Metric을 계산하는 모양으로 변형하여 parameter를 줄이고 서로간의 유사도를 측정할 수 있게 해준다.

#### 2-1. Siamese Network
<p><span style="width: 400px" class="image right"><img src="{{ '/assets/images/study/similarity-img-02.jpg' | relative_url }}" alt="" /></span>Google에서 Face Recognition을 수행하기 위해 만들었던 Siamese Network는 Image분야에서 Similarity 학습을 위한 가장 기본적인 Deep Network 구조로 볼 수 있다. 샴(Siamese)이라는 말 그대로 같은 구조의 CNN 모델을 서로 다른 Image에 대해 수행하여 얻은 Feature Vector를 L1 Distance(L1 Norm)을 계산하여 두 Image간의 유사도를 측정한다.  
유사도 측정을 위한 Loss는 Contrasitive Loss라고도 부르며 Image1에 대한 Embedding Vector X1과 Image2에 대한 Vector X2를 D(X1, X2)와 같이 계산 후, Sigmoid를 통해 0에서 1사이로 Mapping하여 두 Image간의 유사도를 표현한다.</p>

Paper : [https://arxiv.org/abs/1503.03832](https://arxiv.org/abs/1503.03832)

#### 2-2. Triplet Network
<p><span style="width: 400px" class="image right"><img src="{{ '/assets/images/study/similarity-img-03.png' | relative_url }}" alt="" /></span>Triplet Network는 Siamese Network을 조금 더 발전시킨 것이다. 기존의 2개의 Image간 유사도를 측정하였다면, Anchor라 불리우는 Reference Image를 사이에 두고 같은 Class인 Positive Sample과 다른 Class인 Negative Sample을 하나의 Pair로 하여 말 그대로 Triplet(세 쌍둥이)를 형성한다.</p>

<span style="width: 400px" class="image right"><img src="{{ '/assets/images/study/similarity-img-04.jpg' | relative_url }}" alt="" /></span>학습의 진행은 아래의 그림과 같다. Anchor와 Positive의 관계는 서로를 Pull하는 효과를 보이고, Anchor와 Negative는 서로를 Push하여 같은 class간 variation은 작고, 다른 class간 variation은 크게 만드는 효과가 있다.  
Siamese와 또 다른 점이 하나 있는데, Margin이라는 변수가 추가되어 학습을 돕는다. Margin은 아래 그림과 같은 역할을 담당한다. 적절한 크기의 Margin을 Loss Term에 추가함으로써, Anchor-Positive간 거리와 Anchor-Negative간 거리를 조절하고 학습의 수렴을 돕는다.

Reference:  
- [https://arxiv.org/abs/1703.07737](https://arxiv.org/abs/1703.07737)
- [https://github.com/adambielski/siamese-triplet](https://github.com/adambielski/siamese-triplet)
- [https://github.com/andreasveit/triplet-network-pytorch](https://github.com/andreasveit/triplet-network-pytorch)


#### 2-3. What's best triplet selection?
Triplet Network 학습에 있어서 Pair의 Selection은 매우 중요하다. 똑같이 학습에 사용되는 Pair라도 당연히 이미 잘 분류되게 짝지어진 Pair가 있을 것이고 어려운 것이 있기 때문이다. 그렇다면 어떻게 Pair를 선택해야 학습에 도움을 줄 수 있을까?

1. 첫 번째 방법으로는 Loss가 크게 되는 것만 고르는 것을 생각해 볼 수 있다. 말 그대로 A-P의 Distance는 크고, A-N의 Distance는 작은 것들만 고르는 것이다.(이런 것들을 Hard Positive, Hard Negative라고 함) 하지만 이런 방법은 Outlier에 overfitting이 될 확률이 크고 이를 샘플링하기 위한 계산량이 너무 많아 좋지 않다.
2. 두 번째 방법으로는 학습에 사용될 Mini-Batch에 Anchor와 같은 1개 class에 대해서 A-P를 적정 수를 넣고, A-N에 대해서 랜덤으로 추출하여 batch를 구성하는 것이다. 이 방법은 실제 실험적으로 Hard Positive, Hard Negative만 사용하는 것보다 연산량 및 학습에 도움이 된 것으로 증명되었다. 또한, 이 방법에서 A-N의 Sampling을 진행 할 때는 짝이 되는 A-P의 Loss보다 큰 것들로만 뽑는 것이 좋다고 한다.(이는 충분히 P와 N 사이의 Margin이 크지 않은 것으로 뽑는다고 볼 수 있다.)

---
### 3. Other Networks
#### Classification based metric learning using proxy
앞서서 classification으로 metric learning이 안된다고 했는데 어째서 classification으로 진행하는가! 논문을 참조하면 무릎을 탁 치게 된다. 기존의 방법들 특히, Triplet Loss의 경우 Triplet sample을 뽑는 것이 주 관점인데 이는 합리적인 생각이지만 많은 단점이 존재한다. 그래서 더 간단하게 연산량을 줄이며 좋은 학습법이 없을까하여 나온 것이 Proxy라는 개념이다.

<span style="width: 400px" class="image left"><img src="{{ '/assets/images/study/similarity-img-05.png' | relative_url }}" alt="" /></span>왼쪽 그림을 보면 그림의 왼쪽부분이 이전부터 사용해오던 Triplet의 개념이다. 여기서 저자는 Proxy 개념을 도입하여, 해당 Class를 대표하는 1개의 Embedding Vector를 선정하여 이 Vector와 같은 Class들의 Embedding Vector의 Distance를 줄이고 타 Class의 Proxy Vector와 Distance는 멀게하여 적은 연산량으로도 매우 효율적인 Metric Learning을 진행할 수 있게 하였다. 심지어 여기서는 Softmax를 사용하여 Multi class learning을 진행한다.  
기존의 Triplet에서 같은 모델로 3번이나 Feature를 추출하여 많은 GPU Memory 소모와좋은 Sampling을 하기 위한 많은 연산이 필요없어져 그냥 output이 좀 더 긴 vector로 된 Classification이 되어 Large Batch가 가능하게 되었다. (말 그대로 학습 효율이 엄청 올라버렸다.) 실제로 Triplet과 Proxy based method로 Fashion MNIST 같은 것을 돌려보면 학습하는 속도도 결과도 많은 차이를 보여준다.

Paper : [https://arxiv.org/pdf/1703.07464.pdf](https://arxiv.org/pdf/1703.07464.pdf)
Github : [https://github.com/dichotomies/proxy-nca](https://github.com/dichotomies/proxy-nca)