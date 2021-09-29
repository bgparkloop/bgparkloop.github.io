---
title: "[Review] (2020) Proxy Anchor Loss for Deep Metric Learning"
categories:
  - Metric Learning
tags:
  - Deep Learning
  - Metric Learning
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---


# 1. Introduction

![image](/assets/imgs/paper/2020-proxy-anchor/00.png)

- 2가지 형태의 Metric Learning이 존재
    - Pair-based
        - Data-to-data의 관계를 학습 (pair 간 distance 계산)
        - 정밀하고 semantic 정보도 학습이 가능하지만, complexity가 높아 학습이 느리고 잘 안됨
        - Complexity를 줄이기 위해 Sampling 기법들이 많이 나왔지만 잘못하면 Overfitting 가능성 높임
        - Generalization이 잘됨
    - Proxy-based
        - Proxy라는 data 군의 대표와 data들 간의 관계를 학습하는 방법
        - Anchor data와 같은 class의 proxy간의 distance 계산
        - Global structure를 학습하기 때문에 detail이 떨어짐
        - 연산량이 적고 학습이 빠르고, label noise와 outlier에 강함
- 제안하는 방법은 Proxy + Pair의 장점을 합친 방법
    - Proxy-NCA 기반에 Pair-based를 합쳐 수렴이 빠르고 연산량도 적음(complexity 낮음)

# 2. Methods

![image](/assets/imgs/paper/2020-proxy-anchor/01.png)

- 원본 Proxy-NCA loss 구조
    - $l(X) = \sum_{x\in X} {-\log { \frac{ e^{s(x,p^+)} }{ \sum_{p^- \in P^-} e^{s(x,p^+) } } } }$
    - $=\sum_{x\in X} \{-{s(x,p^+)}{ + \text{LSE}_{p^- \in P^-}s(x,p^-) } \}$
- Proxy-NCA의 단점을 극복하기 위해 gradient를 pair-based로 계산
    - $l(X) =  \frac{1}{|P^+|} \sum_{p\in P^+} [ {\text{Softplus}(\text{LSE}_{x \in X_p^+}  \alpha(s(x,p^+)-\delta))} ]$
    - $=\frac{1}{|P|} \sum_{p\in P} [ {\text{Softplus}(\text{LSE}_{x \in X_p^-}  \alpha(s(x,p^+)+\delta))} ]$
        - $\alpha$ : margin scaling
        - $\delta$ : margin 값
        - $\text{Softplus}=\log(1+e^z)$
- 미분하면,
    - $\frac{\partial l(X)}{\partial s(x,p)} = \begin{cases} \frac{1}{|P^+|} \frac{-\alpha h_p^+(x)}{1+\sum_{x' \in X_p^+} h_p^+ (x')},& \forall x \in X_p^+ \\ \frac{1}{|P|} \frac{\alpha h_p^-(x)}{1+\sum_{x' \in X_p^-} h_p^- (x')},& \forall x \in X_p^- \end{cases}$
        - $h_p^+(x)=e^{-\alpha (s(x,p)-\delta)}$
        - $h_p^-(x)=e^{\alpha (s(x,p)+\delta)}$
    - sample $x$가 어려울 수록 gradient는 커짐
- Proxy-NCA는 gradient가 positive, negative에 대해 constant하지만, 제안하는 방법에서는 iteration마다 gradient를 새로 계산하기 때문에 generalizability가 높아짐
- Proxy-NCA처럼 batch 내에서 proxy와 나머지 sample간의 계산이므로 complexity도 낮음

![image](/assets/imgs/paper/2020-proxy-anchor/02.png)

# 3. Experimental Results & Conclusion

![image](/assets/imgs/paper/2020-proxy-anchor/03.png)

- 기존 방법들과 CUB-200-2011, Cars-196 dataset에 대해 Recall@K 비교 결과
    - 64, 128, 512는 output embedding vector 크기

![image](/assets/imgs/paper/2020-proxy-anchor/04.png)

- 제안하는 모델은 주어진 Query 이미지에 대해서도 global적인 정보 (자세, 시점 등)과 local한 정보(피부색, 옷 패턴, 털 색 등??)를 골고루 아울러 검색결과를 보여준다.

### 3.1. Impact of Batch Size

![image](/assets/imgs/paper/2020-proxy-anchor/05.png)

- Batch size가 커질수록 한 번에 고려할 수 있는 data-to-data, data-to-proxy 관계가 많아져 좀 더 정확한 모델 학습이 가능해짐

### 3.2. Impact of Embedding Dimension

![image](/assets/imgs/paper/2020-proxy-anchor/06.png)

- embedding dimension이 커질수록 Recall@1의 성능이 높아짐
- 하지만, 일정 이상(512~1024)부터는 큰 차이를 보이지 않음
    - embedding vector로 정보를 압축하는 과정에서 일정 이상의 크기로는 중복된 정보들이 들어오는게 아닌가 싶음

### 3.3. $\alpha$ and $\delta$ of our loss

![image](/assets/imgs/paper/2020-proxy-anchor/07.png)

- Margin과 scale factor의 상관관계 및 결과 성능
- scale factor의 경우는 높을수록 (32 이상)이 대게 좋은 것을 확인하였고, margin도 높을수록 좋긴 하지만 scale factor가 일정이상 올라가면 오히려 낮은것이 좋다
    - 아마 scale factor가 margin의 크기도 scale up해주기 때문에 적정선의 margin정도가 효율이 좋은 것 같다

# References

- [https://arxiv.org/abs/2003.13911](https://arxiv.org/abs/2003.13911)