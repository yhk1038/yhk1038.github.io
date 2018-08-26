---
layout: post
title: "Kaggle 강의 Week3 - Metrics optimization"
subtitle: "Coursera How to win a data science competition 3주차 Metrics optimization"
categories: data
tags: kaggle
comments: true
---

Coursera 강의인 How to Win a Data Science Competition: Learn from Top Kaggler Week3 Metrics optimization를 듣고 정리한 내용입니다  

---

## Metrics optimization
---

- 왜 Metrics은 많은가?
- Competition에서 왜 Metrics에 관심을 가져야 하는가?
- Loss vs metric
- 가장 중요한 metrics
	- 분류와 회귀 Task
	- Regression
		- MSE, RMSE, R-squared
		- MAE
		- (R)MSPE, MAPE
		- (R)MSLE
	- Classification
		- Accuracy, Logloss, AUC
		- Cohoen's (Quadratic weighted) Kappa  
	- metric의 최적화 베이스라인
- metrics의 최적화 테크닉

### Metrics
- 각각의 대회가 metric이 다른 이유는 대회를 주최한 회사가 그들의 특정 문제에 가장 적절한 것을 결정하기 때문
	- 또한 문제마다 다른 metrics 
- Online shop은 effectiveness를 최대화하는 것이 목표
	- Effectiveness를 정의
	- 웹사이트에 방문한 횟수, 주문수로 나눌 수 있음
	- 이런 것을 주최자들이 결정
- 더 나은 점수를 받기 위해 metric을 잘 이해하는 것은 필수

### Regression metrics review 1
- MSE
	- Mean Sqaured Error 
	- $$\frac{1}{N}\sum_{i=1}^{N}(y_{i}-\hat{y_{i}})^{2}$$ 
	- MSE는 우리의 예측값의 평균 squared error를 측정
	- <img src="https://www.dropbox.com/s/thuqgk8f9w84hok/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2011.09.25.png?raw=1">
	- Target value의 mean
	- [참고 자료](http://nbviewer.jupyter.org/gist/DmitryUlyanov/685689cdad7ab2da635605af22cbca6f)
- RMSE
	- Root mean square error
	- $$\sqrt{\frac{1}{N}\sum_{i=1}^{N}(y_{i}-\hat{y_{i}})^{2}}$$ 
- MAE
	- Mean Absolute Error
	- $$\frac{1}{N}\sum_{i=1}^{N}\left|y_{i}-\hat{y_{i}}\right|$$ 
	- <img src="https://www.dropbox.com/s/sfhjp9skzswwqv3/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2011.35.46.png?raw=1">
	- MAE는 finance에서 널리 사용됨
	- Target value의 median
	- Outliear의 영향을 받지 않음(outliear에게 robust)
- MAE vs MSE
	- MAE
		- 아웃라이어가 있는 경우
		- 아웃라이어라고 확신하는 경우
	- MSE
		- 우리가 신경써야 하는 관측하지 못한 값이라고 생각할 경우
- MSE, RMSE, R-squared
	- 최적화 관점에서 같음
- MAE
	- 아웃라이어에 robust    	 

### Regression metrics review 2
- Shop에서 매출을 예측하는 문제
	- Shop1 : predicted 9, sold 10, MSE = 1
	- Shop2 : predicted 999, sold 1000, MSE = 1 
	- Shop1이 더 cirtical한데, MAE와 MSE는 동일함
	- 그래프도 모든 Target value마다 동일(곡선이거나 V이거나)
	- 위 문제점을 해결하기 위해 나온 것이 MSPE, MAPE 
- <img src="https://www.dropbox.com/s/8x4os3ulr5adcyo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2012.24.43.png?raw=1">
- relative error의 합에, 100%/N을 곱함
- MSE와 MAE의 weight 버전이라고 생각할 수도 있음
- MSPE
	- <img src="https://www.dropbox.com/s/utr29ssmbup3gzu/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2012.39.53.png?raw=1"> 
- MAPE	
	- <img src="https://www.dropbox.com/s/pu1qb9omd1g4gu0/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2012.40.35.png?raw=1">
	- [참고 자료](http://nbviewer.jupyter.org/gist/DmitryUlyanov/358a36930e78c33449d0f8caaeee7026)
- RMSLE
	- Root Mean Squared Logarithmic Error
	- <img src="https://www.dropbox.com/s/qeem3qvzld0fuq7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2012.42.55.png?raw=1">
	- MSPE와 MAPE와 동일한 상황에 사용 (relative error)
	- prediction value와 target value가 큰 숫자일 때, 큰 차이를 벌점으로 내고싶지 않을 때 사용
	- 과소평가된 항목에 패널티
	- <img src="https://www.dropbox.com/s/frp4in4c8yxzuk3/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2012.52.47.png?raw=1">
- Conclusion
	- (R)MSPE
		- Weighted version of MSE
	- MAPE
		- Weighted version of MAE
	- (R)MSLE
		- MSE in log space    


### Classification metrics review
- Accuracy
	- 분류기의 성능을 측정할 때 가장 간단히 사용할 수 있음
	- Best constant : predict the most frequent class
	- 데이터셋의 라벨 A가 10%, B가 90%일 때 B로만 예측한 분류기는 90%의 accuracy를 가지는데, 이 분류기는 좋은 것일까?
	- optimize하기 어려움
- Logarithmic loss(logloss)
	- <img src="https://www.dropbox.com/s/69e3a9mn0fmtfpx/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2013.25.07.png?raw=1">
	- logloss가 잘못된 답변에 대해 더 강하게 패널티 부여
	- <img src="https://www.dropbox.com/s/hdbgrb2ouo3zsi8/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2013.51.07.png?raw=1">
- Area Under Curve (AUC ROC)
	- Binary task에서만 사용
	- 특정 threshold를 설정
	- 예측의 순서에 의존적이며 절대값엔 의존적이지 않음
	- 설명 가능
		- AUC
			- <img src="https://www.dropbox.com/s/oos0wxyv9zj4jo7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2014.05.11.png?raw=1">
			- <img src="https://www.dropbox.com/s/8s18234ymr6to3r/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2014.05.57.png?raw=1"> 
		- Pairs ordering
			- AUC = # correctly ordered paris / total number of paris = 1 - # incorrectly ordered pairs / total number of pairs
- Cohen's Kappa motivation
	- Baseline accuracy는 라벨의 개수가 제일 많은 것으로 설정
	- Accuracy = 0.9(baseline) -> my_score = 0
	- Accuracy = 1 -> my_score = 1
	- Mse에서 r squared의 역할과 유사
	- 일종의 normalization
	- <img src="https://www.dropbox.com/s/mmyd14yfv21ygb8/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2014.22.06.png?raw=1">
	- kappa는 baseline을 $$p_{e}$$로 지정(랜덤하게 계산)
	- Weighted error
		- <img src="https://www.dropbox.com/s/bvwqw7aty7bs1cp/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2014.30.20.png?raw=1"> 


### General approaches for metrics optimization
- Loss and metric
	- 동의어 : loss, cost, objective 
- **Target metrics** : 우리가 최적화 하고 싶은 것
	- 그러나 이걸 효율적으로 올리는 방법을 바로 알긴 어려움
- **Optimization loss** : 모델을 최적화 하는 것
	- Target metrics을 직접 최적화하기 어려울 때 사용
- Target metric 최적화 접근 방법
	- 일단 올바른 모델을 실행
		- MSE, Logloss
	- Train을 전처리하고 다른 metric을 최적화
		- MSPE, MAPE, RMSLE
	- 후처리로 예측된 metric 최적화
		- Accuracy, Kappa
	- Custom loss function 
		- 할 수 있으면 언제나!
	- Early stopping을 사용     

### Regression metrics optimization
- RMSE, MSE, R-squared
	- 대부분의 라이브러리에 loss function으로 구현되어 있음
	- L2 loss
	- <img src="https://www.dropbox.com/s/s2kpiwtr8uf840n/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2016.58.44.png?raw=1">
- MAE
	- L1 loss, Median regression
	- <img src="https://www.dropbox.com/s/xrz2yqlr05ahu20/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2017.00.00.png?raw=1">
- MSPE and MAPE
	- 둘 다 Weighted version이기 때문에 쉽게 구현 가능
	- <img src="https://www.dropbox.com/s/7ucho2aj4a04xle/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2017.04.30.png?raw=1">
- RMSLE
	- <img src="https://www.dropbox.com/s/vmnuxo6s8csfijr/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2017.04.59.png?raw=1">

### Classification metrics optimization 1
- Logloss
	- <img src="https://www.dropbox.com/s/9hszkvvytuzjhny/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2017.07.52.png?raw=1">
	- Probability calibration(눈금)
		- Platt scaling
			- 예측값에 Logistic Regression을 fit(마치 stacking처럼)
		- Isotonic regression
			- 예측값에 Isotonic Regression을 fit(마치 stacking처럼)
		- Stacking
			- 예측값에 XGBoost나 neuralnet을 fit  
- Accuracy
	- 이걸 직접적으로 optimize하는 쉬운 방법은 없음
	- 다른 metric을 optimize 

### Classification metrics optimization 2
- AUC
	- Pointwise loss
		- <img src="https://www.dropbox.com/s/ql2s1o0txg48750/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.01.56.png?raw=1">
		- single object에 대한 loss
	- Pairwise loss   	    
		- <img src="https://www.dropbox.com/s/muz4efdrga7ih3v/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.02.13.png?raw=1">
		- pairwise loss
	- <img src="https://www.dropbox.com/s/100wz8pw9adr1bg/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.02.26.png?raw=1">
- Quadratic weighted Kappa
	- Optimize MSE and find right thresholds
		- Simple!
	- Custom smooth loss for GBDT or neural nets
		- Harder   
	- <img src="https://www.dropbox.com/s/zdhe8unk6w8bi1u/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.06.15.png?raw=1">
	- [구현 코드](https://hub.coursera-notebooks.org/user/ynvomuepahfsgzmplvnxhw/notebooks/readonly/reading_materials/Metrics_video8_soft_kappa_xgboost.ipynb)

## Mean encodings
---

- Powerful technique!
- 동의어 : likelihood encoding, target encoding


### Concept of mean encoding
- Mean Encoding
	- Categorical Feature를 기준으로 하여 Target값과 통계적인 연산을 하는 것 
	- Median, Mean, Std, Min, Max 등의 Feature 추가
- Using target to generate features
- <img src="https://www.dropbox.com/s/9yd335yu9qg4ovu/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.35.51.png?raw=1">
	- Moscow는 총 5개가 있고, target에 1인 값이 2개라서 0.4
	- Tver는 총 5개 있고, target에 1인 값이 4개라서 0.8
- 왜 이게 작동하는가?
	- 1) Label encoding은 순서없이 random. target에 대한 correlation이 없음
	- 2) mean encoding은 0부터 1까지로 나뉘어 Target값과 직접적 연관성이 생김
	- <img src="https://www.dropbox.com/s/2tukemvxhwndp5t/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.45.13.png?raw=1">
	- <img src="https://www.dropbox.com/s/qe49sypg7lr2p27/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.54.45.png?raw=1">
	- 짧은 tree여도 더 나은 loss를 얻음
- 사용 예시
	- <img src="https://www.dropbox.com/s/gohmxkm3tdzjifd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.58.28.png?raw=1">
	- Tree depth를 증가시킬 때, 계속해서 점수가 증가하고 validation score도 같이 증가하면(오버피팅이 아니라는 뜻) Tree model이 분할에 대한 정보가 더 필요한 상황! 이럴 때 Mean Encoding이 사용
	- <img src="https://www.dropbox.com/s/b6suevtecrn0qcb/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2019.58.55.png?raw=1">
	- <img src="https://www.dropbox.com/s/8r4vxebq7fm0azg/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.00.47.png?raw=1">  
		- 오버피팅!
		- 우선 오버피팅을 잘 커버해야 함(정규화 필요)
		- 아래와 같은 데이터여서 오버피팅
		- <img src="https://www.dropbox.com/s/pm0lkdgfxjktjxj/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.02.48.png?raw=1">

### Regularization
- Mean encoding 자체는 유용하지 않을 수 있음. 데이터 정규화 필요
- Regularization
	- Training data에서 CV loop 돌리기
	- Smoothing(카테고리 수를)
	- Random noise 추가
	- Sorting and calculating expanding mean
- CV loop
	- Robust하고 직관적
	- 보통 4-5fold를 진행하면 괜찮은 결과
	- LOO같은 극단적 상황을 조심할 필요가 있음
		- 카테고리 수가 너무 작은 경우엔 오버피팅
	- <img src="https://www.dropbox.com/s/rvfcwp9ozw9bozv/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.13.11.png?raw=1">
	- <img src="https://www.dropbox.com/s/gkfsxt3sgxbjflh/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.13.29.png?raw=1">
	- <img src="https://www.dropbox.com/s/6iv50gvhi8r70bf/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.15.29.png?raw=1">
- Smoothing
	- 하이퍼 파라미터 : 알파
		- 보통 알파는 카테고리 사이즈와 동일할 때, 신뢰할 수 있음  
	- <img src="https://www.dropbox.com/s/xjuj2kah7evaipn/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.32.42.png?raw=1">
- Noise
	 - 노이즈가 encoidng의 quality를 저하시킴
	 - 얼마나 noise를 추가해야 할까? 고민해야 해서 불안정하고 잘 사용하기 어려움
	 - 보통 LOO와 함께 사용됨
- Expanding mean
	- 보통의 Mean ecnoding은 각 category 변수에 하나의 값으로 나오지만, expanding mean은 uniform하지 않음
	- leak이 적고 catboost에서 많은 성능 향상을 가져옴 
	- <img src="https://www.dropbox.com/s/rfjc4c1dzegwfzu/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.35.36.png?raw=1"> 


### Extensions and generalizations
- Regression and multiclass
	- Regession엔 median, percentiles, std, distribution bins 등을 추가 (통계적 정보)
	- multiclass엔 클래스만큼 다른 encoding. 각 클래스 encoding마다 다른 정보를 제공해 좋은 결과를 예상
- Many-tomany relations
	- cross product
	- 아래의 예는 APP_id로 나눔
	- <img src="https://www.dropbox.com/s/qd6ofdyjpxn5v1w/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2020.58.52.png?raw=1">
- Time series 
	- Limitation
	- Complicated features
	- Rolling 
	- <img src="https://www.dropbox.com/s/pb1sg8feqjgyod7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2021.01.01.png?raw=1">
- Interactions and numerical features
	- Tree의 상호 작용을 분석
	- Tree Model이 Split할 때 feature1과 feature2 처럼되면 서로 상호작용하! 이런 분할이 많을수록 Mean Encoding시 좋음
	- Feature끼리 합하여 Target Mean Encoding 수행 
	- <img src="https://www.dropbox.com/s/agvi4ftz973t01l/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202018-08-23%2021.03.03.png?raw=1">

## Reference
- [Coursera How to win a data science competition](https://www.coursera.org/learn/competitive-data-science)
- [Competitive-data-science Github](https://github.com/hse-aml/competitive-data-science)