---
layout: post
title:  "딥러닝에서 사용되는 여러 유형의 Convolution 소개"
subtitle:   "딥러닝에서 사용되는 여러 유형의 Convolution 소개"
categories: data
tags: dl
comments: true
---
[An Introduction to different Types of Convolutions in Deep Learning](https://towardsdatascience.com/types-of-convolutions-in-deep-learning-717013397f4d)을 번역한 글입니다. 개인 공부를 위해 번역해봤으며 이상한 부분은 언제든 알려주세요 :)


Convolution의 여러 유형에 대해 빠르게 소개하며 각각의 장점을 알려드리겠습니다. 단순화를 위해서, 이 글에선 2D Convolution에만 초점을 맞추겠습니다


## Convolutions

우선 convolutional layer을 정의하기 위한 몇개의 파라미터를 알아야 합니다

<img src="https://cdn-images-1.medium.com/max/1200/1*1okwhewf5KCtIPaFib4XaA.gif">
<p align="center">2D convolution using a kernel size of 3, stride of 1 and padding</p>



- **Kenel Size** : kernel size는 convolution의 시야(view)를 결정합니다. 보통 2D에서 3x3 pixel로 사용합니다
- **Stride** : stride는 이미지를 횡단할 때 커널의 스텝 사이즈를 결정합니다.  기본값은 1이지만 보통 Max Pooling과 비슷하게 이미지를 다운샘플링하기 위해 Stride를 2로 사용할 수 있습니다
- **Padding** : Padding은 샘플 테두리를 어떻게 조절할지를 결정합니다. 패딩된 Convolution은 input과 동일한 output 차원을 유지하는 반면, 패딩되지 않은 Convolution은 커널이 1보다 큰 경우 테두리의 일부를 잘라버릴 수 있습니다
- **Input & Output Channels** : convolution layer는 Input 채널의 특정 수(I)를 받아 output 채널의 특정 수(O)로 계산합니다. 이런 계층에서 필요한 파라미터의 수는 I\*O\*K로 계산할 수 있습니다. K는 커널의 수입니다


## Dilated Convolutions ( 확장된 Convolution )
(a.k.a. atrous convolutions)

<img src="https://cdn-images-1.medium.com/max/1200/1*SVkgHoFoiMZkjy54zM_SUw.gif">
<p align="center">
2D convolution using a 3 kernel with a dilation rate of 2 and no padding
</p>

Dialted Convolution은 convolutional layer에 또 다른 파라미터인 dilation rate를 도입했습니다. 이것은 커널 사이의 간격을 정의합니다. dilation rate가 2인 3x3 커널은 9개의 파라미터를 사용하면서 5x5 커널과 동일한 시야(view)를 가집니다.  

5x5 커널을 사용하고 두번째 열과 행을 모두 삭제한다고 가정해보겠습니다.  
이것은 동일한 계산 비용으로 더 넓은 시야를 제공합니다. Dialted convolution은 특히 real-time segmentation 분야에서 주로 사용됩니다. 넓은 시야가 필요하고 여러 convolution이나 큰 커널을 사용할 여유가 없는 경우 사용합니다


## Transposed Convolutions
(a.k.a. deconvolutions or fractionally strided convolutions)

어떤 곳에선 doconvolution이라는 이름을 사용합니다. 이는 deconvolution이 아니기 때문에 부적절합니다.  상황을 악화시키기 위해 deconvolution이 존재하지만, 일반적으로 딥러닝 분야에선 흔하지 않습니다.  실제 deconvolution은 convolution의 과정을 되돌립니다. 하나의 convolutional layer에 이미지를 입력한다고 상상해보겠습니다. 이제 출력물을 가져와 블랙 박스에 넣고 원본 이미지가 다시 나타납니다. 이 블랙박스가 Deconvolution을 수행합니다. 이것은 convolutional layer가 수행하는 것의 역 연산입니다.  

transposed(바뀐) convolution은  deconvolutional layer와 동일한 공간 해상도를 생성하기 때문에 유사합니다. 그러나 값에 대해 수행되는 실제 수학 연산은 다릅니다. transposed convolutional layer는 정기적인 convolution을 수행하지만 공간의 변화를 되돌립니다.


<img src="https://cdn-images-1.medium.com/max/1200/1*BMngs93_rm2_BpJFH2mS0Q.gif">
<p align="center">
2D convolution with no padding, stride of 2 and kernel of 3
</p>

이 부분은 매우 혼란스러울 수 있으므로 구체적인 예를 보겠습니다. convolution layer에 넣을 5x5 이미지가 있습니다. stride는 2, padding은 없고 kernel은 3x3입니다. 이 결과는 2x2 이미지가 생성됩니다.  

이 과정을 되돌리기 위해, 역 수학 연산을 위해 input의 각 픽셀으로부터 생성된 9개의 값이 필요합니다. 그 후에 우리는 stride가 2인 출력 이미지를 가로지릅니다. 이것은 deconvolution입니다

<img src="https://cdn-images-1.medium.com/max/1200/1*Lpn4nag_KRMfGkx1k6bV-g.gif">
<p align="center">
Transposed 2D convolution with no padding, stride of 2 and kernel of 3
</p>

transpose된 convolution은 그렇게 하지 않습니다. 공통점은 오직 정상적인 convolution 작업을 하면서 5x5 이미지의 output을 생성한다는 것입니다. 이를 위해 input에 상상의 padding을 수행해야 합니다.  

지금 상상할 수 있듯, 이 단계는 위의 과정을 반대로 수행하지 않습니다. 최소한 숫자값과 관련이 없습니다.  

단순히 이전 공간 해상도를 재구성하고 convolution을 수행합니다. 이것은 수학적 역 관계는 아니지만 인코더-디코더 아키텍쳐의 경우 유용합니다. 이 방법은 2개의 별도 프로세스를 진행하는 것 대신 convolution된 이미지의 upscaling을 결합할 수 있습니다



## Seperable Convolutions

separable convolution에선, 커널 작업을 여러 단계로 나눌 수 있습니다. convolution을 ```y = conv(x, k)```로 표현해봅시다. x는 입력 이미지, y는 출력 이미지, k는 커널입니다. 다음으로 k=k1.dot(k2)로 계산된다고 가정해보겠습니다. 이것은 K와 2D Convolution을 수행하는 대신 k1와 k2로 1D convolution하는 것과 동일한 결과를 가져오기 때문에 separable convolution이 됩니다.  

<img src="https://cdn-images-1.medium.com/max/1200/1*owXMr9DonUUWP1c2Thg_Dw.png">
<p align="center">
Sobel X and Y filters
</p>

예를 들어 이미지 처리에서 자주 사용되는 Sobel 커널을 예로 들겠습니다. 벡터 [1, 0, -1]과 [1, 2, 1].T를 곱하면 동일한 커널을 얻을 수 있습니다. 동일 작업을 하기 위해 9개의 파라미터 대신 6개가 필요합니다. 위 예는 spatical separable convolution의 예입니다. 심층적 학습엔 사용되지 않습니다
> 수정 : 사실 1xN, Nx1 커널 레이어를 쌓아 Seperable convolution과 유사한 것을 만들 수 있습니다. 이것은 최근 유망한 결과를 보여준 EffNet라는 아키텍쳐에서 사용되었습니다.

신경망에서 우린 일반적으로 depthwise separable convolution이라고 불리는 것을 사용합니다. 이렇게 하면 채널을 분리하지 않고 spatial convolution을 수행하고 depthwise convolution을 수행합니다. 이것은 예를 통해 가장 잘 이해할 수 있습니다.


16개의 input 채널과 32개의 output 채널에 3x3 convolutional 레이어가 있다고 가정하겠습니다. 자세히 살펴보면 16개의 채널마다 32개의 3x3 커널이 가로지르며 512(16*32)개의 feature map이 생성됩니다. 그 다음, 모든 입력 채널에서 1개의 feature map을 병합하여 추가합니다. 우리는 32번 반복해 원하는 32개의 output 채널을 얻습니다

같은 예제에서 depthwise separable convolution을 위해 우리는 1개의 3x 커널로 16 채널을 탐색해 16개의 feature map을 생성합니다. 이제 어떤 것을 합치기 전에 32개의 1x1 convolution으로 16개의 featuremap을 지나가며 함게 추가합니다. 결과적으로 위에선 4068(16\*32\*3\*3) 매개 변수를 얻는 반면 656(16\*3\*3 + 16\*32\*1\*1) 매개변수를 얻습니다

이 예는 depthwise separable convolution(depth multiplier가 1이라고 불리는)것을 구체적으로 구현한 것입니다. 이것은 이런 layer에서 가장 일반적입니다

우리는 spatial하고 depthwise한 정보를 나눌 수 있다는 가정하에 이 작업을 합니다. Xception 모델의 성능을 보면 이 이론이 효과가 있는 것으로 보입니다. Depthwise seprable convolution은 매개변수를 효율적으로 사용하기 때문에 모바일 장치에도 사용됩니다

# Questions?
이것으로 여러 종류의 convolution 여행을 끝내겠습니다. 이 문제에 대한 짧은 요약을 가지고가길 바라며 남아있는 질문은 댓글을 남겨주시고, 더 많은 convolution 애니메이션이 있는 [Github](https://github.com/vdumoulin/conv_arithmetic)를 확인해주세요