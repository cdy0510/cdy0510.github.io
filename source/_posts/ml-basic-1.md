---
title: "[ML]수포자가 이해한 머신러닝 - 2.기본용어정리"
categories: MachineLearning
date: 2018-08-20 14:40:38
tags:
    - MachineLearning
    - study
    - GoogleStudyJams
---
수포자가 이해한 머신러닝 시리즈는 구글의 머신러닝 문서 순서를 따릅니다.
[머신러닝 단기집중과정]

[머신러닝 단기집중과정]: https://developers.google.com/machine-learning/crash-course/ "머신러닝 단기집중과정"
<br/>

### 라벨(Label)
예측하는 항목, 단순 선형 회귀의 `y` 변수
ex) 이 메일은 스팸일까?아닐까?
<br/>

### 특성(Feature)
입력 변수, 단순 선형 회귀의 `x` 변수
특성은 여러개가 될 수 있습니다. 
{% img center-img /images/feature.PNG 여러개 특성 %}

스팸인지 아닌지를 판별할 때 메일에는 다음과 같은 특성을 가질 수 있습니다.
- 보낸 사람
- 메일이 전송된 시간
- 특정 단어가 포함된 메일

<br/>

### 예(Example)
예는 두개로 분류될 수 있습니다.

- 라벨이 있는 예(labeled examples) : {features, label} : (x, y) -> 학습시킬 때 사용
- 라벨이 없는 예(unlabeled examples) : {features, ?} : (x, ?) -> 라벨 예측이 필요함

<br/>

### 모델(Model)
특성과 라벨의 관계를 정의합니다. 예를들어 스팸 감지 모델에서 특정 특성을 '스팸'과 긴밀하게 연결할 수 있습니다.
- 모델 수명의 두단계
    - 학습(Training): 모델을 만들거나 배우는 것
    - 추론(Inference): 학습된 모델을 라벨이 없는 예에 적용하는 것

<br/>

### 회귀와 분류(Regression vs. classification)
기존의 데이터를 가지고 새로운 데이터를 예측합니다.
- 회귀(Regression): 연속적인 값 예측    ex) 내가 산 땅 값은 10년 뒤 얼마가 될까? 
- 분류(Classification): 불연속적인 값 예측   ex) 이 메일은 스팸일까?아닐까?