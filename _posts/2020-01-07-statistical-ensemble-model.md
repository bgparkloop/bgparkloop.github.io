---
layout: post
title: "[Paper Review] Statistical Ensemble Model"
categories: [deep learning, ensemble]
tags: [cnn, ensemble]
fullview: true
comments: true
---


## Improving Automated Pediatric Bone Age Estimation Using Ensembles of Models from the 2017 RSNA Machine Learning Challenge

Ensemble model을 선택해야되는 일이 있어 참고용 논문을 읽은 적이 있어 내용을 정리해본다.
이 논문에서는 통계적인 방법을 통해 여러개의 모델들을 어떻게 앙상블해야 좋은 결과를 낼 수 있을지 분석한다.

---
### 1. 앙상블 모델 선택 방법
총 2가지 방법을 사용하여 진행하는데, 하나는 Combination을 통해 가진 모델 중 일부를 선택하여 평균을 내는 방법, 다른 방법은 가진 모델들 중 Top N개의 모델을 선택하는 방법이다. 이 논문에서는 이미 학습된 모델 48개를 Testset 200장에 대해 앙상블 모델을 선택하는 법을 제안한다.

- AMC(All-Model-Combination) Method: 전체 모델 수에서 1부터 N개까지 차례대로 모든 경우의 수만큼 모델 조합을 추출하여 validation set으로 검증하여 앙상블 모델을 선택하고 testset으로 검증한다. 논문에서는 48개 중 1에서 10개까지의 조합만 검증하였다.

- Top N Method: AMC와 다르게 전체 모델에 대해 validation set으로 error를 계산하고 이 중 error가 가장 작은 N개를 추출하여 앙상블 모델로 사용한다.

---
### 2. 통계 분석
논문에서 앙상블 모델의 조합은 매우 간단하게하고 이 중에서 best 앙상블 모델을 선택하는데 통계적인 기법을 사용한다. 1번에서 설명한 것과 같은 방법으로 앙상블 모델을 선택하는데 이를 모든 경우에 대해 1000번씩 테스트한다. 아래에 나오는 Metric들을 통해 다각도에서 분석 후 최종적으로 모델을 선택한다.

- MAD(Mean Absolute Deviation): 평균 절대 편차. 다음과 같은 수식으로 정의된다. <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{150}\bg_white MAD = \frac{1}{n}\sum_{i=1}^n|x_i-m(x)|"/>   
- MAD(Mean Absolute Difference)
- Pearson correlation
- 95% Confidence interval
- RMSD(Root Mean Square Deviation)
- RMM
- 

---
