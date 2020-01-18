---
layout: post
title: "[Paper Review] Metric Learning 02"
categories: [deep learning]
tags: [metric learning]
fullview: true
comments: true
---


## Classification is a Strong Baseline for Deep Metric Learning

---
### 1. Introduction
이전의 Proxy 기반의 방법을 이어받아 좀 더 발전시킨 논문이다. 기본적으로 CNN의 Convolutional feature 부분은 그대로 가져가고 NCA 대신 저자들이 제안하는 부분으로 변경하여 성능을 끌어올렸다.  
NCA와 마찬가지로 Classification 형태로 학습이 진행되고, 기존의 triplet based 방법들의 문제들은 해결이 된다. 하지만, classification based 방법에서는 extreamly multi label을 갖는 문제에서 scalability 이슈가 있어 이 부분을 해결하는 방법을 추가하였다.

---
### 2. Proposed Methods
논문에서는 크게 3가지에 대해 이야기하고 있으며, 전체적인 Flow는 아래와 같다.
<center><img src='{{ "/assets/images/proxy_10.PNG" | relative_url }}' width="600" height="400"></center>


#### 2.1. Normalized Softmax Loss
저자들은 기존의 FC Layer에서 bias term을 없애고, Cosine similarity를 최적화해줄 L2 Normalization을 추가하였다. 또, Softmax 안에 Temperature scaling <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white \sigma"/>를 두어 클래스들 간의 학습률의 차이를 boosting하는 방식을 사용하였다.

<br><center><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white L_{Norm} = -\log(\frac{\exp(x^T p_y / \sigma)}{ \sum_{z\in{Z}}\exp(x^T p_z / \sigma)})"/> </center>

#### 2.2. Layer Normalization



#### 2.3. Class Balanced Sampling

---
### 3. Experimental Results


---
### 4. Conclusion



Paper : [https://arxiv.org/pdf/1811.12649.pdf](https://arxiv.org/pdf/1811.12649.pdf)  
Code : [https://github.com/azgo14/classification_metric_learning](https://github.com/azgo14/classification_metric_learning)