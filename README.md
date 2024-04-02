# MUSINSA 회귀 모델을 이용한 판매량 예측
## 프로젝트 개요
### 한국 e커머스 내 무신사의 위치
- 국내 e커머스 시장은 5년 이상 꾸준히 성장세
- 그 중 패션 분야에서 트렌드로 자리잡은 무신사

> 패션앱 시장에서 독보적인 위치를 가진 무신사
> 
> -> <b>무신사의 제품 정보를 크롤링하여, 제품의 1년 누적 판매량을 예측해보자</b>

<br>

## 데이터 수집 및 전처리
### 1. 데이터 정의
1) 모델링에 사용할 피쳐가 될 제품 데이터 종류 정의

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/1fe1fc79-324f-4e12-9dca-b190b0df5d0d)

2) 리뷰감성분석점수 파생변수 생성을 위한 데이터 종류 정의

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/bee2001a-5081-461c-a646-a296aebc804c)

### 2. 웹 데이터 수집
1) 상품 데이터 크롤링
- 대분류가 상의, 바지, 아우터인 카테고리의 데이터 크롤링 진행
- 무신사 추천순으로 상의, 바지, 아우터 카테고리별로 총 8000여개 수집
2) 리뷰 데이터 크롤링
- 수집한 각 상품별 리뷰 데이터 데이터 크롤링 진행
- 각 상품 별 스타일/이미지/일반 각각 최대 15개의 리뷰 수집
- 총 180,000여개 수집
<br>

### 3. 리뷰감성분석 파생변수 생성
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/3c4d47de-249c-40df-9fe3-2b543e8430f9)

1) 무신사에서 학습용 리뷰 데이터 약 2만 건을 추가 수집
- pos, neu, neg 3가지 클래스를 골고루 수집하기 위해 ‘평점 높은 순’, ‘평점 낮은 순’필터 이용 
- 중복 제거
2) 지도 학습 진행을 위해 user_rating 기준으로 label 부여
- 80점 이상이면 ‘pos’, 60점이면 ‘neu’, else ‘neg’
3) Okt().pos()로 토큰화 및 등장 빈도수로 정리
  (1) 한국어 분석을 위해 Okt().pos()를 이용해 형태소별 토큰화
  - 한국어의 경우 동음이의어 구별을 위해 품사를 함께 사용하면 분석 정확도가 향상됨
  - stopwords를 지정하여 사용
    ['의/Josa','가/Josa','이/Josa','은/Josa','들/Josa','는/Josa','좀/Noun','걍/Noun','과/Josa','도/Josa','를/Josa','으로/Josa','에/Josa','와/Josa','한/Noun','하다/Verb','을/Josa', '에서/Josa','에게/Josa', '하고/Josa', '이다/Verb', './Punctuation']
  
  ![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/cc77b1b5-bff5-4d3f-a780-f4bcc8bd302d)
  
  (2) 등장 빈도수가 낮은 토큰 정리
  threshold = 3
  등장 빈도가 threshold 미만인 희귀 단어 조사
  희귀 단어의 수는 많지만, 훈련 데이터에서 희귀 단어 등장 빈도 비율은 매우 적은 수치인 2.35%
  등장 빈도가 3회 미만 단어들은 제거

  ![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/2cf8384e-6784-4cae-8194-1c759cc5f485)

4) TfidfVectorize와 Oversampling
토큰을 하나의 text로 합치고 TfidfVectorize 진행
클래스 간 비율 차이가 큰 것을 보완하기 위해 학습데이터에 SMOTE Oversampling 진행

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/a07cc770-a630-4cbc-b16a-644a423d6076)
<pre>     ↓ </pre>  
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/619bea44-f3d7-42f7-a619-530a9617060c)


5) 모델 학습과 성능 평가 및 변수화
  (1) 모델 학습과 성능 평가
  MultinomialNB, LightGBMClassifier, LogisticRegression 학습 후 성능 평가
  모델 별로 Precision과 Recall에서 큰 차이가 없기 때문에 Accuracy가 가장 높은 LightGBMClassifier 모델 채택
  (2) 변수화
  - 실제 제품 리뷰를 위 LightGBMClassifier를 이용해 예측값 도출
  - ‘pos’는 1, ‘neu’는 0.5, ‘neg’는 0으로 치환하여 제품별 리뷰스코어(score) 파생변수 생성 및 추가
  - 
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/2adab8b9-dcb9-4b0d-a615-81c66719f559)

<br>

### 4. 데이터 전처리
1) feature별 NaN값 처리
2) 
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/8834e108-fcb4-46f4-ad2f-7dc082f2cee5)

3) 논리적 이상치 제거
리뷰수(review)가 판매량(buy)보다 많은 경우 웹에서 표기가 잘못된 데이터는 고쳐주되 그렇지 않은 데이터들에 한에서 제거를 진행하였음

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/6dcbb414-2fb7-43de-b6e9-b61b271ab243)

4) buy_age 범주형 데이터 맵핑
- 해당 상품을 가장 많이 구매한 연령대를 나타내는 buy_age의 값
- 동률이 아닐 경우 (‘나이구간’, 0)이고 동률일 경우 (‘나이구간’, ‘나이구간’)으로 수집
- buy_age의 경우 동률인 경우가 있기 때문에 buy_age1, buy_age2로 컬럼을 나눔
- 범위를 0(NaN),1,2,3,4,5,6으로 구분지어주었음

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/0c6245ae-776c-456a-add5-5487634c0486)

<br>

## EDA
1) 데이터 분포
   
   ![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/2d2c844f-0cc4-4da3-8165-898010b34377)
<br>

2) 상관 관계, Target 데이터인 누적 판매량(buy)과 상관도가 높은 feature:
- 조회수(view)
- 좋아요(like)
- 후기수(review)

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/37a3182d-6780-4fad-98d1-228861727e00)
<br>

3) feature 탐색

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/688f32e3-7430-4072-90fc-8f35244f8523)
<br>

- 한정판매 또는 단독판매인 상품들이 아닌 상품들보다 누적판매량의 평균이 약 2배 더 높다.
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/70704330-0a58-4c1a-ad09-ff3cfdf914dd)
  
- 타겟 성별: 상품 생산 시 타겟으로 한 성별 (예: 스커트, 여자)
- 누적 판매량 평균이 가장 높은 상품: 남녀 공용 타겟
- 누적 판매량 평균이 가장 낮은 상품: 여성 타겟
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/14766b65-6a37-45e6-a93e-7af367dc0f6d)

- 상품들은 2.3 ~ 5.0까지의 평점을 받았다.
- 4.7 이상 좋은 평점을 받은 상품들이 많은 것으로 보아 
- 무신사 고객들은 구매한 상품에 대한 만족도가 높다.
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/59e907df-09c6-463d-873b-ae2226875091)

- 판매량과 평점은 상관관계 0.03으로 높지 않음
- 판매량이 적더라도 평점이 높을 수 있기 때문
- 하지만 평점이 높은 제품에서는 판매량이 높게 나타남.
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/be39b7f8-4dc6-407a-bf95-782f4b31c211)

- 주로 20대인 고객을 만족시키는 상품이 많은 것으로 보인다.
- 30대에서 줄어들다가 40대 이상이 높아지는 이유로 ‘힙한 브랜드’를 판매하는 무신사의 이미지를 고려하여 학부모 계정으로 구매한 미성년자를 고려할 수 있다.
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/eda92516-95b9-4f7d-a1c5-84c9e49fc4b0)

- 남성은 남성 제품보다 공용 제품을 더 많이 구매
- 남성이 여성 제품을 구매한 경우도 적지 않다.
- 여성도 여성 겨냥 제품보다 남자 혹은 공용제품을 더 많이 구매
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/e2639c7e-4679-4d37-8cc0-79c87521a0d6)

- 20대 이상의 모든 연령층에서 공용 제품 판매량이 가장 많다.
- 0대에서는 남성 타겟 제품이 가장 많다
- 여성 타겟 제품은 비교적 판매량이 떨어진다.
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/2481a989-fe54-4252-b5cb-a6543120c4c8)
<br>

4) 구매나이에 따른 구매전환율 및 리뷰작성전환율

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/d756eff9-05f2-4e22-9823-e775c19f4086)

- 구매전환율: 판매량/조회수, 리뷰작성전환율: 리뷰수/판매량
- 남성의 구매전환율이 여성의 구매전환율보다 약 0.07% 정도 더 높음.
- 여성이 남성보다 실제 구매를 하기까지 더 많은 시간과 노력을 투자한다고 추측
- 남성의 리뷰작성전환율이 여성의 리뷰작성전환율보다 0.04% 더 높음.
- 남성이 여성의 경우보다 리뷰 작성에 좀 더 긍정적이거나 조금 더 성실하다고 볼 수 있음.
<br>

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/753c2d02-1320-4b52-942b-e8b25ea8dec3)

- 구매 전환율을 10대 후반에서 20대 초반이 가장 높고, 이후로 점점 떨어지다가 40세 이상에서 다시 상승
- 리뷰 작성은 10대를 제외하고 대부분 비슷한 것을 확인
- 구매 나이에 따른 리뷰작성전환율은 구매 나이에 따른 구매전환율과 다소 상반됨
<br>

5) 리뷰 데이터 EDA
- 등장 빈도수

  ![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/1d7aefb1-7364-46f7-b944-879213c9a08b)
<br>

- 워드 클라우드

  ![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/1fb401f6-f040-4ae1-98be-870d1f38975b)
<br>

## 모델 학습과 평가
1. Ridge

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/f446d7d2-f47a-47ba-926f-87bc4157e3f0)
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/0e44cdd0-a663-4a01-9ef2-fd6e857407d8)

2. OLS

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/823fcc2d-3912-469a-a717-801fe97762b0)
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/b90dc1bf-7194-4fc2-8fee-e8ac05106161)

3. Multiple Regression + Lasso
   
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/18665d6c-a0c7-4346-b1f0-24486b23ef1b)
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/03d573ac-b234-48ef-87fb-397d3df893ef)

4. LGBM Regressor

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/020ec010-5267-4eff-a115-56a94aaa6c6d)
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/266527e3-a221-4aa9-8fb1-126442084d31)

5.XGBoost Regressor

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/c7ae1511-4f4a-49fb-a81c-0095d4897776)
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/91b67038-4972-48d7-9b4a-4449cb4a1914)

### 모델 성능 비교

![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/8cd8aa4d-4d00-4a56-85bf-d6945e8fb292)


## 결론
### 최종 모델
<b> Multiple Regression + Lasso </b>

### lesson&learned
![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/4fa21409-3e87-4bfa-b73b-d1e2cfb02594)


![image](https://github.com/gaeju/Predicting_sales_volume_for_Musinsa/assets/100760127/ba3c3a54-9bf3-4f6f-a49a-2a8cdb76407c)
