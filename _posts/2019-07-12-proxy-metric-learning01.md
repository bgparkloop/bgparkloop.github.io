---
layout: post
title: "[Paper Review] No Fuss Distance Metric Learning using Proxies"
categories: [deep learning]
tags: [metric learning]
fullview: true
comments: true
---


## No Fuss Distance Metric Learning using Proxies

---
### 1. Introduction
Recognition, Retrieval 등 다양한 분야에서 사용되는 Metric Learning에서 Contrasitive loss, Triplet loss 기반의 학습법이 많이 유행했었다. 하지만 이 방법들은 sampling issue가 굉장히 큰 단점으로 존재해서 많은 연구자들이 이 부분을 해결하려고 노력했다. 최대한 좋은 효율로 학습할 수 있는 sample들을 추출하기 위한 방법들을 많이 연구했는데, 그만큼 연산량도 많이 필요해서 큰 덕을 많이 못보았다.  
이 논문에서는 이러한 문제점을 해결해줄 수단으로 Proxy라는 개념을 도입하여 학습 방법 자체를 바꾸었다. 기존의 방법들은 Positive-Negative의 한 쌍이나, Anchor-Pos-Neg의 한 쌍을 sample로 하여 학습이 진행되었는데, Proxy는 각 class의 대표값으로써 model의 한 parameter로 취급하여 학습한다. 그래서 기존의 classification처럼 cross-entropy loss로 학습이 가능해진다! 획기적인 발견인 것 같다.

<center><img src='{{ "/assets/images/proxy_02.PNG" | relative_url }}' width="500" height="200"></center>

위의 그림이 바로 Triplet과 Proxy의 차이점을 나타낸 것이다. 그림의 왼쪽은 Anchor를 기준으로 유사한 sample들을 가깝게하고 다른 것들을 멀리에 배치하는 것을 목표로 한다면, Proxy 기반의 학습법은 오른쪽 그림처럼 각 class에 해당하는 대표값(proxy)를 두어서 anchor의 embedding vector가 대표값에 가까워지게 학습을 한다.

---
### 2. Proposed Methods
저자들은 Proxy기반의 학습법으로 진행하기 위해 NCA(Neighborhood Component Analysis) loss라는 것을 제안한다. NCA loss는 아래 수식처럼 sample vector x를 proxy vector y에 가깝게 만드는 것이 목적이다. Z는 모든 샘플이 있는 set을 지칭한다.
<br><center><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white L_{NCA}(x,y,Z) = -\log(\frac{\exp(-d(x,y))}{ \sum_{z\in{Z}}\exp(-d(x,z))})"/> </center>
이렇게 Proxy 기반의 방법을 이용하면 기존의 Triplet의 엄청난 샘플링 연산량(Data수를 N으로 가정하면 최대 O(N^3))에서 엄청나게 줄일 수 있으며, loss 계산도 매우 간단해진다. 따라서, 최종적으로 아래 수식처럼 Loss를 sample vectors x와 z, y class의 proxy vector들 사이의 관계로 나타낼 수 있다.
<br><br><center><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white \begin{align*} L_{Ranking}(x,y,z) &= H(\left \| \alpha x -p(y) \right \| - \left \| \alpha x -p(z) \right \|) \\
&= H({\left \| \alpha x -p(y) \right \|}^2 - {\left \| \alpha x -p(z) \right \|}^2) \\
&= H(2\alpha (x^T p(z)-x^T p(y))) = H(x^T p(z) - x^T p(y)) \end{align*}"/> </center>
<br>
Proxy 방법을 사용하면, 기존의 방법들보다 학습 속도 및 정확도가 상당히 높아지는 것을 확인 할 수 있다.
<center><img src='{{ "/assets/images/proxy_01.PNG" | relative_url }}' width="500" height="300"></center>

---
### 3. Experimental Results
<center><img src='{{ "/assets/images/proxy_03.PNG" | relative_url }}' width="250" height="250"><img src='{{ "/assets/images/proxy_06.PNG" | relative_url }}' width="250" height="250"></center>
실험 성능은 기존 것들에 비해 상당히 우수하다. 위 그림처럼 Query image에 대해서도 유사도 거리에 따른 순서로 검색된 이미지들이 정답에 가깝게 잘 찾아주는 것을 확인 할 수 있다.

<center><img src='{{ "/assets/images/proxy_05.PNG" | relative_url }}' width="500" height="300"></center>
위 그래프는 기존의 다른 방법들과의 성능을 비교한 것으로, Recall@1의 성능이 가장 좋은 것을 확인 할 수 있다. 
- Recall@K : Query image에 대해 distance가 가장 작은 순으로 K개를 골랐을 때, 정답이 있을 확률. Recall@1이면 일반적인 Classification과 같다고 봐도 될 듯..?


<center><img src='{{ "/assets/images/proxy_09.PNG" | relative_url }}' width="500" height="300"></center>
저자들이 제안한 Proxy 방법은 label과 proxy가 1 to 1으로 매칭되는 것이 기본인데, 특이하게 multi label에 대해 proxy를 매칭하는 것을 실험하였다. Label 갯수보다 proxy 갯수가 적을 때와, 같을 때, 많을 때를 구분하여 실험을 진행하였다. 위 그래프처럼 Proxies per class가 0.5를 넘을 때부터 기존 방법들보다 좋은 성능을 보였다. 0.5 수치는 label 갯수가 proxy보다 2배 많은 경우를 이야기한다.


---
### 4. Conclusion
제안된 방법은 기존의 triplet loss 구조보다 간단하고, 메모리 및 연산량의 감소, 효율적인 학습률의 많은 장점이 있다. Contrasitive loss나 triplet loss 기반의 방법들보다 classification에 가까운 쉬운 방법론으로 인해 이해도 어렵지 않았다. 구현하기에도 까다롭지 않아 실 사용에도 어렵지 않다. 여러모로 구글이라는 생각이 든다. 


Paper : [https://arxiv.org/pdf/1703.07464.pdf](https://arxiv.org/pdf/1703.07464.pdf)  
Code : [https://github.com/dichotomies/proxy-nca](https://github.com/dichotomies/proxy-nca)