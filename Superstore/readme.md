# [DA-Learning] Superstore

## 1. 데이터 둘러보기.

### 칼럼 설명

- Row ID : 일련 번호
- Order ID : 주문 코드
- Order Date : 주문 일자
- Ship Date : 배송 일자
- Ship Mode : 배송 유형 (1st, 2nd, standard class)
- Customer ID : 고객 ID
- Customer Name : 고객 이름
- Segment : 주문 세그먼트 (어떤 목적으로 주문했는지 - customer, corporate, home-office)
- Country : 주문 국가
- City : 주문 도시
- State : 주문 주
- Postal Code : 주문 우편번호
- Region : 주문 지역 (central, east, south, west)
- Product ID : 상품 코드
- Category : 카테고리-대분류
- Sub-Category : 카테고리-중분류
- Product Name : 상품 이름
- Sales : 판매 가격
- Quantity : 수량
- Discount : 할인
- Profit : 마진

### 범주형 변수 vs 연속형 변수

- 범주형 변수 (빈도 계산 가능)
  > Row ID, Order ID, Ship Mode, Customer ID, Customer Name, SEgment, Country, City, State, PostalCode, Region, Product ID, Category, Sub-Category, Product Name
- 연속형 변수 (수치 계산 가능)
  > Order Date, Ship Date, Sales, Quantity, Discount, Profit

info, describe, 결측치를 확인해보니 결측치 손실은 없었다. 그리고 데이터에 관하여 확인해보니, 아래와 같은 결과가 나왔다.

```
총 데이터 수는 209874개 이다.
총 결측치의 개수는 0개이고, 총 데이터의 0.00%를 차지하고 있다.
주문 기간은 2014-01-03 ~ 2017-12-30 까지이다.
전체 판매 물건 종류는 1850개이다.
배송 유형은 ['Second Class' 'Standard Class' 'First Class' 'Same Day'] 이다
구매 목적은 ['Consumer' 'Corporate' 'Home Office'] 이다.
구매 지역은 ['South' 'West' 'Central' 'East'] 이다.
상품 대분류는 ['Furniture' 'Office Supplies' 'Technology'] 이다.
상품 중분류는 ['Bookcases' 'Chairs' 'Labels' 'Tables' 'Storage' 'Furnishings' 'Art'
 'Phones' 'Binders' 'Appliances' 'Paper' 'Accessories' 'Envelopes'
 'Fasteners' 'Supplies' 'Machines' 'Copiers'] 이다.
```

## 2. 질문하기.

### [강사님 질문 / 나의 질문]

- 어떤 종류(sub-category)의 물건이 가장 많이 팔렸니?
- 어느 도시에서 가장 주문량이 많았니?
- 세그먼트와 지역에 따른 주문량과 판매 금액?
- 할인률이 높을 수록 마진은 낮을까?
- 어떤 종류의 상품이 매출이 가장 높을까?
- 지도 위에 판매량을 나타낼 수 있을까?

## 3. 데이터 정비하기.

### 3-1. Date 모두 Datetime으로 변경하기.

datetime으로 변경한 뒤에 데이터 분석을 편리하게 하기 위하여 미리 다 쪼개서 칼럼으로 저장해준다.

### 3-2. 모든 칼럼 소문자로 변경하기.

데이터 분석 시에 칼럼이 소문자인 것이 더 편리하기 때문에, 소문자로 변경해주면 된다.

### 3-3. 중복이 있는지 확인하기.

```
df_copy.duplicated().sum()
```

### 3-4. 사용하지 않는 칼럼 제거하기.

```
df_copy=df_copy.drop(["postal code"], axis=1)
```

## 4. EDA & Visualization

### 4-1. 간단한 분석

- 주문 순위가 상위 10등인 상품들을 알아보니, 문구류, 사무용품 그리고 가구가 많은 것을 알 수 있다.

![](https://velog.velcdn.com/images/tino-kim/post/332b48d5-9a51-4b63-a9f7-de0921914b00/image.png)

funiture는 0 근처에 많이 분포되어 있다. funiture는 5000 미만의 판매가를 가지고, office supplies는 10000 미만의 판매가를, technology는 20000 이상의 판매가를 가지고 있다.

funiture는 0%, 20% 에서 할인율이 많이 분포되어 있고, 40% ~ 60% 도 분포되어 있다. office supplies는 0%, 20% 에서 많이 분포되어 있고, 60% ~ 80% 좀 넘게 할인율이 분포되어 있다. technology도 0%, 20% 에서 할인율이 많이 분포되어 있고, 40% 에도 좀 분포되어 있다.

![](https://velog.velcdn.com/images/tino-kim/post/a1cb4149-57c6-48b2-8669-442cb30e55b3/image.png)

가구와 사무용품이 많은 분포를 차지하고 있다.

![](https://velog.velcdn.com/images/tino-kim/post/8188dc5f-451f-4bbf-b1a1-7080a1337b79/image.png)

대분류와 중분류로 나누어서 주문량을 알아보았다.

![](https://velog.velcdn.com/images/tino-kim/post/058cccf2-bee2-4f39-b22f-a737587aa54c/image.png)

Binders가 가장 많고, 그 다음은 Paper가 많다. 그리고 Machines와 Copiers가 가장 적다. 종이류와 사무 용품이 많이 나가고, 기계가 덜 나가는 것 같다.

### 4-2. 주 별 주문량 확인하기.

```
df_copy["city"].value_counts().nlargest(10)
# nlargest : 숫자가 큰 것 골라오기.
```

뉴욕이 가장 주문량이 많고, 스프링 필드가 10위를 차지하고 있다.

```
df_copy["state"].sort_values().drop_duplicates().values
# (중복 되는 부분은 제외하고) 알파벳 순서로 인덱스 만들어주기.
```

![](https://velog.velcdn.com/images/tino-kim/post/22c64bdc-6180-4520-b64d-2df21c546391/image.png)

캘리포니아에서 가장 주문량이 많았고, 그 다음은 뉴욕 그리고 텍사스이다.

### 4-3. 세그먼트 별 / 지역 별 주문량

![](https://velog.velcdn.com/images/tino-kim/post/62254e65-67f6-4929-b343-7b70b9c554f9/image.png)

consumer 목적인 경우가 가장 많았고, 그리고 home office 목적인 경우가 가장 적었다. (Customer > Corporate > Home Office) 그리고 west에서 가장 주문량이 많고, south에서 가장 주문량이 적었다. (West > East > Central > South)

### 4-4. 할인율에 따른 마진

- 할인을 많이 하면, 당연히 파는 사람의 마진이 줄 것이다.
- lineplot을 이용해서 정말 이 자료에서도 이렇게 되는지 파악해보기.

![](https://velog.velcdn.com/images/tino-kim/post/f3824ffd-4516-4695-8309-b6002f309dd6/image.png)

0% ~ 50% 까지는 내가 예상한 대로 판매자의 이익이 줄고 있다. 하지만 50% ~ 60% 에서는 판매자의 이익이 증가한 것을 알 수 있다. 왜 그럴까? (통상적으로는 할인율이 높아질 수록, 판매자의 이익이 줄어들어야만 한다. 어떤 상품 때문에, 이익이 변화하는 것일까?)

**[TIP] 구매 데이터가 있는 경우에, lineplot을 잘 이용하면 회귀 추정선을 그려서 인사이트를 도출해낼 수 있다.**

### 4-5. 상관 관계 Heatmap 그리기.

```
# 상관 관계 표현하기. 수치형 변수를 모두 상관 계수로 표현하기.
corr=df_copy[["sales", "quantity", "discount", "profit"]].corr()
corr
```

![](https://velog.velcdn.com/images/tino-kim/post/4e668b53-074b-45d8-8600-245281f3eec2/image.png)

- 판매가가 높을 수록 마진이 높다. (당연한 사실) - 정비례 관계
- 할인율이 높을 수록 마진은 적다. - 반비례 관계

상관 관계 그래프에서 대각선 윗부분은 아랫 부분과 겹치기 때문에, 가독성을 위하여 제거해줄 수 있다.

![](https://velog.velcdn.com/images/tino-kim/post/8864b582-ca7c-46a0-84fc-9a4730c6aa29/image.png)

### 4-6. 판매 금액에 따른 상위 10개 품목

![](https://velog.velcdn.com/images/tino-kim/post/dee7c073-5af5-4551-bc86-f8de5ef4408e/image.png)

### 4-7. 주 별 총 판매 금액을 지도 위에 반응형으로 나타내기.

- plotly를 이용해서 반응형 그래프를 그리기.
- 각각의 주 당 어떤 코드랑 연결되는지 적어주기.
- 마지막에 지도와 데이터를 매핑시킬 예정이다. 그 과정에서 state_code가 필요하다.
- 각각의 주 당 어떤 코드랑 연결되는지 적어주기.
- 마지막에 지도와 데이터를 매핑시킬 예정이다. 그 과정에서 state_code가 필요하다.

```
state = ['Alabama', 'Arizona' ,'Arkansas', 'California', 'Colorado', 'Connecticut', 'Delaware', 'Florida',
         'Georgia', 'Idaho', 'Illinois', 'Indiana', 'Iowa', 'Kansas', 'Kentucky', 'Louisiana', 'Maine', 'Maryland',
         'Massachusetts', 'Michigan', 'Minnesota', 'Mississippi', 'Missouri', 'Montana','Nebraska', 'Nevada', 'New Hampshire',
         'New Jersey', 'New Mexico', 'New York', 'North Carolina', 'North Dakota', 'Ohio', 'Oklahoma', 'Oregon', 'Pennsylvania',
         'Rhode Island', 'South Carolina', 'South Dakota', 'Tennessee', 'Texas', 'Utah', 'Vermont', 'Virginia', 'Washington',
         'West Virginia', 'Wisconsin','Wyoming']

state_code = ['AL','AZ','AR','CA','CO','CT','DE','FL','GA','ID','IL','IN','IA','KS','KY','LA','ME','MD','MA',
              'MI','MN','MS','MO','MT','NE','NV','NH','NJ','NM','NY','NC','ND','OH','OK','OR','PA','RI','SC','SD','TN',
              'TX','UT','VT','VA','WA','WV','WI','WY']

state_cd=pd.DataFrame(state, state_code)
state_cd

sales.insert(1, "state_cd", state_cd["state_cd"])
# 칼럼 1에 state_cd 라는 칼럼명을 가진 칼럼을 넣어주기.
sales
```

2개의 데이터 프레임을 병합해준 뒤에, plotly를 이용하여 반응형 그래프를 그려주기.

![](https://velog.velcdn.com/images/tino-kim/post/3c365885-c84b-4b35-a055-87d26ddc739b/image.png)

### 4-8. 워드 클라우드 이용하여, 상품 이름과 중분류의 빈도를 살펴보기.

- 상품 이름에서는 "어떤 단어가 많이 나왔는지" 보여주고 있다.

> **WordCloud 이용하여 원하는 그림대로 그려주는 방법**

1. 그리고 싶은 그림 가져오기.
2. 뒷 배경을 흰색으로 변경하기. (new)
3. 흰 배경 위에 이미지 올려주기. (paste)
4. 배열 형태로 만들어주기.

![](https://velog.velcdn.com/images/tino-kim/post/33b24592-0720-400b-9317-8a6d0324c116/image.png)

상품 이름에서는 Series와 Chair, Name가 많이 등장했음을 알 수 있다. (단어의 크기는 상품 이름 빈도수와 비례한다.)

![](https://velog.velcdn.com/images/tino-kim/post/7b2d6779-703f-47a0-927b-7a6cdac87cc7/image.png)

중분류에서는 가구, 테이블, 책꽂이 등등 가구에 관련된 단어와 사무용품이 많은 느낌이 든다.

> **빈도수로 단어 크기 표현하는 방법**

1. 일단, list로 만들어주고 dict를 이용해서 각각의 빈도수를 그려준다.
2. 빈도수로 워드 클라우드를 그리고 싶은 경우에는 fit_words를 이용한다.
3. 그 워드 클라우드를 이미지로 만들어준다.

![](https://velog.velcdn.com/images/tino-kim/post/bf343100-bce6-4bc6-be62-83e07dd40a00/image.png)

글자가 크면 클수록, 자주 등장한 단어를 의미한다. 빈도수를 구하는 경우는 fit_words를 이용한다.

product name도 단어의 빈도수를 이용해서 워드 클라우드를 그려주기.

![](https://velog.velcdn.com/images/tino-kim/post/3619e599-3643-4831-8b51-4fe2045e8e77/image.png)

글자가 크면 클수록, 자주 등장한 단어를 의미한다. 빈도수를 구하는 경우는 fit_words를 이용한다.

가장 자주 나온 단어 상위 20개로 워드 클라우드 만들어주기. 일단 내림차순으로 정렬한 뒤에, 20개만 뽑아와야 한다.

![](https://velog.velcdn.com/images/tino-kim/post/62cfb696-85b2-4983-bb4e-dfbc8192945c/image.png)

### 4-9. 시계열 그래프 그리기.

![](https://velog.velcdn.com/images/tino-kim/post/6e46d928-5fb0-481d-90d3-46845d4c0816/image.png)

1. **[주문]** 일요일에 가장 적게 주문하였고, 화요일에 가장 주문을 많이하였다.
2. **[배송]** 일요일에 가장 배송이 많이 되었고, 목요일에 가장 배송이 적게 되었다.

![](https://velog.velcdn.com/images/tino-kim/post/896ef8a3-c314-4426-b55a-ac8730029da9/image.png)

대부분 배송 기간은 4일 정도 걸리고, 1일이 가장 적었다. 거의 주(state)에서 주(state)로 배송되어지기 때문에 기간이 좀 오래걸리는 것 같다.

각각의 배송 기간 별로 가장 많이 팔린 중분류를 살펴보니, 모두 Paper 또는 Binders 였다. 사무용품 중에서도 Binders가 많음을 알 수 있다. (위에서 알 수 있듯이, 전체적으로 Paper와 Binders가 많았다.)

![](https://velog.velcdn.com/images/tino-kim/post/7adb8d0f-d6a7-4806-9e62-3c1a2a3dd1ae/image.png)

**[주문]**

1. 매년 총 매출액이 증가하고 있다.
2. 9월에 매출액이 가장 높았고, 2월에 매출액이 가장 낮았다.
3. 17일에 매출액이 가장 높았고, 29일에 매출액이 가장 낮았다.

**[배송]**

1. 2014년 ~ 2017년까지 꾸준히 매출액이 증가하다가, 2018년에 갑자기 급감하였다. (왜 그럴까? 더 자료가 많으면 좋았을 것 같다.)
2. 위와 마찬가지로, 9월에 가장 매출액이 높고 2월에 가장 매출액이 낮았다.
3. 21일에 매출액이 가장 높았고, 7일에 매출액이 가장 낮았다.

중분류 별 매출액의 총합을 보니, Phones가 가장 많았고 그 뒤로는 가구와 사무용품이 있었다. Binders가 의외로 큰 매출액을 뽑아내었다.

9월에 가장 총 매출액이 많았는데, 가구와 사무용품이 상위권을 차지하였다. Binders가 3위로 많은 매출액을 뽑아내고 있다. (밑에서 알아보니, 금액이 천차만별이다.)

2월에 가장 총 매출액이 적었는데, 기계와 가구가 많이 팔린 편이었다. 각각의 금액 자체는 크지만, 주문 개수가 적어서 총 매출액이 가장 적다.
