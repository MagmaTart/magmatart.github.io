---
layout: article
title:  "[논문] Sampling Techniques for Large-Scale Object Detection from Sparsely Annotated Objects"
date:   2019-06-08 12:00:00 -0400
modify_date: 2019-06-08 12:00:00 -0400
tags:
- Deep Learning
- Paper Reading
- Object Detection
category: 
- deep learning
use_math: true
---

:memo: __논문 리딩 스터디 #21__: 데이터셋의 불완전성을 데이터셋 내부에서 직접 해결하는 Part-aware Sampling을 고안한 논문을 읽고 정리해봅니다.

<!--more-->
-----
논문 링크: [Sampling Techniques for Large-Scale Object Detection from Sparsely Annotated Objects](https://arxiv.org/pdf/1811.10862.pdf)

## Abstract

Object Detector의 수요가 늘어남에 따라 규모가 거대한 Detection Dataset의 수요도 점점 커지고 있습니다. 논문은 그 예로 Open Images Dataset v4(OID)를 소개하는데, 이 데이터셋의 가장 큰 문제는 __Sparsely Annotated__ Dataset이라는 것입니다. 이러한 데이터셋의 불완전성을 해결하기 위해, Pretrain된 Detector 모델을 사용하여 이미지 상에서 Annotation이 없는 Object를 찾아내는 방법을 생각할 수 있습니다. 그러나 이 방법은 Pretrained Model의 성능에 크게 영향을 받을 수밖에 없습니다. 이러한 문제를 해결하기 위해, 이 논문에서는 __Part-aware Sampling__ 을 고안합니다. 이 방법은 물체 사이의 어떠한 구조적인 관계를 파악하기 위한 인간의 직관을 사용합니다. 예를 들어, 논문에서 제시한 모델은 *'자동차의 Bounding box 안에는 타이어의 Bounding Box가 포함되어야 한다'* 와 같은 가정을 바탕으로 작동합니다. 그리고 논문에서 제시한 방법이 기존의 Pretrained Model을 사용한 Sampling보다 더 좋은 성능을 보여준다는 것을 확인합니다.

## Introduction

Object Detection이 주요한 AI 기술로써 큰 관심을 받는 추세에 따라, Detector Model을 트레이닝하기 위한 거대한 데이터셋의 수요도 점점 증가하고 있습니다. 논문에서 주목하는 데이터셋은 __Open Images Dataset v4 (OID)__ 로, 160만장의 이미지에 1400만개의 Object가 500개의 카테고리로 나뉘어져 레이블링되어 있습니다. 이 거대한 데이터셋의 각 이미지는 평균 7개씩의 Object가 포함되어 있습니다. 하지만 __인간이 직접 Annotation__ 을 수행한 점이, 데이터셋의 완전성에 대한 의문을 일으킬 수밖에 없습니다. 확인된 __Annotation Recall은 43%__ 인데, 이는 즉 평균적으로 하나의 이미지에서 실제로 존재하지만 Annotation이 없는 물체가 반이 넘는다는 것입니다. 이러한 상황을 나타내는 용어를 두 가지 정의하고 가겠습니다.

- Verified Object : 이미지 상의 Object들 중, 해당하는 GT(_Ground-truth_) Annotation이 있어 이미지 내 존재가 확인된 물체
- Unverified Object : Verified Object와 반대로, 이미지 상에 존재하지만 그것에 해당하는 GT Annotation이 없는 물체

Unverified Object가 많으면 Training 과정에서 당연히 문제가 될 수밖에 없습니다. Unverified Object들에 대한 Detection 결과가 모두 False Positive로 학습에 반영될 것이기 때문입니다. 이러한 문제를 피할 Naive한 방법 중 하나는, Objective Function을 계산하여 모델의 Loss를 구할 때, 검출된 Box 주변에 GT Annotation이 없을 경우 해당 Detection을 Objective Function 계산에 반영하지 않는 것입니다. 그러나 이 방법은 Unverified Object에 대한 검출 능력에 큰 영향을 받습니다. 따라서 Pretrain된 Detector 모델을 추가로 사용하여 Unverified Object가 있음을 알려주는 용도로 사용하는 방법을 생각할 수 있으나(Oracle), 이 또한 Pretrained Model의 성능에 따라 결과가 제한적일 수 밖에 없습니다.

논문은 인간의 직관에서 아이디어를 얻었습니다. 만약 이미지에 자동차가 있다면, 타이어는 높은 확률로 자동차의 Bounding box 안에 존재할 것입니다. 마찬가지로 이미지에 문이 있다면, 문 손잡이가 문의 Box 안에 존재할 확률이 높습니다. 여기서 중요한 포인트를 발견할 수 있습니다. 만약 자동차에 대한 Annotation이 있는데 그 Bounding box 안에 타이어에 대한 Annotation이 없다면, 그것이 우리가 문제삼아야 할 _Dangerous Case_ 일 것입니다. 즉 타이어는 이미지 안에 존재하지만 Annotate되지 않은 물체이며, Detector Model의 학습 도중에 이 타이어를 Detection한 결과에 대한 Loss는 반영하지 말아야 할 것입니다. 또한, 이미지 안에 자동차에 대한 Annotation이 있는 경우에는 당연히 타이어가 이미지 안에 존재할것으로 예상할 수 있습니다.

이것이 바로 논문에서 제시하는 __Part-aware Sampling__ 이 동작하는 원리입니다. 이미지 안의 모든 Verified Object와 그것들의 Bounding box 안에 포함되는 모든 Sub-bounding box들을 고려하여, Detector가 그러한 Sub-bounding box들을 검출하지 않도록 만드는 것입니다. 즉, 검출된 Unverified Object 중 Verified Object 안에 포함되는 것들을 무시하는 것입니다.

논문은 이 Approach의 성능 향상을 확인하기 위해, Pretrained Model을 사용하는 기존 Approach의 동작 방식을 다시 정리하고, __Pseudo label-guided Sampling__ 으로 이름붙였습니다. Pretrained Model을 사용하여 이미지 내의 Unverified Object를 찾아내 *pseudo-label*을 줘서, 해당 Pseudo-label과 겹치는 Detection Box에 대해서 그 카테고리에 해당하는 Loss를 반영하지 않는 방식입니다.

논문에서 디자인한 Part-aware Sampling은, Pseudo-label guided Sampling에 비해 학습되는 Detector에게 더 높은 AP를 가져다주었습니다. 특히 위에서 Sub-bounding box로 표현한 *Part Object* 에 대해 특히 큰 AP의 향상을 보였습니다.

## Methods

논문은 기준 Architecture를 Two-stage Detector, 그 중에서도 Faster R-CNN으로 설정하였습니다. 이 구조의 모델은 __Region Proposal Network__ 를 통해 물체의 대략적인 위치를 찾은 후, Main Network에서 물체를 Background 또는 카테고리 중 하나로 분류하고 더 정확히 Localize합니다. 학습 도중에는 각 Proposal마다 하나의 예측 카테고리가 분류되어 배정되며, 이는 Classification Loss의 계산에 사용됩니다.

### Basic Architecture

논문의 주요 Approach는, 논문에서 제시하는 Sampling Method를 사용하여, 학습 도중 각각의 Proposal들에 대해서 Classification Loss 계산 시에 무시할 카테고리를 골라내는 것입니다. 그리고 실제 계산 시에는 그 카테고리를 무시하도록 만들어, 모델이 Unverified Object로 인한 영향을 최대한 적게 받을 수 있도록 만드는 것이 목표입니다. 이를 위해서는 모든 Proposal과 모든 카테고리에 대해 각각의 Loss가 모두 있어야 합니다. 그래야 선택적으로 Loss를 무시할 수 있기 때문이죠. 하지만 기존의 Classification Loss는 _Softmax Loss_ 로, 각각의 Loss를 선택적으로 무시할 수 없는 구조였습니다. 따라서 논문은 __Sigmoid Cross Entropy들의 총합__ 으로 Classification Loss를 다시 정의합니다.

$$\mathbf{L}_\text{cls} = -\sum_i \sum_c l_{ic} \log p_{ic}$$

$$l_{ic} = 1$$일 경우 $$i$$번째 Proposal에 Category $$c$$가 배정되었다는 것이고, $$l_{ic} = -1$$일 경우 그렇지 않다는 것입니다. 또한 $$l_{ic} = 0$$일 경우, $$i$$번째 Proposal의 Category $$c$$에 대한 Loss는 무시하겠다는 이야기입니다. 각 Proposal마다 무시할 카테고리를 정하는 것은 차후 Unverified Object들 때문에 발생하는 잘못된 트레이닝의 방지에 매우 중요한 역할을 하게 됩니다.

### Part-aware Sampling

본격적으로 Approach를 설명하기 전, 논문은 제기한 문제가 실제로 많은 경향을 보이는지 확인합니다. 먼저 두 가지 용어를 정의합니다. 

- Subject category : 여러 Part category들을 포함할 수 있는 메인 카테고리.
- Part category : Subject category의 Object가 일반적으로 포함하거나, 종속 관계에 있는 Object의 카테고리.

예를 들어 '인간'이 Subject category라면, Part category는 '인간의 눈', '인간의 코', '인간의 손' 등이 될 것입니다. 마찬가지로 '자동차'의 Part category는 '타이어', '번호판' 등이 있습니다. 논문의 저자들은 대부분의 경우 Subject object에 비해 Part object들에 대한 Annotation이 매우 적다는 사실을 관찰했습니다.

이러한 경향의 확인 작업을 위해, 물체의 Box가 다른 물체의 Box 안에 포함되어 있는지를 정의하는 Metric인 __AIOU__(Asymmetric Intersection Over Union)를 정의합니다. 두 박스 $$b_1, b_2$$에 대한 AIOU는 $$\text{aiou}(b_1, b_2) = \frac{area(b1 \  \cap \ b_2)}{area(b_1)}$$ 으로 정의됩니다. 그리고 AIOU 점수가 특정 Threshold $$\tau$$를 넘을 경우 Part object가 Subject object에 포함되었다고 판단합니다. 이를 이용해, 전체 데이터셋 내에서 특정 Part category의 물체들이 그를 주로 포함할 것으로 예상되는 Subject category에 얼마나 포함되어 있는지 비율을 계산합니다(_Included_). 그와 동시에 Subject와 Part의 물체가 동시에 둘 다 Labeling되어 있는 경우의 비율도 계산합니다(_Co-occur_). 결과는 아래의 표와 같았습니다.

<p align="center"><image src="/assets/posts/images/SamplingTech/figure1.png"/></p>

_Included_ 줄을 보면, 전체 데이터셋에서 대부분의 Part category가 그에 해당하는 Subject category의 물체 안에 90% 이상 포함되어 있는 모습을 보입니다. 이를 통해 Part와 Subject는 서로 Spatial한 관계에 놓여 있음을 알 수 있습니다. 또한 _Co-occur_ 줄을 보면, 실제로 Part와 그에 해당하는 Subject가 이미지 내에서 같이 Labeling된 비율은 극히 적은 것을 확인할 수 있었습니다. 예를 들어 인간과 인간의 눈이 같이 레이블링된 비율은 2.8%에 불과합니다. 예상했던 문제가 크게 나타나는 것을 확인할 수 있었습니다.

__Part-aware Sampling__ 은 특정 Proposal에서 Classification loss를 계산할 때 무시할 Category를 선택하는 방법을 제공합니다. 이 기법의 중심 아이디어는, Proposal의 물체가 특정 Subject object의 Part일 확률이 높다고 판단될 떄, 그 Proposal에서는 해당 Subject의 Part category들에 대한 Classification Loss를 무시하는 게 안전하다는 것입니다. 이 기법은 해당하는 Part category가 이미지 상에서 Unverified일 때만 사용됩니다.

이 기법의 원리는 간단합니다. 현재 이미지 안의 모든 Annotation의 Category label들을 모은 집합을 만들수 있으며, 이를 Verified label이라고 하겠습니다. 또한 모든 카테고리에 대해, 그것이 보통 포함할 수 있는 물체들의 카테고리인 Sub-category들을 모은 세트를 정의합니다(Part와 동치). 이제 어떤 Proposal이 특정 GT Box 내에 포함되어 있다고 가정합니다. 그러면 그 Proposal이 가리키는 Object의 카테고리는 자신을 포함시키는 GT의 Sub-category일 가능성이 클 것입니다. GT의 모든 Sub-category들 중, 현재 이미지에서 Verify되지 않은 카테고리를 골라냅니다. 만약 현재 Proposal이, 골라낸 Category중 하나의 클래스로 Classify된다면, GT 중에는 그 물체에 대한 Annotation이 없으므로 False Alarm 처리되어 학습에 악영향을 미칠것입니다. 따라서 해당 Proposal이 골라낸 Category들에 대한 Classification Loss를 반영하지 않도록 만드는 것입니다. 그렇게 하면 Unverified object가 학습에 미치는 악영향을 막을 수 있습니다.

<p align="center"><image width="400" src="/assets/posts/images/SamplingTech/figure2.png"/></p>

<p align="center"><image src="/assets/posts/images/SamplingTech/figure3.png"/></p>

Part-aware Sampling의 예를 보여주는 그림입니다. (a)는 이미지 상의 GT를 표시한 그림으로, 자동차와 사람에 대한 GT가 표시되어 있습니다. (b)는 Detector가 만들어낸 Proposal들입니다. (c)는 자동차와 사람의 GT 안에 포함된 Proposal들을 각각 그린 그림입니다. Part-aware Sampling을 거치면, 사람의 GT 안에 포함된 보라색 Proposal들에 대해서는 사람의 Sub-category인 '신발', '사람 얼굴' 등과의 Loss가 무시될 것입니다. 또한 자동차의 GT 안에 포함된 초록색 Proposal들에 대해서는 '번호판', '타이어' 등이 무시됩니다.

### Pseudo label-guided Sampling

Part-aware Sampling과의 성능 비교를 위한 기존 Approach인 Pseudo label-guided Sampling도 소개합니다. 

미리 현재 Dataset에서 학습된 Pretrained Detector를 준비합니다. 트레이닝 이미지에 Pretrained Detector를 한번 돌리고, Detection들을 추출합니다. 그리고 추출한 Detection 중 GT Box와 겹치지 않는 Detection을 이미지 상에 __Pseudo label로 임시 레이블링__ 합니다. 예로, 아래의 이미지에서 빨간 박스는 GT Label이고, 초록 박스는 Pseudo label입니다.

<p align="center"><image width="500" src="/assets/posts/images/SamplingTech/figure4.png"/></p>

이제 학습중인 Detector가 이미지의 Proposal들을 생성했을 때, Pseudo label과 IOU가 일정 이상인 Proposal이 있는지 확인합니다. 그리고 Pseudo label과 겹치는 Proposal에 대해서, 해당 Pseudo label의 카테고리에 대한 Loss를 무시하게 만드는 방식입니다.

<p align="center"><image width="400" src="/assets/posts/images/SamplingTech/figure5.png"/></p>

## Experiments

실험을 위해 두 가지의 데이터셋을 사용합니다. Sparse COCO라는 데이터셋을 새로 만드는데, 전체 MS COCO 데이터셋에서 각 카테고리의 Annotation을 랜덤하게 $$\alpha$$의 확률로 지운 데이터셋입니다. $$a = 0.7$$이라면 각 카테고리별로 70%의 Annotation을 지웠다는 뜻입니다. 나머지 하나는 논문의 Target이었던 OID v4입니다.

주목해야 할 실험 결과는 OID를 이용한 Sampling 방법 별 mAP 비교입니다. OID의 Validation Set에서, 기존 학습과 제안된 Sampling 기법을 사용한 학습의 mAP 차이를 비교해보았더니, 확실히 성능의 향상을 볼 수 있었습니다.

<p align="center"><image width="350" src="/assets/posts/images/SamplingTech/figure6.png"/></p>

동시에 관찰된 또 하나의 놀라운 결과가 있습니다. 아래에 제시된 표와 그래프에 나타나는 바와 같이, 주로 Part에 속하는 카테고리의 물체들에 대한 mAP가 놀라운 수준으로 향상된 것입니다.

<p align="center"><image src="/assets/posts/images/SamplingTech/figure7.png"/></p>

<p align="center"><image src="/assets/posts/images/SamplingTech/figure8.png"/></p>

이러한 결과는, Part-aware Sampling이 주로 Unverified Object의 영향을 많이 받는 Part category의 물체들에 대한 검출 성능을 확실히 향상시킨다는 사실을 알려줍니다.