---
layout: post
title: "Pytorch를 활용한 자연어 처리(NLP)"
subtitle: "Pytorch를 활용한 자연어 처리(NLP)"
categories: data
tags: dl
comments: true
---
Pytorch를 활용한 자연어 처리 내용입니다.

# Distributed Word Representation

NLP Task는 지금까지 봤던 접근법이랑(CNN류) 많이 다릅니다. RNN을 배우기 전에 Word Vector를 알아야 하기 때문에 이 내용을 추가했습니다  

```
파이토치가 짱이야. 
아닌데? 텐서플로우가 짱이지 구글이 만들었잖아
그래도 파이토치는 쉬워 페북도 장난 아니야
아마존이 만든 맥스넷이란 것도 좋더라
```
다음 대화에서 어떤 정보를 얻을 수 있을까요?

```
파이토치(페북) ≈ 텐서플로우(구글) ≈ 맥스넷(아마존)
```

사람은 문맥을 통해 각 단어의 유사함과 관계를 쉽게 추론할 수 있음! 컴퓨터에게 넣기 위해선 컴퓨터가 이해할 수 있도록 변형해줘야 합니다. 이 과정을 Word Representation이라고 합니다

## Word Representation
Idea/Thing을 여러 Symbol로 표현할 수 있습니다.(같은 것을 다르게 표현할 수 있습니다) 여기서 각 단어간의 관계도 알 수 있습니다(유사한지)  

<img src="../assets/img/nlp-1.png">

### WordNet 

<img src="../assets/img/nlp-2.png">

- 단어 의미를 그래프 형태로 출력
- 사람이 직접 구축해야 함(비싼 비용)
- 신조어의 의미를 이해 못함
- 뉘앙스를 놓치기 쉬움
- 주관적임
- 단어 간의 정확한 유사도 계산이 어려움(얼마나 유사한지) 그래프상 몇 노드가 연결되는지 정도만 알 수 있음


### One-hot Vector

```
파이토치	 [1,0,0,0,0,0,0,0]
짱		  [0,1,0,0,0,0,0,0]
텐서플로우	[0,0,1,0,0,0,0,0]
구글		 [0,0,0,1,0,0,0,0]
쉽다		 [0,0,0,0,1,0,0,0]
페북		 [0,0,0,0,0,1,0,0]
맥스넷		 [0,0,0,0,0,0,1,0]
아마존		 [0,0,0,0,0,0,0,1]
```

- 단어 하나를 하나의 Discrete Variable로 취급
- 각 단어의 구분은 가능하자 유사도를 측정할 수는 없음
- 두 벡터를 내적해서 유사도를 측정!
- 만약 1만개의 단어를 One-hot vector로 표현하면?
	- 1만 차원의 벡터가 필요해 차원의 저주가 발생함

### Distrivuted word representation
- 그렇다면, 단어의 의미를 분산시켜 벡터로 표현하자!
- 그 단어의 속성은 함꼐 쓰이는 단어(맥락)에 의해 결정될 것이다
- Dense한 Vector를 만들어봄!

```
파이토치	 [0.6, -0.2, 0.7, 0.3, 0.7, -0.2, 0.1, 0.1]
텐서플로우	[0.4, -0.1, 0.6, -0.2, 0.6, -0.2, 0.3, 0.4]
고양이	     [-0.3, 0.2, 0.1, 0.2, -0.2, 0.1, -0.3, 0.1]

유사도
파이토치^T * 텐서플로우 = 1.15
파이토치^T * 고양이 = -0.26
```

- 비교적 낮은 차원의(50 ~ 1000차원) 벡터에 단어의 의미를 분산해 Dense한 벡터로 표현
- Word vector라고 부름


## NLP Task
[NLP Task](https://github.com/DSKSD/Pytorch_Fast_Campus_2018/blob/master/week6/1_Bag_of_Words.ipynb) 연습

Bag of Words : Count 방식으로 표현

### 1. Tokenize
- 문장을 단어 또는 형태소 단위로 토큰화
	- 형태소 : 언어를 이루는 최소 단위
- 문장이란 것은 단어의 연속이기 때문에 리스트로 표현할 수 있음
- 토큰은 문장, 단어, character, 형태소가 될 수 있음
	- 한국어는 형태소로 쪼개는 방법에 유효함
	
```
token = nltk.word_tokenize("Hi, my name is sungdong. What's your name?")
print(token)
>>> ['Hi', ',', 'my', 'name', 'is', 'sungdong', '.', 'What', "'s", 'your', 'name', '?']
```

```
# 꼬꼬마 형태소 분석기
token = kor_tagger.morphs("안녕하세요! 저는 파이토치를 공부하는 중입니다.")
print(token)
>>> ['안녕', '하', '세요', '!', '저', '는', '파이', '토치', '를', '공부', '하', '는', '중', '이', 'ㅂ니다', '.']
```

### 2. Build Vocab
- 단어의 인덱스를 가지고 있어야 함(각 자리에 맞게 넣어주기 위해)
- 따라서 Vocab을 구축
	- 각 단어의 ID를 붙여주는 작업
	
```
word2index={} # dictionary for indexing
for vo in token:
    if word2index.get(vo)==None:
        word2index[vo]=len(word2index)
print(word2index)
>>> {'하': 1, '.': 13, '를': 8, '중': 10, '안녕': 0, '파이': 6, '세요': 2, '토치': 7, '저': 4, '!': 3, '공부': 9, 'ㅂ니다': 12, '이': 11, '는': 5}
``` 

### 3. One-hot Encoding
- 자신의 인덱스에는 1을 채우고, 나머지엔 0을 채움

```
def one_hot_encoding(word,word2index):
    tensor = torch.zeros(len(word2index))
    index = word2index[word]
    tensor[index]=1.
    return tensor
```

```
torch_vector = one_hot_encoding("토치",word2index)
print(torch_vector)
>>> 
 0
 0
 0
 0
 0
 0
 0
 1
 0
 0
 0
 0
 0
 0
```

- 유사도를 계산해보면, (One Hot Encoding의 한계)

```
py_vector = one_hot_encoding("파이",word2index)
py_vector.dot(torch_vector)
>>> 0.0
```

### Bag of Words를 통한 분류
```
train_data = [["배고프다 밥줘","FOOD"],
                    ["뭐 먹을만한거 없냐","FOOD"],
                    ["맛집 추천","FOOD"],
                    ["이 근처 맛있는 음식점 좀","FOOD"],
                    ["밥줘","FOOD"],
                    ["뭐 먹지?","FOOD"],
                    ["삼겹살 먹고싶어","FOOD"],
                    ["영화 보고싶다","MEDIA"],
                    ["요즘 볼만한거 있어?","MEDIA"],
                    ["영화나 예능 추천","MEDIA"],
                    ["재밌는 드라마 보여줘","MEDIA"],
                    ["신과 함께 줄거리 좀 알려줘","MEDIA"],
                    ["고등랩퍼 다시보기 좀","MEDIA"],
                    ["재밌는 영상 하이라이트만 보여줘","MEDIA"]]

test_data = [["쭈꾸미 맛집 좀 찾아줘","FOOD"],
                   ["매콤한 떡볶이 먹고싶다","FOOD"],
                   ["강남 씨지비 조조 영화 스케줄표 좀","MEDIA"],
                   ["효리네 민박 보고싶엉","MEDIA"]]


# 0. Preprocessing
train_X,train_y = list(zip(*train_data))

# 1. Tokenize
train_X = [kor_tagger.morphs(x) for x in train_X]

# 2. Build Vocab
word2index={'<unk>' : 0}
for x in train_X:
    for token in x:
        if word2index.get(token)==None:
            word2index[token]=len(word2index)
            
class2index = {'FOOD' : 0, 'MEDIA' : 1}
print(word2index)
print(class2index)

# 3. Prepare Tensor
def make_BoW(seq,word2index):
    tensor = torch.zeros(len(word2index))
    for w in seq:
        index = word2index.get(w)
        if index!=None:
            tensor[index]+=1.
        else:
            index = word2index['<unk>']
            tensor[index]+=1.
    
    return tensor
    
train_X = torch.cat([Variable(make_BoW(x,word2index)).view(1,-1) for x in train_X])
train_y = torch.cat([Variable(torch.LongTensor([class2index[y]])) for y in train_y])

# 4. Modeling
class BoWClassifier(nn.Module):
    def __init__(self,vocab_size,output_size):
        super(BoWClassifier,self).__init__()
        
        self.linear = nn.Linear(vocab_size,output_size)
    
    def forward(self,inputs):
        return self.linear(inputs)
        
# 5. Train
STEP = 100
LR = 0.1
model = BoWClassifier(len(word2index),2)
loss_function = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(),lr=LR)

for step in range(STEP):
    model.zero_grad()
    preds = model(train_X)
    loss = loss_function(preds,train_y)
    if step % 10 == 0:
        print(loss.data[0])
    loss.backward()
    optimizer.step()
    
# 6. Test
index2class = {v:k for k,v in class2index.items()}

for test in test_data:
    X = kor_tagger.morphs(test[0])
    X = Variable(make_BoW(X,word2index)).view(1,-1)
    
    pred = model(X)
    pred = pred.max(1)[1].data[0]
    print("Input : %s" % test[0])
    print("Prediction : %s" % index2class[pred])
    print("Truth : %s" % test[1])
    print("\n")           
```

- \<unk>란 단어는 모르는 단어를 처리하기 위해 넣은 단어(인덱스가 존재하지 않으면 unk!)
- Long Tensor로 변환


## Word2Vec
- 우린 엄청 많은 Text를 가지고 있음(인터넷상에 텍스트는 많음)
- 모든 단어는 fixed vocabulary (Train에 정의한 단어가 fixed voca가 됨)
- 단어의 속성은 주변 단어로부터 결정된다라는 전제가 있었는데, 주변 단어가 Input이고 중심 단어가 Output이 나오는 CBOW 모델과 중심 단어가 Input이고 주변 단어가 Output인 Skip-gram이 있습니다 

### Skip-gram
<img src="../assets/img/nlp-3.png">

- 중심 단어가 있으면 주변 단어가 나올 조건부 확률을 구할 수 있음
- 윈도우 사이즈는 하이퍼 파라미터

### Learnable Embedding Matrix
- 7개의 단어를 5차원의 Vector로 임베딩하고 싶은 경우엔 7*5의 Embedding Matrix가 필요함
- 1\*7와 7\*5를 행렬곱하면 자신의 Index로 인덱싱
- Embedding Matrix를 학습하는 것이 Word2Vec

```
# Pytorch
embed = nn.Embedding(총 단어의 갯수, 임베딩 시킬 벡터의 차원)
embed.weight
>>> Parameter Containing : 학습 가능
```

- Embedding 모듈은 index를 표현하는 LongTensor를 인풋으로 기대하고 해당 벡터로 인덱싱합니다
- 따라서 원핫벡터로 명시적으로 바꿔주지 않아도 됩니다


### Object Function
<img src="../assets/img/nlp-4.png">

- Corpus : 텍스트의 뭉치
- 각 토큰마다 그 단어가 중심단어가 될 수 있음
- 중심 단어가 등장했을 때, 맥락 단어가 함께 등장할 확률을최대화하는 방향으로 Parameter를 업데이트 
	- 최적화를 쉽게하기 위해 Log로 바꿔서 곱을 합으로 변경
	- -를 붙여서 Log-Likelihood를 **최소화** 

<img src="../assets/img/nlp-5.png">

- 각 단어는 Center Word와 Context word가 될 수 있음
<img src="../assets/img/nlp-6.png">

$u_o^T*v_c$ : 내적을 해서 유사도를 구함

### 예시
```
I have a puppy . His name is Bori . I love him .
```

```
Corpus : 텍스트의 뭉치
T : 14 (코퍼스 내의 단어의 갯수)
m : 2 (Window size, 모델러가 정할 하이퍼 파라미터)
V : 11 (코퍼스 내의 단어의 집합. 중복을 제거한 집합)
```

1. Corpus에서 단어 집합**(Vocabulary)**을 구해서 **Index**를 매긴다.
2. **Window size**를 정하고 **T개의 데이터셋**를 준비한다.
3. **Center word**와 **Context word**를 표현할 **2개의 Embedding Matrix**를 선언한다.
4. **P(o|c)**를 구해서 **Negative log-likelihood(loss)**를 구한다.
5. **Gradient Descent**를 사용하여 **loss**를 최소화한다.
6. 학습이 끝난 뒤에는 **Center vector와 Context vector를 평균**해서 사용한다.


