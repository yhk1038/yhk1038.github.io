---
layout: post
title: "Pytorch를 활용한 RNN(Recurrent Neural Network)"
subtitle: "Pytorch를 활용한 RNN"
categories: data
tags: dl
comments: true
---
[김성동](https://github.com/dsksd)님의 Pytorch를 활용한 딥러닝 입문 중 RNN 파트 정리입니다.

## Language Modeling

```
철수와 영희는 식탁에 앉아 사과를 __(A)__
```

- (A)에 들어올 단어는? **먹었다!**

- 이런 아이디어 기반해서 만들어진 것이 Language Model

$$P(x^{(t+1)}=w_j|x^{(t)}, ...,x^{(1)})$$

- 키보드의 자동 완성 기능, 서치 엔진에서 쿼리 자동 완성 기능 모두 Language Model을 적용한 Application으로 볼 수 있음
- 음성 인식을 할 때도 Language Model이 쓰임! 해당 단어만 잘못 들었을 경우(noise가 껴있다거나) 이전까지 인지한 단어를 기반으로 단어를 추론!
- N-gram으로 모델링을 합니다. gram은 gramma의 줄임말

```
철수와 영희는 식탁에 앉아 사과를 ______
```

- Unigram : 토큰 하나가 변수가 됨 : "철수", "와", "영희", "는", "식탁", "에"  
- Bigram : 두개의 토큰이 하나의 변수가 됨 : "철수 와", "와 영희", "영희 는", "는 식탁", "식탁 에", "에 앉아"  
- Trigram : 3개의 토큰이 하나의 변수 : "철수 와 영희", "와 영희 는", "영희 는 식탁", "는 식탁 에", "식탁 에 앉아"  
- 4-gram : 4개의 토큰 : "철수 와 영희 는", "와 영희 는 식탁", "영희 는 식탁 에", "는 식탁 에 앉아"  
- N-gram : N개의 토큰이 하나의 변수가 됨

#### 가정 : t+1의 확률은 이전 n-1개의 단어(토큰)에만 의존한다


<img src="../assets/img/rnn1.png">


### 문제점 
- 앞의 정보를 무시하고 있음. 가정 자체에 한계 존재
- n-1 이전의 맥락을 모델링할 수 없음
- 해당 n-gram이 Corpus에 없거나 희소한 경우 확률이 0이나 매우 낮게 나올 수도 있음
- n이 커질수록 더욱 확률은 희박해짐
- Corpus에 있는 n-gram을 모두 카운트해서 저장해야 하기 때문에 모델의 공간 복잡도가 O(exp(n))
- **위 문제점 때문에 Neural Language Model로 접근하기 시작함**

### Window-based Language Model
- 고정된 Window size(~n-1)를 인풋으로 받는
FFN(Feed Forward Neural Network)
- 카운트할 필요가 없기 때문에 Sparsitiy 문제 없음
- 모델의 사이즈도 작음
- **여전히 고정된 window size에 의존하기 때문에 Long-term Context를 포착하지 못함**
- **토큰의 윈도우 내의 위치에 따라 다른 파라미터를 적용 받음(Parameter sharing이 없음)**

### Recurrent Neural Network
- 모든 Timestamp에서 같은 Parameter를 공유!
- Input의 길이가 가변적입니다


<img src="../assets/img/rnn2.png">

<img src="../assets/img/rnn3.png">

- time t의 hidden state는 이전 모든 time step x를 인풋으로 받는 함수 g의 아웃풋으로 볼 수 있습니다(모두 연결되어 있으니까-!)

#### Notation
<img src="../assets/img/rnn4.png">
<img src="../assets/img/rnn5.png">

- 인풋의 차원에 대한 감이 있어야 합니다!
- x는 word vector
- 모든 Time Step에서 Parameter를 Sharing!

[RNN 참고 자료](https://ratsgo.github.io/natural%20language%20processing/2017/03/09/rnnlstm/)


#### 예시
```
뭐 먹 을까?
```

D=3로 총 4개의 Timestamp가 있어서 4x3 매트릭스를 indexing했습니다

| 0.1 	| 0.1 	| 0.2 	|
|-----	|-----	|-----	|
| 0.5 	| 0.2 	| 0.3 	|
| 1.0 	| 0.0 	| 0.2 	|
| 0.1 	| 0.1 	| 0.  	|

첫 행이 $h^{(0)}$! 

<img src="../assets/img/rnn6.png">

- 마지막 step의 Hidden state는 "뭐 먹을까?"라는 문장을 인코딩한 벡터로 볼 수 있습니다


```
input_size = 10 # input dimension (word embedding) D
hidden_size = 30 # hidden dimension H
batch_size = 3
length = 4

rnn = nn.RNN(input_size, hidden_size,num_layers=1,bias=True,nonlinearity='tanh', batch_first=True, dropout=0, bidirectional=False)

input = Variable(torch.randn(batch_size,length,input_size)) # B,T,D  <= batch_first
hidden = Variable(torch.zeros(1,batch_size,hidden_size)) # 1,B,H    (num_layers * num_directions, batch, hidden_size)

output, hiddne = rnn(input, hidden) 
output.size() # B, T, H   
hidden.size() # 1, B, H
# (배치 사이즈, 시퀀스 길이, input 차원)을 가지는 Input 
# (1,배치 사이즈, hidden 차원)을 가지는 초기 hidden state

```

```
나는 너 좋아
오늘 뭐 먹지
```

- Batch Size, 2
- Time : 문장의 길이, 3
- Dimension : 인풋의 차원, 10
- Style마다 먼저 쓰는 것이 다른데, B, T, D로 많이 사용하곤 함 (batch_first=Ture)
- hidden은 마지막 hidden state를 뜻합니다

<img src="../assets/img/rnn7.png">



#### 하이퍼 파라미터 세팅
<img src="../assets/img/rnn8.png">

- ?에 들어갈 것은 무엇일까요?  
$E$ : VxD  
$W_e$ : DxH  
$W_h$ : HxH  
$U$ : HxV  




