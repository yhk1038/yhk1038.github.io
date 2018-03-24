---
layout: post
title: "Pytorch를 활용한 Advanced Sequence models"
subtitle: "Pytorch를 활용한 Advanced Sequence models"
categories: data
tags: dl
comments: true
---
[김성동](https://github.com/dsksd)님의 Pytorch를 활용한 딥러닝 입문 중 Advanced Sequence Model 파트 정리입니다.

## Machine Translation
### Statistical Machine Translation
- Source Language(X)를 토대로 Target Language(Y)로 번역하는 Task
- 예시 : 스페인어를 Input으로 넣고, Output을 영어!
#### 목적 함수

```$$argmax_yP(y|x)$$```  
source 문장이 주어졌을 떄, 가장 좋은 Target 문장을 찾는 것
$$argmax_yP(x|y)P(y)$$ 으로 식을 분해
$$P(x|y)$$는 Translation Model : 어떻게 번역하는지 학습  

Parallel Corpus가 필요 (병렬 코퍼스)  
$$P(y)$$는 Language Model : 어순 및 문법 보정

$$P(x|y)$$를 $$P(x,a|y)$$로 변경  
여기서 $$a$$는 Alignment로 Source language와 Target language의 **단어간 상호 관련성**을 의미  
one to one, one to many, many to many alignment가 필요


<img src="https://github.com/zzsza/zzsza.github.io/blob/master/assets/img/sequence1.png?raw=true">

- 실제로 이 확률을 계산하기 위해서는 모든 가능한 경우의 수를 다 대조해봐야 하기 때문에 너무 비싼 연산입니다. 실제론 너무 확률적으로 낮은 것들은 무시해버리고 Heuristic Search Algorithm 사용합니다


### Neural Machine Translation
#### Sequence to Sequence
- 번역에 Neural Network를 사용했습니다. 2014년부터 기존의 Machine Translation 성능을 엄청 끌어올렸습니다
- 특별히 Sequence-to-Sequence라는 새로운 형태의 뉴럴넷 구조를 사용하기 시작!

- Encoder와 Decoder 역할을 하는 2개의 RNN을 사용

<img src="https://github.com/zzsza/zzsza.github.io/blob/master/assets/img/sequence2.png?raw=true">

- Decoder는 Encoder의 Last Hidden state를 초기 Hidden state로 사용하여 Target 문장에 대한 
Language model을 학습
- \<START>, \<END> 토큰 : 문장의 시작과 끝을 알려주며, 디코더 입장에선 condition에 따라 디코딩을 시작하고, end 토큰이 나오면 디코딩을 멈춰라
- NMT는 확률 $P(y|x)$을 바로 계산합니다(Statisctical에선 2개로 나눠서 계산했음)
- Source 문장(x)과 이전(T-1)까지의 Target 문장이 주어졌을 때 step T에 $$y_T$$일 조건부 확률

#### 학습
- End to End Training
- Decoder의 각 time step에서 발생한 Loss(Cross-Entropy)를 사용하여 두 모델의 파라미터를 한번에 업데이트

