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

- MAD(Mean Absolute Deviation): 평균 절대 편차. 다음과 같은 수식으로 정의된다.  <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white MAD = \frac{1}{n}\sum_{i=1}^n|x_i-m(x)|"/>   
- MAD(Mean Absolute Difference): 평균 절대 차. 다음과 같은 수식으로 정의된다.  <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white MAD = \frac{1}{n^2}\sum_{i=1}^n\sum_{j=1}^n|x_i-y_i|"/>   
- Pearson correlation: 두 변수 X와 Y간의 선형적인 상관관계를 수치적으로 표현할 때 많이 사용된다. 코시-슈바르츠 부등식에 의해 -1 ~ +1 사이의 범위를 갖으며, +1은 완벽한 상관관계, 0은 상관관계 없음, -1은 완벽한 음의 상관관계를 갖는다. 일반적으로 상관관계를 말할 때는 피어슨 상관관계를 의미한다.  <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white r_{XY} = \frac{\sum_i{n}(X_i-\bar{X})(Y_i-\bar{Y})}{\sqrt{\sum_i{n}{(X_i-\bar{X})}^2}\sqrt{\sum_i{n}{(Y_i-\bar{Y})}^2}}"/>   
- 95% Confidence interval: 신뢰구간은 추출한 표본이 모집단의 모수값일 가능성을 나타낸 값의 범위이다. 표본은 모집단 혹은 그 외에서 랜덤하게 추출되기 때문에 반복적인 추출작업을 통해 평균적으로 이 범위안에 들어갈 것이라고 예측하는 것이다. 95% 신뢰구간이라는 것은 어떤 값을 기준으로 5% 가량의 오차 한계를 두어 +5%, -5% 만큼은 허용하는 것을 의미한다. 90%, 80%와 같이 오차 한계가 커질 수록 신뢰성은 떨어지게 된다.  <img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white 95percent Confidence Interval = \bar{X}\pm1.96\times\frac{s}{\sqrt{n}}"/>   
- RMSD(Root Mean Square Deviation): RMSE(Root-mean-square-error)라고 더 많이 쓰인다. 
---
