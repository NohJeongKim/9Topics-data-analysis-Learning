# [DA-Learning] Commerce

## 1. 데이터 보기.

**[Error Solve]** 사실 .xlsx 파일이 안 불러와져서 구글링도 해보고, 네이버 카페에도 질문 글을 올렸었다.

- [문제점 1] import xlrd를 해주지 않았다. xlrd는 .xls, .xlsx 파일을 읽어주는 모듈이다.
- [문제점 2] fsspec module을 설치하지 않았다. fsspec은 파일 시스템 사양과 관련된 모듈이다.
- [문제점 3] 절대 경로를 이용하지 않았다. 반드시 C://이라고 고쳐주고, 나머지도 /라고 고쳐주기. 절대 경로를 변수에 담아서 이용하면 편리하다. csv 파일은 상대 경로로 불러와도 잘 되지만, xlsx는 절대 경로로 불러와야 잘 된다.

두 가지 문제점을 해결하고 나니, 제대로 데이터 프레임을 불러왔다.

```
excel="C://Users/ajouu/Documents/python-project-Basic/data/Online Retail.xlsx"
# C:// 라고 적어주기. 그리고 절대 경로를 변수에 넣어서 이용하기.
excel # 절대 경로로 적어주기.

df=pd.read_excel(excel, engine="openpyxl") # engine="openpyxl" 적어주기.
df.head()
```

원본 데이터를 보호하기 위하여, copy를 이용하여 복제한 뒤에 df_copy에 넣어주기.

### 칼럼 설명

- InvoiceNo : 주문 번호
- StockCode : 상품 코드
- Description : 상품 설명
- Quantity : 수량
- InvoiceDate : 주문 날자
- UnitPrice : 개별 가격
- CustomerID : 고객 번호
- Country : 국가

### 범주형 변수 vs 연속형 변수

- [범주형 변수] : InvoiceNo, StockCode, Description, CustomerID, Country <빈도 계산 가능>
- [연속형 변수] : Quantity, InvoiceDAte, UnitPrice <통계적 수치 계산 가능>

dtypes, info, 결측치를 확인해보니 결측치가 많은 행이 존재한다.

```
총 데이터의 개수 : 4335272개
총 결측치의 개수는 136534개이고, 전체 데이터의 3.15%가 결측치이다.
전체 국가의 수는 38개이다.
전체 판매 물건 수는 : 4223개이다.
```

## 2. 질문하기.

### [강사님 질문]

- 어떤 고객이 가장 지출을 많이 했을까?
- 상품 금액의 분포?
- 어떤 물건의 주문량이 높을까?
- 날짜에 따라 판매 금액을 확인하자.
- 요일에 따라 주문량이 다를까?
- 국가 별 평균 주문 금액?
- 이 쇼핑몰 판매 물품의 주요 키워드는?

## 3. 데이터 정비하기.

### 3-1. 칼럼 명 변경하기.

- 칼럼 명이 대문자인 경우에는 굉장히 불편하다. 모두 소문자로 변경해주는 것이 편리하다.
- 칼럼에 어떤 값이 들어있는지 확인해보니, quantity 칼럼에 음수가 들어있다. 음수를 처리해줘야 한다.

### 3-2. 결측치 처리하기.

- any를 이용해서 결측치가 하나라도 있는 행을 가져올 수 있다.
- customerid가 결측치인 경우에는 제대로 배송된 건지의 여부를 알 수 없기에, 그냥 모두 제거하고 데이터 분석하기.

### 3-3. 칼럼 타입을 변경하기.

**내가 고려해야 되는 점**

- customerid : float64 > int64 로 변경하기. 아예 데이터 타입을 변경하기.
- quantity : 음수 > 양수인 것만 고려하기. 이 부분은 데이터 타입 변경 후, spent 라는 새로운 칼럼을 만들어서 다루기.

### 3-4. 새로운 칼럼 만들기.

- 총 소비 금액을 담은 spent 칼럼을 만들어주기.
- invoicedate를 쪼개서 칼럼으로 만들어주기. 나중에 분석할 때 편하게 이용하기 위함이다.

```
df_copy["spent"]=df_copy["quantity"]*df_copy["unitprice"]
df_copy.head() # 소비 금액 칼럼을 생성하기.
```

- 원할한 데이터 분석을 하기 위해서 미리 쪼개서 칼럼으로 만들어주기.

```
df_copy["year"]=df_copy["invoicedate"].dt.year
df_copy["month"]=df_copy["invoicedate"].dt.month
df_copy["day"]=df_copy["invoicedate"].dt.day
df_copy["weekday"]=df_copy["invoicedate"].dt.dayofweek
# 0 ~ 6 (월요일 ~ 일요일)
df_copy["day_name"]=df_copy["invoicedate"].dt.day_name()
df_copy["hour"]=df_copy["invoicedate"].dt.hour
df_copy.head()
```

## 4. EDA & Visualization

### 4-1. 간단한 분석

1. 주문 순위가 상위 5위인 제품 그래프
   ![](https://velog.velcdn.com/images/tino-kim/post/0844fc89-2afb-4e32-981b-d3a68adaf6a4/image.png)

2. argmax와 argmin을 이용하여 가장 큰 값의 인덱스와 가장 작은 값의 인덱스를 구해보자.

```
print(f"가장 적게 구매한 고객 아이디는 {spent_cus.argmin()}이고, 금액은 {spent_cus.min()}이다.")
print(f"가장 많이 구매한 고객 아이디는 {spent_cus.argmax()}이고, 금액은 {spent_cus.max()}이다.")
```

```
가장 적게 구매한 고객 아이디는 3217이고, 금액은 3.75이다.
가장 많이 구매한 고객 아이디는 1689이고, 금액은 280206.02이다.
```

추가로 최소 가격, 최대 가격, 평균, 중앙값 등을 구할 수 있다.

```
개별 품목의 최소 가격은 0.0 파운드이고, 최대 가격은 8142.75 파운드다.
개별 품목의 평균은 3.1161744805540756 파운드다.
개별 품목의 중앙값은 1.95 파운드다.
```

3. [boxplot] 최대값, 최소값, 중앙값을 이용해서 자료의 측정 값들의 분포를 쉽게 볼 수 있다.
   ![](https://velog.velcdn.com/images/tino-kim/post/c00bfb32-adb4-4c81-80f4-406b9bed9c36/image.png)

[해석하는 방법] 0에서 7.xx 파운드까지 대부분의 상품 금액이 분포하는데, 초록색 상자 안 정도의 금액이 많이 분포되어 있음을 의미한다. 점으로 표기되어 있는 부분은 이상점이다.

4. n 파운드 이하의 상품 주문의 전체 물건 건수 / 전체 수익의 몇 퍼센트를 차지하는가?

```
7 파운드 이하의 상품 주문이 전체 물건 건수의 91.05%를 차지한다.
5 파운드 이하의 상품 주문이 전체 물건 건수의 87.14%를 차지한다.
```

이 상점에서는 상대적으로 저렴한 물건을 팔아서 이익을 내는 구조를 가지고 있다. (7 파운드 이하의 상품과 5 파운드 이하 상품의 비율이 크게 차이가 나지 않기 때문이다.)

```
7 파운드 이하의 상품은 전체 수익의 85.27%를 차지한다.
5 파운드 이하의 상품은 전체 수익의 79.68%를 차지한다.
```

이 상점에서는 전체 수익과 n 파운드 이하의 상품이 크게 관련성이 있지 않다.

### 4-2. 날짜에 따른 판매 금액 (시계열)

![](https://velog.velcdn.com/images/tino-kim/post/c610c653-912b-431c-bb46-d13d72aebd48/image.png)

![](https://velog.velcdn.com/images/tino-kim/post/1f7922eb-ef15-4c5d-989c-fa7690f12dad/image.png)

곳곳에 판매 금액이 어마어마하게 높은 데이터들은 유심하게 지켜봐야 한다.
groupby를 이용하여 그 지점을 찾아보니, 2011-12-09 > 2011-01-18 > 2011-06-10 순으로 소비 금액이 많은 편이다.

### 4-3. 요일 별 / 시간 별 주문 량

주문 번호가 1개인 것이 필요하기 때문에, drop_duplicates를 이용해서 주문 번호 1개 씩만 남겨 두고 처리하기.

![](https://velog.velcdn.com/images/tino-kim/post/a7eae9bd-b3cf-4ac7-b1f3-bbb7a66b0b8c/image.png)

주문량이 목요일이 가장 많고, 일요일이 가장 적다. 그리고 거의 2배 정도 차이가 난다. 목요일에 프로모션을 진행하거나 일요일에 주문량을 끌어올릴 수 있는 기획을 세우는 것이 좋을 것 같다.

![](https://velog.velcdn.com/images/tino-kim/post/9ab2bda5-7b72-4e67-a248-988776ce2660/image.png)

새벽과 저녁 시간에는 적고, 낮 시간 대에 주문량이 많이 분포되어 있다. 12시에서 1시 쯤 주문량이 가장 많은 편에 속한다.

### 4-4. 국가 별 1회 주문의 평균 구매 금액

국가 별 주문 건수를 센 후에, (전체 금액 / 주문 건수) 로 해주면 1회 주문의 구매 금액이 나온다.

```
df2=df_copy[["invoiceno", "quantity", "country", "spent"]]
df2

df2=df2.groupby(["country", "invoiceno"]).sum()
df2["count"]=1 # 주문 건수를 만들기 위함이다.
df2

df2=df2.groupby(["country"]).sum()
df2
# count가 전체 주문 건수가 된다. spent는 전체 주문 비용이 된다.

df2["avg_spent"]=df2["spent"]/df2["count"]
df2

df2.sort_values(["count", "avg_spent"], ascending=False, inplace=True)
df2.head()
```

![](https://velog.velcdn.com/images/tino-kim/post/b527ceaf-05a7-4e4d-b170-7c19640e302d/image.png)

싱가포르가 가장 1회 주문 평균 금액이 가장 높고, 사우디 아라비아가 1회 주문 평균 금액이 가장 낮다. 거의 5 ~ 6배 정도 차이나는 것 같다.

- 영국은 주문 건수는 많으나, 1회 주문 금액은 그리 높지 않다.
- 하지만, 해외에서는 주문 건수는 영국에 비해 적으나, 1회 주문 금액은 높은 편에 속한다.

![](https://velog.velcdn.com/images/tino-kim/post/a113c1fe-dfd2-49cd-92b0-0ad6dfd9130c/image.png)

영국이 가장 많고, 그 뒤로는 독일, 프랑스 등이 있다. 밑의 그래프는 y축 범위를 1 ~ 480 까지 지정해주었다.

### 4-5. 상품 명에서 가장 빈번하게 등장하는 단어

```
from PIL import Image
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator
```

![](https://velog.velcdn.com/images/tino-kim/post/67bceaf6-1460-4db8-80f1-98abdecbead8/image.png)

하얀색과 하트 그리고 아이 관련 물품이 많이 판매된 것을 볼 수 있다. 소녀 관련 물품도 많은 편이고, 주방 용품 판매량도 꽤 높은 편에 속한다.

### [나의 질문]

- 소비 금액이 많이 나왔던 3개 날짜에 대해서 알아보기.
- 월 별 판매 금액 시계열 그래프와 주문 횟수 시계열 그래프 그리기.
- 소비 금액이 가장 높은 상위 3개 국가의 월 별, 요일 별, 시간 별 시계열 그래프 그리기.

### 4-6. 소비 금액이 많이 나왔던 3개 날짜에 대해서 알아보기.

![](https://velog.velcdn.com/images/tino-kim/post/7f95b998-f56c-4f54-af33-bbb1d061afe7/image.png)

2011-12-09 에는 오전 9시에 총 주문량과 소비 금액 합계가 가장 많다. 소비 금액 합계와 총 주문량이 차이가 나는 것으로 보아, 한번에 많은 돈을 지불한 고객이 있어
보인다.

2011-08-18 에는 오전 10시에 총 주문량과 소비 금액 합계가 가장 많다. 소비 금액 합계와 총 주문량이 차이가 많이 나지 않는 것으로 보아, 소량 구매한 고객이 많아 보인다.

2011-06-10 에는 소비 금액 합계는 오후 3시에 가장 많았고, 총 주문향은 오전 12시에 가장 많았다. 오후 3시에 소비 금액 합계와 총 주문량이 많이 차이나는 것으로 보아, 한번에 많은 돈을 지불한 고객이 있어 보인다.

그렇다면, 각 날짜에 어떤 나라의 고객들이 최대로 이용했는지 수치를 알아볼 수 있다.

가장 비싼 상품의 가격은 18 달러 이고, 오전 10시, 11시, 12시에 가장 많이 팔렸다. 그리고 모두 영국 고객이다.

그리고 가장 비싼 상품의 가격은 647.50 달러이고, 오후 3시에 가장 많이 팔렸습니다. 모두 영국 고객입니다.

![](https://velog.velcdn.com/images/tino-kim/post/c1c6c897-9e21-48ec-9d09-1e0f4be01e2a/image.png)

2011-12-09 에는 총 소비 금액이 오전 9시에 가장 많았는데, 오전 9시에 개별 상품의 최대 구매 금액은 아니였다. 어느 정도 가격이 있는 제품을 많이 구매했거나 많은 수량을 구매했을 것이라고 추정할 수 있다. **_허나, 소비 금액에 비해 총 주문량도 그리 높지 않은 것으로 보아 한번에 여러 수량을 구매한 고객들이 많음을 추정할 수 있다._**

2011-01-18 에는 총 소비 금액은 오전 10시에 가장 많았는데, 오전 8시에 개별 상품 최대 가격이었다. 오전 10시에 적은 금액의 상품을 여러 번 구매했음을 알 수 있다.

2011-06-10 에는 총 소비 금액이 많은 시간은 오후 3시 였고, 총 주문량이 많은 시기는 오전 12시였다. 오후 3시에 엄청나게 비싼 물건을 팔아서 매출을 올렸고, 오전 12시에는 가격이 비교적 낮은 상품을 여러 번 판 것으로 볼 수 있다.

### 4-7. 월 별 평균 판매 금액 시계열 그래프와 주문 횟수 시계열 그래프 그리기.

![](https://velog.velcdn.com/images/tino-kim/post/2c51ac26-bc0d-40df-9754-baf226c3c4d9/image.png)

- 1월, 12월에 판매 금액이 굉장히 높다.
- 2월에 구매 횟수가 가장 적고, 11월에 구매 횟수가 굉장히 많다.
- 8월부터 11월까지 총 주문 횟수는 가파르게 증가하였다. (약 17 ~ 18달러) 평균 주문 금액은 9월부터 11월까지 가파르게 감소하였다. (약 6 ~ 7달러)

### 4-8. 소비 금액이 가장 높은 상위 3개 국가의 월 별, 요일 별, 시간 별 시계열 그래프 그리기.

![](https://velog.velcdn.com/images/tino-kim/post/1a45a34b-bd31-41bd-893e-039c5cd2f63d/image.png)
