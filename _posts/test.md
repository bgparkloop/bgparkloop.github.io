---
layout: post
title: "[Study] Deep Learning: Image Augmentation"
categories: [deep learning]
tags: [cnn, image, augmentation]
fullview: true
comments: true
---
### Image Augmentation을 하는 이유
Deep learning model은 학습을 필요로하는 parameter가 너무 많기 때문에 그만큼 많은 데이터를 필요로 한다. (물론 데이터를 적게해서 학습하는 방법들도 없지는 않지만 많으면 더 성능이 좋다. 그만큼 질 좋은 데이터는 구하기 힘듦) 가장 학습을 잘되게 하는 방법은 풀고자 하는 문제에 대해 모든 상황을 갖는 데이터로 학습을 하는 것이지만! 그런식으로 되어있는 데이터는 없다.

많은 딥러닝 모델 중 CNN의 경우 Convolution 연산이라는 것을 통해 데이터의 특징을 잡아낸다. Convolution의 연산 특성상 이미지에서 추출하고자 하는 특징(Object)의 위치, 회전정도, 왜곡정도 등 많은 변수에 따라 그 값이 달라지기때문에 우리는 Image augmentation이라는 방법을 통해 이런 변화를 인위적으로 데이터셋에 적용시켜준다. (이를 "해당 변화(문제)에 대해 Invariant 또는 Robust하게 만든다"라고 표현할 수도 있을 것 같다.)

실제로 학습 시 우리가 찾고자하는 Object가 정중앙에 있을 때와 오른쪽으로 조금 이동된 이미지가 있을 때, 우리가 정중앙에 있는 데이터로만 학습했다면 오른쪽으로 이동된 이미지에서는 찾는 Object를 못 찾을 수도 있다. 하지만, Translation이라는 Augmentation을 주어 정중앙에 있는 Object를 오른쪽으로 이동한 이미지를 만들어내어 학습에 사용한다면 우리는 이런 이미지에서도 찾고자하는 Object를 잘 찾을 것이다.

우리가 학습하고자하는 데이터들은 종류가 다양하기 때문에 각각의 데이터가 수많은 변화를 갖는다. 이런 변화에 대해 모델이 적응하게 도우려면 그에 맞는 적절한 Augmentation기법이 필요하다.

---
### Image Augmentation 종류
#### 1. Translation (Shift)
의미 그대로 이미지 내에서 pixel을 shift한다. 또는 crop시에 특정 pixel만큼 shift하여 crop한다. 이렇게 하는 이유는 pixel이 1칸 이동되어 달라진 이미지라도 convolution하여 나온 feature map은 다르기 때문이다.

#### 2. Flip
Horizontal Flip과 Vertical Flip 2종류가 있다. Horizontal의 경우 거울로 보는 것과 같이 상이 좌/우로 뒤바뀌게 하면되며, Vertical은 상/하로 뒤집으면 된다. 보통 랜덤 확률을 추출하여 0.5를 기준으로 Flip할지 안할지를 결정한다.

#### 3. Rotation
Translation과 마찬가지로 Rotation도 중요한 문제 중 하나이다. 사람의 눈에야 회전하든 이동이 되어있든 같게 보이지만 컴퓨터는 해당 위치의 데이터를 보기 때문에 회전 정보를 추가로 넣어주어야 잘 인식한다. 회전 각도는 풀고자하는 문제에 따라 0~180도까지 다양하다.

#### 4. Zoom (Scale)
Zoom In/Out 둘다 필요한 문제이다. 학습에 사용된 데이터셋에서의 object 크기와 실제 적용하려는 Real data에서의 크기는 다를 수 있기 때문이다. 색상, 빛, 회전, 이동 등 나머지가 같더라도 크기가 달라지게 되면 인식하기 어렵다.

#### 5. Brightness / Contrast
영상에서 밝기와 대조 또한 굉장히 중요한 issue중 하나이다. 예를 들어 X-ray사진과 같이 grayscale이미지의 경우 Brightness와 Contrast의 차이는 Color 이미지보다 극명하게 드러난다. 이에 대해 학습을 도와주면 학습 성능은 크게 차이날 것이다.

#### 6. Normalization
보통 Color 이미지에서 많이 사용되며, 일반적으로 데이터셋 전체에 대해 채널 별 평균/표준편차 값을 구하여 Input 이미지에 대해 계산된 평균값을 빼고 표준편차로 나누어 normalization을 수행한다.

#### 7. CLAHE
Histogram equalization 방법 중 하나로 기존의 HE와 달리 전체 영역을 grid하게 나누어 각 영역 내에서 HE를 수행하여 Adaptive하게 적용한다. Chest-Xray와 같은 의료영상에 사용이 많이 된다.

#### 8. Mixup / Cutout / CutMix
<center><img src='{{ "/assets/images/study/image-augmentation01.png" | relative_url }}' width="450" height="250"></center>
Naver에서 발표한 augmentation들로 기존의 이미지 자체를 변형한 방법들과 차별성이 있다.
Mixup은 서로 다른 Label의 이미지를 alpha값을 조절하여 합성 후 label도 그에 맞게 1에서 해당 값으로 수정한다.
Cutout은 Label은 1개를 그대로 사용하지만, 이미지의 일부를 cut하여 제거한다.
CutMix는 Mixup과 Cutout을 합성한 것으로 Cutout하여 제거한 부분에 다른 Label 이미지를 끼워넣는다. 당연히 Label도 이미지 비율만큼 나눠진다.
굉장히 특이한 방식의 augmentation인데 이 중 CutMix가 가장 좋은 성능을 보인다. 하지만, Classification이 아닌 Segmentation에서 이와 같은 Augmentation을 진행할 경우 오히려 성능이 떨어진다. (생각에는 prediction된 pixel의 정확도가 보장되지 못하기 때문인 것 같다.)

Paper : [link](https://www.researchgate.net/publication/333078138_CutMix_Regularization_Strategy_to_Train_Strong_Classifiers_with_Localizable_Features)
<br><br>
이 외에도 많은 방법들이 존재하고, 또 데이터마다 특색있는 Augmentation을 customization할 수 있을 것이다. Augmentation은 Deep learning학습에 있어 빠질 수 없는 부분이고 data별로 맞춤형이 필요하기 때문에 다양한 augmentation 기법을 알아놓고 그에 맞게 사용하는 연습이 필요할 것 같다.

#### 9. Elastic Distortion
기존의 Rotation, Shearing, Translation 등의 Affine 변형에 Probabilistic spin을 주어 떨림을 준 것과 같이 자연스러운 이미지 변형을 주는 방법이다. 일반적으로 Biomedical data에 많이 사용된다. Alpha와 Sigma라는 두 개의 파라미터를 사용하는데 일반적으로 Alpha와 Sigma를 10:1로 조합하면 자연스럽게 떨리는 이미지를 생성할 수 있다.

---
Python 패키지 중에 imgaug라는 멋진 패키지가 있는데, 이를 이용하면 왠만한 기본 image augmentation은 전부 사용 가능하다.
친절하게 예시도 주어져 있으며, 한글 블로그 등에도 사용법들이 많이 나와 있으므로 잘써주도록 하자.

Ref : [https://imgaug.readthedocs.io/en/latest/](https://imgaug.readthedocs.io/en/latest/)
