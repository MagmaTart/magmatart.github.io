---
layout: article
title:  "선형 보간법, 쌍선형 보간법 (Linear Interpolation, Bilinear Interpolation)"
date:   2017-11-19 12:00:00 -0400
modify_date: 2017-11-19 12:00:00 -0400
tags:
- Machine Learning
- Deep Learning
- Math
category: 
- Development
use_math: true
---

선형 보간법(Linear Interpolation) 에 대해 간단히 정리합니다.

<!--more-->
-----
보간법이란 어떤 데이터에 나타나있지 않은 부분을 그 데이터들을 이용해 추정하는 방법을 말합니다. 컴퓨터 비전을 비롯해 수학과 CS의 다양한 분야에서 활용되고 있습니다.

### 선형 보간법
__선형 보간법(Linear Interpolation)__ 은 수직선 상에서 두 점 사이의 임의의 점의 함숫값을 구하는 방법입니다. 수식적으로는 아래와 같이 정의됩니다. 수직선 상의 두 지점 $$x_1, \ x_2$$에 대한 함숫값이 $$f(x_1), \ f(x_2)$$일 때, $$x_1$$과 $$x_2$$ 사이에 위치하는 임의의 점 $$x$$에 대한 함숫값 $$f(x)$$는 다음과 같이 구할 수 있습니다.

$$f(x) = \frac{\vert x - x_2 \vert}{\vert x - x_1 \vert + \vert x - x_2 \vert} f(x_1) \ + \ \frac{\vert x - x_1 \vert}{\vert x - x_1 \vert + \vert x - x_2 \vert} f(x_2)$$

실제 데이터를 임의로 만들어서 테스트해보겠습니다.

수직선 상에서 함숫값 $$f(1) = 3, \ f(9) = 27$$일 때, $$f(6)$$의 값을 선형 보간법을 이용해서 구해 봅시다. 주어진 값들을 식에 그대로 대입하면 $$f(6)$$의 값을 구할 수 있습니다. 수직선 상의 점의 위치와 대응하는 함숫값만 알아도 가능합니다.

$$f(6) = \frac{\vert 6 - 9 \vert}{\vert 6 - 1 \vert + \vert 6 - 9 \vert} f(1) \ + \ \frac{\vert 6 - 1 \vert}{\vert 6 - 1 \vert + \vert 6 - 9 \vert} f(9)$$

$$= \frac{3 \times 3}{8} + \frac{5 \times 27}{8} = 18$$

정확히 값을 알아내는 것을 볼 수 있습니다. 이 함수는 $$f(x) = 3x$$ 입니다. 생각해보면 그냥 수직선 상의 내분점을 구하는 작업과 동일합니다.

### 쌍선형 보간법
__쌍선형 보간법(Bilinear Interpolation)__ 은 2차원 데이터에 적용하는 방법으로, 1차원 선상의 선형 보간법을 기본으로 합니다.

<p align="center"><image width="350" src="/assets/posts/images/31/figure1.png"/></p>

점 $$\text{A, B, C, D}$$ 사이의 쌍선형 보간 $$\text{R}$$을 구하는 과정은 다음과 같습니다.
1. 점 $$\text{A}$$와 $$\text{B}$$ 사이의 선형 보간 $$\text{m}$$을 구한다.
2. 점 $$\text{C}$$와 $$\text{D}$$ 사이의 선형 보간 $$\text{n}$$을 구한다.
3. $$\text{m}$$과 $$\text{n}$$ 사이의 선형 보간 $$\text{R}$$을 구한다.

방향을 90도 돌려도 결과는 같습니다.
1. 점 $$\text{A}$$와 $$\text{D}$$ 사이의 선형 보간 $$\text{p}$$를 구한다.
2. 점 $$\text{B}$$와 $$\text{C}$$ 사이의 선형 보간 $$\text{q}$$를 구한다.
3. $$\text{p}$$와 $$\text{q}$$ 사이의 선형 보간 $$\text{R}$$을 구한다.

실제로 선형 보간법은 임의 위치의 데이터를 추정할 때 많이 쓰입니다. 제가 쌍선형 보간이 사용되는 모습을 보았던 첫 논문이 [FCN](https://magmatart.dev/deep%20learning/2017/11/17/FCN.html) 이었습니다.
