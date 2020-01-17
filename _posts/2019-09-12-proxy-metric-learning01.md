---
layout: post
title: "[Paper Review] Metric Learning 01"
categories: [deep learning, metric, learning, proxy]
tags: [cnn, metric, learning, proxy]
fullview: true
comments: true
---


## No Fuss Distance Metric Learning using Proxies

---
### 1. Introduction
Recognition, Retrieval 등 다양한 분야에서 사용되는 Metric Learning에서 Contrasitive loss, Triplet loss 기반의 학습법이 많이 유행했었다. 하지만 이 방법들은 sampling issue가 굉장히 큰 단점으로 존재해서 많은 연구자들이 이 부분을 해결하려고 노력했다. 최대한 좋은 효율로 학습할 수 있는 sample들을 추출하기 위한 방법들을 많이 연구했는데, 그만큼 연산량도 많이 필요해서 큰 덕을 많이 못보았다.
이 논문에서는 이러한 문제점을 해결해줄 수단으로 Proxy라는 개념을 도입하여 학습 방법 자체를 바꾸었다. 기존의 방법들은 Positive-Negative의 한 쌍이나, Anchor-Pos-Neg의 한 쌍을 sample로 하여 학습이 진행되었는데, Proxy는 각 class의 대표값으로써 model의 한 parameter로 취급하여 학습한다. 그래서 기존의 classification처럼 cross-entropy loss로 학습이 가능해진다! 획기적인 발견인 것 같다.

<center><img src='{{ "/assets/images/proxy_02.PNG" | relative_url }}' width="700" height="300"></center>

위의 그림이 바로 Triplet과 Proxy의 차이점을 나타낸 것이다. 그림의 왼쪽은 Anchor를 기준으로 유사한 sample들을 가깝게하고 다른 것들을 멀리에 배치하는 것을 목표로 한다면, Proxy 기반의 학습법은 오른쪽 그림처럼 각 class에 해당하는 대표값(proxy)를 두어서 anchor의 embedding vector가 대표값에 가까워지게 학습을 한다.

---
### 2. Proposed Methods
저자들은 Proxy기반의 학습법으로 진행하기 위해 NCA(Ceighborhood Component Analysis) loss라는 것을 제안한다. NCA loss는 아래 수식처럼 sample vector x를 proxy vector y에 가깝게 만드는 것이 목적이다.
<br><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white L_{NCA}(x,y,Z)=-log(\frac{exp(-d(x,y))}{\sum_{z\inZ}exp(-d(x,z))})"/>   



---
### 3. Experimental Results
