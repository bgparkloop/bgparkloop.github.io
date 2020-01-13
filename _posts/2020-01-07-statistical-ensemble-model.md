---
layout: post
title: "[Paper Review] Statistical Ensemble Model"
categories: [deep learning, ensemble, statistics]
tags: [cnn, ensemble, statistics]
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


#### 2.1. Metrics
- MAD(Mean Absolute Deviation): 평균 절대 편차. 다음과 같은 수식으로 정의된다.
<br><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white MAD = \frac{1}{n}\sum_{i=1}^n|x_i-m(x)|"/>   
- MAD(Mean Absolute Difference): 평균 절대 차. 다음과 같은 수식으로 정의된다.
<br><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white MAD = \frac{1}{n^2}\sum_{i=1}^n\sum_{j=1}^n|x_i-y_i|"/>   
- Pearson correlation: 두 변수 X와 Y간의 선형적인 상관관계를 수치적으로 표현할 때 많이 사용된다. 코시-슈바르츠 부등식에 의해 -1 ~ +1 사이의 범위를 갖으며, +1은 완벽한 상관관계, 0은 상관관계 없음, -1은 완벽한 음의 상관관계를 갖는다. 일반적으로 상관관계를 말할 때는 피어슨 상관관계를 의미한다.
<br><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white r_{XY} = \frac{\sum_i{n}(X_i-\bar{X})(Y_i-\bar{Y})}{\sqrt{\sum_i{n}{(X_i-\bar{X})}^2}\sqrt{\sum_i{n}{(Y_i-\bar{Y})}^2}}"/>   
- 95% Confidence interval: 신뢰구간은 추출한 표본이 모집단의 모수값일 가능성을 나타낸 값의 범위이다. 표본은 모집단 혹은 그 외에서 랜덤하게 추출되기 때문에 반복적인 추출작업을 통해 평균적으로 이 범위안에 들어갈 것이라고 예측하는 것이다. 95% 신뢰구간이라는 것은 어떤 값을 기준으로 5% 가량의 오차 한계를 두어 +5%, -5% 만큼은 허용하는 것을 의미한다. 90%, 80%와 같이 오차 한계가 커질 수록 신뢰성은 떨어지게 된다.
<br><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white 95_{CI} = \bar{X}\pm1.96\times\frac{s}{\sqrt{n}}"/>   
- RMSD(Root Mean Square Deviation): RMSE(Root-mean-square-error)라고 더 많이 쓰인다. 예측 값과 관찰 값(원본 값)과의 차이를 나타낼 때 많이 사용된다. RMSD는 평균 제곱 오차의 제곱근이기 때문에 큰 차이값이 날 수록 RMSD값이 급속도로 상승한다. 따라서, 특이값(Outlier)에 매우 민감하다.
<br><img style="vertical-align:middle" src="http://latex.codecogs.com/png.latex?\dpi{100}\bg_white RMSD=\sqrt{\frac{\sum_i{N}{(\hat{x_i}-x_i)}^2}{N}}"/>  

#### 2.2. Figure & Table Analysis
Figure와 Table 분석을 통해 앙상블 모델을 어떻게 선택하는지 설명하고 있다.
<br><br>
<center><img src='{{ "/assets/images/ensemble_01.PNG" | relative_url }}' width="700" height="600"></center>
- 처음 Figure는 RMM(Mean model correlation for a model)을 분석한 것으로써 선택된 모델이 다른 모델들과 어느정도 상관관계를 갖는가에 대해 나타낸 것이다. 계산은 pair를 이루게 2개의 모델을 선택하여(한 모델은 기준이 되는 모델 - 예를들어 1번 모델과 3번, 1번모델과 4번 등) 두 모델간 상관관계를 계산한다. 기준이 되는 모델(1번)과 그 외 나머지 모델과의 상관관계를 전부 계산하고 이를 평균을 취해 나타낸다. 상관관계가 낮을수록 파란색으로 채워져있고, 높을수록 붉은색으로 표현된다.
<br>
<center><img src='{{ "/assets/images/ensemble_00.PNG" | relative_url }}' width="500" height="350"></center>
- RMM은 Table1에서 보는 것처럼 모델의 성능(MAD Error)과 관련이 있는데, 성능이 좋을수록(MAD가 낮을수록) RMM도 동시에 낮아진다. 이를 통해 모델 간 상관관계가 적을 수록 좋은 성능을 지닌 모델로 볼 수 있다.
<br>
<center><img src='{{ "/assets/images/ensemble_02.PNG" | relative_url }}' width="700" height="600"></center>
- Figure4도 Figure3과 마찬가지로 모델의 성능을 표현한다. Fig.4에서는 MAD 기준으로 표현하였으며, Fig.3과 마찬가지로 4번과 16번이 가장 성능이 좋음을 알 수 있다.
<br>
<center><img src='{{ "/assets/images/ensemble_03.PNG" | relative_url }}' width="400" height="500"></center>
- Table 3, 4는 계속해서 MAD를 기준으로 모델의 성능평가를 보여준다. Table 3에서는 AMC와 Top-N 모두에 대해 성능평가를 보여준다. 전체적으로 AMC가 성능이 좋음을 확인 할 수 있다. Tabel 4는 아예 AMC에 대해서만 나타낸다. 1~10개를 선택하여 나타낸 것으로 Best Ensemble 모델 조합을 표현했고 괄호안에 1000번 중 얼만큼 비율로 해당 모델 조합이 나왔는지 표기하였다. Runner-up은 2번째로 성능이 좋은 조합이다.
<br>
<center><img src='{{ "/assets/images/ensemble_04.PNG" | relative_url }}' width="700" height="600"></center>
- Figure 5는 이 논문의 핵심이라고 볼 수 있다. 이 Matrix를 통해 Optimal Ensemble 모델을 선택한다.(Optimal이 맞다고 할 수 있을련지..?) M1, A2, ..., A10, T2, ..., T10이 나와있는데 각각 M1(Single model), A2(AMC 2), T2(Top-2)로 표현된다. Matrix 안의 숫자는 X축의 모델이 Y축의 모델보다 좋은 성능을 낸 비율을 나타낸다. 만약 X축의 A4에 Y축의 T2라면, A4모델이 T2모델보다 1000번 중 97%만큼 좋은 성능을 보였다는 것을 나타낸다. X축의 모델 밑에 확률이 또 있는데 이 확률은 해당 모델이 1000번 중 가장 낮은 Error값을 보인 비율이다. 이를 토대로 A4모델의 경우 1000번 중 24%만큼 최소 error를 보였기 때문에 논문에서는 이 모델을 가장 좋은 앙상블 모델로 선정했다.

---
### 3. 결론
이 논문에서 제안하는 앙상블 모델 선택 방법은 굉장히 일반적인 선택 방법들 중 하나이다. 다만, 통계적으로 접근하여 해당 조합을 선택했다는 점에서 무작정 선택하는 조합보다 설득력이 있다고 본다. 이 논문에서는 MAD를 기준으로 모델 선택을 하였지만, Classification과 같은 다른 Task에서는 AUC나 F1 score 등 다양한 Metric을 통해 같은 방식으로 선택해 볼 수 있을 것 같다.