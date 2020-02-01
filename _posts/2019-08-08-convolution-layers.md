---
layout: post
title: "[Study] Types of Convolution Layer"
categories: [deep learning]
tags: [cnn, convolution layer]
fullview: true
comments: true
---
## Convolution Layer의 종류들
### 1. Basic 2D Convolution Layer
<center><img src='{{ "/assets/images/basic-conv.gif" | relative_url }}' width="300" height="300"></center>
Convolutional layer는 5개의 Component로 이루어진다.
- Input Channel : 말그대로 입력할 데이터의 크기이다. 512x512와 같은 사이즈를 말한다.
- Output Channel : 출력할 데이터의 크기이다.
- Kernel Size : Convolutional layer가 한 번 연산 시 Feature를 추출할 영역을 의미. 얼만큼의 영역을 지켜볼 지(View)라고도 할 수 있다. 일반적으로 3x3 크기를 가장 널리 사용한다.
- Stride : Convolution 연산을 실행 할 때, 얼만큼 건너뛰며 진행할지 결정한다. 기본크기는 1로 지정되어 한칸 씩 움직이지만, Down sampling을 위해 2나 3과 같은 값도 사용한다.
- Padding : Input 또는 Output 데이터 테두리(border)에 값을 얼만큼 추가할지 결정한다. Convolution 연산을 수행하면 Input보다 Output 크기가 줄어들기 때문에 Input과 Output 크기를 맞춰주기 위해 보통 사용한다.

Computational Cost
- W : Input Width
- H : Input Height
- C : Input Channel
- K : Kernel Size
- M : Output Channel
Basic Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white K^2 CMHW"/>

---
### 2. Dilated Convolution Layer
<center><img src='{{ "/assets/images/dilated-conv.gif" | relative_url }}' width="300" height="300"></center>
말 그대로 확장된 Convolution layer로 같은 3x3 kernel을 갖더라도 kernel 내 간격이 달라져 view가 넓어지는 효과를 낸다. Segmentation 또는 Object detection과 같은 task에서 dilation을 각기 다르게 준 Convolution layer들을 이용하여 다양한 물체 크기를 한 번에 학습하기에도 용이하다. 이 Layer를 이용하면 pooling을 넣지 않아도 receptive field를 크게 가질 수 있다. 
> receptive field: kernel(filter)가 한 번에 볼 수 있는 영역

---
### 3. Grouped Convolution Layer
<center><img src='{{ "/assets/images/group-conv.png" | relative_url }}' width="450" height="300"></center>
나머지는 일반적인 Convolution Layer와 같지만, Input의 Channel 단위를 group으로 나누어 계산하는 점이 다르다. N개의 채널이 있다면 이를 g만큼의 그룹으로 나누어 각 그룹마다 같은 kernel을 사용하여 연산량이 많이 줄어든다.
> 병렬처리에 우수하며, 기존의 conv layer보다 연산량이 많이 줄어들고, 각 그룹에 높은 correlation을 갖는 channel 학습이 가능함.

Computational Cost
- W : Input Width
- H : Input Height
- C : Input Channel
- K : Kernel Size
- M : Output Channel
- g : # of Group
Grouped Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white {(K^2 CMHW)}/g"/>

---
### 4. Separable Convolution Layer

### 4.1. Depth-wise(Channel-wise) Convolution Layer
<center><img src='{{ "/assets/images/depth-conv.png" | relative_url }}' width="300" height="300"></center>

Computational Cost
- W : Input Width
- H : Input Height
- C : Input Channel
- K : Kernel Size
- M : Output Channel
Basic Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white K^2 CMHW"/>

#### 4.2. Point-wise Convolution Layer
<center><img src='{{ "/assets/images/point-conv.png" | relative_url }}' width="300" height="300"></center>

Computational Cost
- W : Input Width
- H : Input Height
- C : Input Channel
- K : Kernel Size
- M : Output Channel
Basic Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white K^2 CMHW"/>


---
### 5. Transpose Convolution Layer
<center><img src='{{ "/assets/images/transpose-conv.gif" | relative_url }}' width="300" height="300"></center>

---
### 6. 3D Convolution Layer


---
Reference
1. [https://zzsza.github.io/data/2018/02/23/introduction-convolution/](https://zzsza.github.io/data/2018/02/23/introduction-convolution/)
2. [https://hichoe95.tistory.com/48](https://hichoe95.tistory.com/48)