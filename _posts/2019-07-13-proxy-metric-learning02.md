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
저자들은 기존의 FC Layer에서 bias term을 없애고, Cosine similarity를 최적화해줄 L2 Normalization을 추가하였다. 또, Softmax 안에 Temperature scaling <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white \sigma"/>를 두어 클래스들 간의 학습률의 차이를 boosting하는 방식을 사용하였다. 논문에서는 이 값을 0.05로 고정하였다. 이에 대해 정리된 수식은 다음과 같다.
<br><center><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white L_{Norm} = -\log(\frac{\exp(x^T p_y / \sigma)}{ \sum_{z\in{Z}}\exp(x^T p_z / \sigma)})"/> </center>

#### 2.2. Layer Normalization
Affine parameter가 없는 Layer Normalization을 Convolutional feature 바로 다음에 적용함으로써, embedding vector를 zero-centered로 만들어 0을 기준으로 binarize embedding하기 쉽게 만들어 주었다.

#### 2.3. Class Balanced Sampling
Scalability issue를 해결하기 위한 Mini-batch안에 최적의 class수를 정하도록 batch size 내에서 balance 있는 class sampling 방법을 제안하였다. 후에 실험 결과를 통해 class당 몇 장의 image를 mini-batch에 포함시켜야 최적의 학습효율이 나오는지 보여준다.

---
### 3. Experimental Results
#### 3.1. Loss Function Comparison
<center><img src='{{ "/assets/images/proxy_12.PNG" | relative_url }}' width="500" height="800"></center>
위 그림은 각 Loss function 별로 성능을 비교한 것이다. Loss Function 옆의 작은 숫자는 embedding vector의 크기를 나타낸 것이다. SOP, CARS-196 dataset에서는 NormSoftmax가 가장 뛰어난 성능을 보였으며, CUB-200에서도 1~2위를 다투는 좋은 성능을 보여준다. Table의 성능 비교를 통해 NormSoftmax가 학습 효율성이 좋은 Loss Function임이 증명된다.

#### 3.2. Embedding Dimensionality
<center><img src='{{ "/assets/images/proxy_11.PNG" | relative_url }}' width="600" height="250"></center>
다음으로는 Embedding vector의 크기 및 정밀도(float or binary)에 따라 성능 차이를 비교하였다. 그래프에서 보듯 차원 크기가 커질수록 당연히 예상하게도(?) 성능이 좋아짐을 볼 수 있다. 그래프에서는 Float와 Binary의 2가지 정밀도도 비교를 하고 있는데 Float는 embedding vector를 Layer Normalization으로 zero-centered로 바꾼 뒤 그대로 사용한 것이고, Binary는 이 상태에서 0을 기준으로 threholding하여 각 dimension마다 0 또는 1의 binary형태로 변환한 것을 의미한다. 차원의 크기가 커질수록 2개의 성능은 유사해지긴 하지만, float 형태가 기본적으로 정보량이 더 풍부하기 때문에 성능의 우세함을 보인다. 메모리를 아끼려면 binary 형태로 사용하는 것도 고려해 볼만 하다.

#### 3.3. Layer Normalization 유/무
<center><img src='{{ "/assets/images/proxy_13.PNG" | relative_url }}' width="600" height="250"></center>
Layer Normalization의 유/무에 따라 Loss가 수렴하는 정도와 성능의 비교를 보여준다. 확실히 적용을 할 때가 수렴 속도, 최저값, 성능 모두 더 나은 것을 볼 수 있다.

#### 3.4. Class Balanced Sampling 실험
<center><img src='{{ "/assets/images/proxy_14.PNG" | relative_url }}' width="500" height="150"></center>
SOP dataset(클래스 수가 약 1,000개)에 대해 mini-batch에 class 비율을 1% ~ 10%까지 변화시켜 테스트하였다. 사실상 엄청난 차이를 보이지는 않지만, 그래도 balanced sampling을 하는게 안하는 것보다는 낫다고 저자는 말한다. (실제로 회사에서 적용할 때, 적용하는 편이 그래도 수렴이 빨라 더 효율적이었다. 계속 학습하면 결국 거의 같아지긴 하였다. 시간 절약을 위해 적용하는 것이 좋겠다.)

---
### 4. Conclusion
NCA와 마찬가지로 크게 어려울 것 없는 논문이었다. NCA에서 다소 부족했던 점을 보충하여 학습 속도와 성능을 모두 끌어올려 훌륭한 것 같다. 논문에서는 SOTA들과 성능 비교한 table도 올려두었는데 Recall@1의 경우 1~3% 더 좋았고, Recall@10, Recall@100의 경우도 비슷한 정도로 좋았다. 이 방법도 결국 Proxy라는 형태를 도입한 것이 가장 큰 차별점이며, proxy가 좀 더 효율적으로 학습될 수 있도록 다양한 방법을 시도해 본 것 같다.


Paper : [https://arxiv.org/pdf/1811.12649.pdf](https://arxiv.org/pdf/1811.12649.pdf)  
Code : [https://github.com/azgo14/classification_metric_learning](https://github.com/azgo14/classification_metric_learning)