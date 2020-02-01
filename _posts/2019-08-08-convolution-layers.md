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
Separable Conv Layer는 2개의 part로 나뉘며, 보통 이 2개의 layer를 합쳐서 separable conv layer로 사용한다.

### 4.1. Depth-wise(Channel-wise) Convolution Layer
<center><img src='{{ "/assets/images/depth-conv.png" | relative_url }}' width="300" height="300"></center>
기본 Conv layer에서 각 Channel별로 Kernel을 따로 둔다고 생각할 수 있다. 이렇게하면 각 Channel 별로 독립적인 Spatial Feature 추출이 가능해진다. Grouped Conv layer에서 g값이 Input Channel수와 같다고 보면 된다.
Group의 수가 Input channel과 같고, 특성상 Input과 Output의 Channel수가 같으므로 연산량이 상당히 줄어든다.

Computational Cost
Depthwise Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white K^2 CHW"/>

#### 4.2. Point-wise Convolution Layer
<center><img src='{{ "/assets/images/point-conv.png" | relative_url }}' width="450" height="300"></center>
1x1 Kernel을 이용한 굉장히 Tricky한 방법이다. Channel에 대해서만 다루는 특이한 Conv layer인데, 이런 특성을 이용하여 Channel 크기를 줄였다 늘였다하여 dimension 조절을 담당한다. 주로 channel 크기를 일시적으로 줄여서 dimension reduction 효과를 주는데 사용한다.

Computational Cost
Pointwise Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white CMHW"/>

> Separable Conv Layer는 Depthwise 다음에 Pointwise를 연계하여 Spatial과 Channel 모두 아울러 기존 Conv layer와 유사한 동작을 수행한다. 기존 Conv layer보다 월등히 적은 parameter수를 갖기 때문에 Network를 깊게 만들 때 필수적인 Layer이다.

Separable Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white CHW(K^2 + M)"/>

---
### 5. Transposed Convolution Layer
<center><img src='{{ "/assets/images/transpose-conv.gif" | relative_url }}' width="300" height="300"></center>
Deconvolution, Upconvolution, Fractionally-stride convolution 등 다양한 방식으로 불린다. 주어진 입력에 대해 압축된 정보를 추출하는 것이 아닌, 압축된 정보(sparse)에서 더 자세하고 촘촘한 정보(dense)로 복원(?)하는 개념이다.
왜 Transposed인지는 Reference 3번의 내용을 살펴보는 것이 빠를 것 같다. (저것보다 잘 설명하기 어려울 것 같음...) 이어서 4번 ref도 참조하면 좋다.(Transposed conv layer에서 나타날 수 있는 checkboard 현상을 극복하는 법에 대해 알려준다.)

---
### 6. 3D Convolution Layer
기존 2D standard conv layer에서 Volume 부분이 추가된 Layer이다. CT나 MRI와 같이 여러 장의 이미지가 하나의 Volume으로 되있는 데이터를 처리 할 때나, 비디오와 같이 시계열 이미지 데이터를 처리할 때 사용된다.

Computational Cost
- W : Input Width
- H : Input Height
- C : Input Channel
- K : Kernel Size
- M : Output Channel
- T : 데이터의 Volume(Time or # of images 등)
3D Convolutional Layer : <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white K^2 CMHWT"/>

---
### 7. 1D Convolution Layer
주로 시계열 데이터를 처리할 때 사용되며, Biomedical 데이터 또는 다양한 시계열 데이터에서 RNN류와 더불어 사용된다.

---
Reference
1. [https://zzsza.github.io/data/2018/02/23/introduction-convolution/](https://zzsza.github.io/data/2018/02/23/introduction-convolution/)
2. [https://hichoe95.tistory.com/48](https://hichoe95.tistory.com/48)
3. [https://medium.com/activating-robotic-minds/up-sampling-with-transposed-convolution-9ae4f2df52d0](https://medium.com/activating-robotic-minds/up-sampling-with-transposed-convolution-9ae4f2df52d0)
4. [https://distill.pub/2016/deconv-checkerboard/](https://distill.pub/2016/deconv-checkerboard/)
5. [https://datascience.stackexchange.com/questions/6107/what-are-deconvolutional-layers](https://datascience.stackexchange.com/questions/6107/what-are-deconvolutional-layers)
6. [https://towardsdatascience.com/understanding-1d-and-3d-convolution-neural-network-keras-9d8f76e29610](https://towardsdatascience.com/understanding-1d-and-3d-convolution-neural-network-keras-9d8f76e29610)