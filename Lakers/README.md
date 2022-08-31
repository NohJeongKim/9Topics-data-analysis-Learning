# [DA-Learning] LA Lakers

## 1. 데이터 보기.

사용할 패키지를 모두 불러오고, 한글 깨짐을 방지하는 코드를 적어준다. (이 때 그래프에 마이너스를 표기해줄 수 있는 코드도 적어주면 아주 좋다. 설정을 위에서 끝내버리면 그래프 그릴 때에 통일하게 그려낼 수 있다.)

데이터를 깊은 복사를 이용해서 복사한 후에 그 데이터를 이용하자.

### 1-1. 칼럼 설명

- date : 경기 일자
- opponent : 대전 팀
- game_type : 홈 경기 vs 원정 경기
- time : 분, 초
- period : 쿼터 (한 쿼터 당 12분 씩, 동점인 경우 5 쿼터 진행)
- etype : 유형
- team : 팀 구분 (LAL : LA lakers 팀)
- player : 선수 명
- result : 결과
- points : 점수
- type : 세부 행동
- x, y : 상대편 뒤 골대 뒤에서 바라본 (x,y) 좌표이다. 골대의 위치를 표시해준다.

몇 개의 null 값이 있는지, 데이터 타입은 어떻게 되는지, 통계값은 어떻게 이루어져 있는지 확인해준다.

**[tip]** 범주형 변수와 연속형 변수를 분리하면 데이터 분석 시에 유용하다.

- 범주형 변수 : opponent, game_type, period, etype, team, player, result, type (빈도 계산 가능)
- 연속형 변수 : date, time, points, x, y (수치 계산 가능)

```
print("Opponent : ", df_copy["opponent"].unique())
print("Game_Type :", df_copy["game_type"].unique())
print("Period : ", df_copy["period"].unique())
print("Etype : ", df_copy["etype"].unique())
print("Team : ", df_copy["team"].unique())
print("Player : ", df_copy["player"].unique())
print("Result :", df_copy["result"].unique())
print("Type : ", df_copy["type"].unique())
# game_type, period, result 등으로 묶어서 나타낼 수 있다.
```

모든 칼럼에 대해서 어떤 데이터 값을 가지고 있는지 확인해보자.

```
print("총 데이터의 개수 : ", df_copy.shape[0]*df_copy.shape[1])
print(f"결측치의 개수 : {df_copy.isnull().sum().sum()}, 전체 데이터의 {((df_copy.isnull().sum().sum())/(df_copy.shape[0]*df_copy.shape[1]))*100:.2f}%를 차지한다.")
print("LAL와 경기한 상대편 팀의 개수 : ", df_copy["opponent"].nunique())
print("LAL의 행동 개수 : ", df_copy["etype"].nunique())
print("LAL의 세부 행동 개수 : ", df_copy["type"].nunique())
```

이런 식으로 데이터의 전체 개수와 결측치는 몇 퍼센트인지 등을 확인할 수 있다.

## 2. 데이터를 보고 질문하기.

### 2-1. 강사님 질문

- LAL의 홈 경기 비율 vs 원정 경기 비율 ?
- 경기에서 선수들이 제일 많이 한 행동 유형(etype)은?
- 이번 시즌에서의 LAL의 경기 결과는?
- LAL 선수들은 코트의 어디 위치에서 어떤 동작을 했는가?

### 2-2. 나의 질문

- LAL 기준 시간 별 etype 분포 vs OPP 기준 시간 별 etype 분포?
- 경기에서 팀별 제일 많이 한 행동 유형(etype)은?
- period 별 type 비율?

## 3. 데이터 정비하기.

### 3-1. 시간 타입의 데이터 정비하기.

새로운 datetime 칼럼을 만들어서 pd.to_datetime을 이용해서 데이터 타입을 변경시켰다.

```
df_copy["datetime"]=df_copy["date"].astype("str")+" "+df_copy["time"]
df_copy.head()

df_copy["datetime"]=pd.to_datetime(df_copy["datetime"], format="%Y%m%d %H:%M:%S")
df_copy.head()
```

원래 date 칼럼도 데이터 타입을 변경시켜주기.

```
df_copy["date"]=pd.to_datetime(df_copy["date"], format="%Y%m%d")
df_copy.head()
```

### 3-2. 결측치 처리하기.

**결측치를 어떻게 처리할 것인가?**

- player, result, type은 고유한 값이기 때문에 따로 NaN 값을 채워줄 필요가 없다.
- x, y는 좌표이기 때문에, 적절한 값을 대입시켜줘야 한다.

**[tip] x,y에 0을 채워 넣기 전에 실제 데이터에 0이 있는지 확인해야한다. 확인하지 않으면, 새로 채운 값이 실제 값을 가릴 수 있기 때문이다.**
<br/> ➡ 확인해 본 결과, x column 에는 0이 있기 때문에, 다른 값을 채워야 한다. (그래서 나는 x에 -1을 y에 0을 대입하였다.)

```
print(df_copy.loc[df_copy["x"]==-1, "x"].index.tolist()) # x=0인 부분은 존재한다.
print(df_copy.loc[df_copy["y"]==0, "y"].index.tolist()) # y=0인 부분은 없다.
# 나는 결측치에 x에는 -1, y에는 0을 대입시켜주었다.
```

## 4. EDA & Visualization

### 4-1. LAL의 홈 경기 비율 vs 원정 경기 비율 ?

일단 date와 game_type의 칼럼이 필요하기 때문에, 그 칼럼들을 다 가져오고 집계 함수를 이용해서 date, game_type 별 합계를 구해주고 값을 세어주면 홈 경기와 원정 경기의 개수가 나온다. 그 값을 이용해서 그래프를 그려주면 된다.

```
df_sum=df_copy[["date", "game_type"]].groupby(["date", "game_type"]).sum().reset_index()

_=df_sum['game_type'].value_counts().plot.pie(colors=["tomato", "yellow"], shadow=True, explode=[0, 0.1], autopct="%.2f%%")
_=plt.savefig("./../images_Lakers/practice.png")
```

![](https://velog.velcdn.com/images/tino-kim/post/ef93b074-0f62-4783-ab60-ced5f877ac36/image.png)

**[주의] size, count VS sum 의 차이를 알아야 한다. 서로 사용하는 경우가 다르다. 위와 같은 경우는 sum을 이용해야 올바르게 값이 나온다. count나 size를 이용하는 경우에는 home, away가 정말 몇 번 나왔는지를 체크하기 때문에 내가 원하는 결과가 나오지 않는다.**

위와 같은 방식으로 groupby를 이용해도 되고, drop_duplicates를 이용해서 중복되는 부분을 모두 제거하고 데이터를 이용해도 된다.

```
df_drop=df_copy.drop_duplicates(subset=["date"], keep="first") # 앞에 있는 것을 살린다.
df_drop # 중복되는 것들이 사라진다.
```

왼쪽에는 파이 그래프를 그리고, 오른쪽에는 바 그래프를 그리자.

```
fig,ax=plt.subplots(1, 2, figsize=(18,8))
color_list=sns.color_palette("Set2", df_copy["game_type"].nunique())

_=df_drop["game_type"].value_counts().plot.pie(ax=ax[0], autopct="%.2f%%", explode=[0, 0.1], shadow=True, colors=color_list)
_=ax[0].set_title("LAL의 홈 경기 비율 vs 원정 경기 비율", size=15)
_=ax[0].set_ylabel("")

_=sns.countplot(data=df_drop, x="game_type", ax=ax[1], palette="husl")
_=ax[1].set_title("LAL의 홈 경기 비율 vs 원정 경기 비율", size=15)

fig.savefig("./../images_Lakers/LAL의 홈 경기 비율 vs 원정 경기 비율.png", dpi=200, facecolor="#F6F7FB")
```

![](https://velog.velcdn.com/images/tino-kim/post/4883fe4c-61ee-4402-96e5-957f2e754907/image.png)

∴ 홈 경기와 원정 경기의 비율은 50%로 각각 동일함을 알 수 있다.

### 4-2. 경기에서 선수들이 제일 많이 한 행동 유형(etype)은?

단순하게 표기하고 싶으면 countplot을 이용하면 된다.
<br/>만약에 period 별로 etype의 개수를 세고 싶으면 ➡ period, etype groupby 실행하기 ➡ reset_index와 as_index=False 이용하기. ➡ pivot table로 변경하기. ➡ Stacked Bar Graph 그리기.

#### 경기에서 선수들이 가장 많이 한 행동 유형 그래프

```
fig, ax=plt.subplots(1, 1, figsize=(9,8))

_=sns.countplot(data=df_copy, y="etype", palette="husl")
_=ax.set_title("경기에서 선수들이 제일 많이 한 행동 유형(etype)은?", size=15)

fig.savefig("./../images_Lakers/경기에서 선수들이 제일 많이 한 행동 유형(etype).png", dpi=200, facecolor="#F6F7FB")
```

![](https://velog.velcdn.com/images/tino-kim/post/d7e1c547-628b-44ad-863c-5e9bbe1b44e0/image.png)

∴ shot이 가장 많고, jump ball과 violation이 가장 적은 편이다.

#### period 별 경기에서 선수들이 가장 많이 한 행동 유형 그래프

stacked=True로 표기하고 싶으면, pivot table로 변경해서 bar를 이용해서 stacked를 표현해주기.

```
df2=df_copy.groupby(["etype", "period"]).size().reset_index() # 위와 아래는 같은 코드이다.
df2.columns=["etype", "period", "total_count"]
df2

_=df_huePer.plot.bar(stacked=True, figsize=(18,8)) # index : x, columns : stacked
_=plt.legend(fontsize=15)
_=plt.xticks(fontsize=13, rotation=45)
_=plt.xlabel("etypes", fontsize=15)
_=plt.title("경기에서 선수들이 제일 많이 한 행동 유형(etype)은?", fontsize=15)

plt.savefig("./../images_Lakers/경기에서 선수들이 제일 많이 한 행동 유형(etype)_bar.png", dpi=200, facecolor="#F6F7FB")
# stacked 처리한 것이 훨씬 보기 편리하다.
# period 마다 크게 차이는 없다.
```

![](https://velog.velcdn.com/images/tino-kim/post/43eca476-8793-45d8-b31a-522813a8cd8e/image.png)

∴ period 별 etypes의 비율은 크게 변동 없다. period와 etypes는 크게 관계가 없는 편이다.

hue를 이용해서 period 별 바 그래프를 그릴 수 있다.

![](https://velog.velcdn.com/images/tino-kim/post/874c6418-1c37-4bb7-8eb7-e7639c8a52d6/image.png)

seaborn의 barplot을 이용해서 그려낼 수도 있다. 이 경우는 따로 전처리 없이 바로 df_copy를 이용해서 그릴 수 있다.

![](https://velog.velcdn.com/images/tino-kim/post/6a811ad0-311b-4976-aac3-82221012e033/image.png)

### 4-3. 이번 시즌에서의 LAL의 경기 결과는?

- 시계열을 통해서 경기 결과를 표현하기.
- 시간 순으로 나타낸 그래프를 의미한다. 시간이 있으면 시계열 그래프를 그려보는 것이 도움이 된다.

일단 result가 made인 데이터만 불러오기. (isin을 이용해서 불러오기.) ➡ date, team, points 칼럼을 가져와서 집계한 뒤에 date, time 별로 points가 몇 개인지 개수를 세어주기. ➡ LAL 팀과 OPP 팀으로 나눠주기. ➡ merge를 이용해서 두 개의 데이터 프레임을 붙여주기. ➡ LAL 팀이 승리한 경우와 패배한 경우를 나눠주기.

이렇게 전처리를 하는 경우, LAL 팀이 얼마나 이겼는지 파악이 가능하고, 날짜 별로 팀의 상황이 어떻게 흘러가는지 파악이 가능하다.

![](https://velog.velcdn.com/images/tino-kim/post/ede5d38e-938a-47a1-9cf0-ddd0e07781e2/image.png)

LAL 팀과 OPP 팀으로 나눴었고, LAL 팀 그 속에서 이긴 날과 진 날을 나누었기 때문에 이렇게 그래프로 표현된다.

∴ 시계열을 통해서 LAL가 언제 이기고, 졌는지를 알 수 있는 그래프이다. 또한, 승리한 횟수가 패배한 횟수보다 훨씬 많다.

### 4-4. LAL 선수들은 코트의 어디 위치에서 어떤 동작을 했는가?

#### 어디 위치에서 어떤 동작을 했는지의 분포 (result 별, points 별)

LAL 팀만 확인하면 된다.

```
# 이번에는 LA Lakers 팀만 볼 것이다.
lal=df_copy.loc[df_copy["team"]=="LAL"]
lal
```

그리고 분포를 알아보기 위해서, scatterplot과 kdeplot을 이용하자.

```
fig, ax=plt.subplots(1, 2, figsize=(18,8))

# 득점 성공 / 실패 분포
_=sns.scatterplot(data=lal, x="x", y="y", hue="result", alpha=0.3, ax=ax[0], palette="Paired")
_=sns.kdeplot(data=lal, x="x", y="y", hue="result", ax=ax[0], palette="Paired")

# 득점 점수 별 분포
_=sns.scatterplot(data=lal, x="x", y="y", hue="points", alpha=0.3, ax=ax[1], palette="Set2")
_=sns.kdeplot(data=lal, x="x", y="y", hue="points", ax=ax[1], palette="Set2")
```

![](https://velog.velcdn.com/images/tino-kim/post/eeabe107-f841-4651-8f0d-76bb45ff210f/image.png)

∴ 골대 근처로 슛이 많이 들어가기 때문에, 그 쪽에 분포가 많이 쏠려있다. 그리고 points가 클 수록 멀리 던진 슛을 의미하고, 그 슛들의 분포도 알 수 있었다.

#### 가장 많이 한 세부사항 상위 10개의 분포

```
# 가장 많이 한 세부 행동 10개 고르기.
lal_10=lal.loc[lal["type"].isin(lal["type"].value_counts().head(10).index), :]
lal_10 # 세부 행동이 너무 많기 때문에, 10개만 그려보기.
```

세부 행동 상위 10가지를 고른 뒤에, 그 데이터를 그래프로 그려보았다.

```
sns.set_style("whitegrid")
fig, ax=plt.subplots(1, 1, figsize=(5,7))

_=sns.scatterplot(data=lal_10, x="x", y="y", hue="type", alpha=1, palette="Set3")
_=plt.legend(fontsize=12, bbox_to_anchor=(1.03, 1), title="Type")

fig.savefig("./../images_Lakers/type 상위 10개 산점도.png", dpi=200, facecolor="#F6F7FB")
```

![](./../images_Lakers/type%20%EC%83%81%EC%9C%84%2010%EA%B0%9C%20%EC%82%B0%EC%A0%90%EB%8F%84.png)

∴ type의 상위 10개의 분포를 알아보았다. jump가 가장 많았고, 멀리서는 (3점 슛을 던지는 기술인 것 같다.) 3pt가 많았다. 골대 근처에서는 shooting, driving layup, lay up, hook 등을 이용하였다.

### 4-5. LAL 기준 시간 별 etype 분포 vs OPP 기준 시간 별 etype 분포?

일단 result가 made인 경우만 가져오기. ➡ LAL 팀과 OPP 팀으로 나눠주기. ➡ date 별 etype과 points의 sum과 max 구해주기. ➡ Multi-Index 이므로 droplevel을 이용해서 Single-Index로 변경하기. ➡ 불필요한 칼럼 제거하기.

이런 식으로 전처리를 진행하면, etype와 points의 개수와 최댓값이 나오게 된다.

이를 이용해서 그래프를 그리면 된다.

```
fig, ax=plt.subplots(2, 1, figsize=(20,12), constrained_layout=True)
fig.suptitle("가장 많이 사용한 기술인 shot의 합계 포인트와 최대 포인트의 분포", fontsize=20)

twin_ax=ax[0].twinx() # x축을 공유한다는 의미이다.
twin_ax.set_ylim(0, 10)
_=sns.lineplot(data=lal_drop, x=lal_drop.index, y="total_points", ax=ax[0], color="red")
_=sns.lineplot(data=lal_drop, x=lal_drop.index, y="max_point", ax=twin_ax, color="green")
# _=ax[0].set_xticklabels(ax[0].get_xticklabels(), rotation=90)
_=ax[0].set_ylabel("total_points",fontsize=14)
_=twin_ax.set_ylabel("max_point",fontsize=14)
_=ax[0].legend(["total_points"], loc="upper right", fontsize=15)
_=twin_ax.legend(["max_point"], loc="lower right", fontsize=15)
_=ax[0].set_title("LAL 팀이 가장 많이 사용한 기술인 shot의 합계 포인트와 최대 포인트의 분포", fontsize=15)

ax_twin=ax[1].twinx()
ax_twin.set_ylim(0, 10)
_=sns.lineplot(data=opp_drop, x=opp_drop.index, y="total_points", ax=ax[1], color="yellowgreen")
_=sns.lineplot(data=opp_drop, x=opp_drop.index, y="max_point", ax=ax_twin)
_=ax[1].legend(["total_points"], loc="upper right", fontsize=15)
_=ax_twin.legend(["max_point"], loc="lower right", fontsize=15)
_=ax[1].set_ylabel("total_points",fontsize=14)
_=ax_twin.set_ylabel("max_point",fontsize=14)
_=ax[1].set_title("LAL 아닌 팀이 가장 많이 사용한 기술인 shot의 합계 포인트와 최대 포인트의 분포", fontsize=15)

fig.savefig("./../images_Lakers/etype 전체 포인트와 최대 포인트.png", dpi=200, facecolor="#F6F7FB")
```

![](https://velog.velcdn.com/images/tino-kim/post/eaac3909-bd56-4ad1-a975-82830ed31eab/image.png)

∴ 모든 팀 (LA Lakers와 LAL가 아닌 팀)이 가장 많이 이용한 기술은 shot 이다. 그리고 두 팀의 shot 기술은 모두 최대 포인트는 3점이다. 위의 시계열 그래프와 비교하였을 때, shot 기술의 total points가 높을 수록, 그 해당 팀이 이겼을 확률이 크다.

### 4-6. 경기에서 팀별 제일 많이 한 행동 유형(etype)은?

#### 경기에서 팀별 상위 10개 행동 유형 분포

가장 팀별로 제일 많이 한 행동 유형 상위 10개를 집계해보니, LAL이 모두 차지하였다. 행동 유형을 order로 설정하고 그래프를 그려 보니, 상위 10개만 그래프를 큰 순서부터 제대로 그릴 수 있게 되었다.

```
df_max=df_copy.groupby(["team", "type"]).size().reset_index().sort_values(by=0, ascending=False).head(10)
df_max.columns=["team", "type", "total_max"]
df_max
```

전처리가 끝난 뒤에 그래프를 그려보자.

```
fig, ax=plt.subplots(1, 1, figsize=(12,8))
_=sns.barplot(data=df_max, x="type", y="total_max", order=order, palette="rainbow_r") # 가장 많은 것부터 작은 순으로 그래프 그림.
_=ax.set_title("전체적으로 기술을 가장 많이 사용한 팀과 기술 타입", fontsize=15)
_=plt.xlabel("team : LAL / type", fontsize=12)
_=plt.xticks(rotation=30)
_=ax.set_ylabel("team : LAL / total_Max Count", fontsize=12)

fig.savefig("./../images_Lakers/total max type head 10.png", dpi=200, facecolor="#F6F7FB")
```

![](https://velog.velcdn.com/images/tino-kim/post/209f335d-478d-44fc-b91d-0237b34dbe1f/image.png)

∴ 상위 10개의 데이터를 살펴 보면, 전체적으로 기술을 가장 많이 사용한 팀은 LA LAkers이고, def를 가장 많이 이용하였고, turnaround jump를 가장 적게 사용하였다.

#### 팀 별 types의 유형 분포

![](https://velog.velcdn.com/images/tino-kim/post/e3585e01-e879-475a-9c3d-11721bfbd549/image.png)

∴ LAL 팀은 turnaround jump를 압도적으로 다른 팀 보다 많이 이용한다. 다른 팀들도 몇 개의 팀 빼고는 turnaround jump를 많이 이용한다.

#### 팀 별 가장 많이 한 기술 타입들의 분포

```
fig, ax=plt.subplots(1, 1, figsize=(15,8))
_=df_type_pivot.plot.bar(ax=ax, stacked=True)
_=ax.set_ylabel("팀 별 가장 많이 한 기술 타입들의 횟수")
_=plt.legend(fontsize=13)
_=plt.title("팀 별 가장 많이 한 기술 타입들의 분포?", fontsize=15)

fig.savefig("./../images_Lakers/팀 별 가장 많이 한 기술 타입들의 분포.png", dpi=200, facecolor="#F6F7FB")
```

![](https://velog.velcdn.com/images/tino-kim/post/8c119387-d109-4e0e-978f-006992e8085d/image.png)

∴ 전반적으로 turnaround jump 기술이 가장 많게 분포되어 있다. 각 팀들이 많이 사용하는 기술이다. 위의 내용을 뒷받침해주는 근거가 된다.

### 4-7. period 별 type 비율?

```
fig, ax=plt.subplots(1, 1, figsize=(20,10))
_=df3.plot.bar(stacked=True, ax=ax)
_=plt.legend(fontsize=15, title="Period")
_=plt.title("period 별 type 분포?", fontsize=15)

_=plt.savefig("./../images_Lakers/period 별 type 분포.png", dpi=200, facecolor="#F6F7FB")
```

![](https://velog.velcdn.com/images/tino-kim/post/b76481db-10e8-4f0f-816a-dac449b0fe24/image.png)

∴ 전체적으로 많이 이용한 세부 기술을 알아 보니, 각 period 별로는 세부 기술의 분포와는 관련 없다. 하지만, LAL 팀의 승률과 세부 기술을 살펴 보니, LAL 팀이 다른 팀들보다 turnaround jump를 잘 구현하기 때문에 승률이 높음을 알 수 있다.

## 5. Review

- 총 데이터의 개수 : 450112

- 결측치의 개수 : 76625, 전체 데이터의 17.02%를 차지한다.

- LAL와 경기한 상대편 팀의 개수 : 29

- LAL의 행동 개수 : 10

- LAL의 세부 행동 개수 : 73

- 홈 경기와 원정 경기의 비율 = 1 : 1

- 경기에서 선수들이 가장 많이 한 기술은 shot이 가장 많고, jump ball과 violation이 가장 적은 편이다. 그리고 period와 기술 간의 관계는 거의 없다.

- LA Lakers 팀이 LAL가 아닌 팀 보다 승리한 횟수가 패배한 횟수보다 훨씬 많다.

- 골대 근처로 슛이 많이 들어가기 때문에, 그 쪽에 분포가 많이 쏠려있다. 그리고 points가 클 수록 멀리 던진 슛을 의미하고, 그 슛들의 분포도 알 수 있었다. 또한 ype의 상위 10개의 분포를 알아보았다. jump가 가장 많았고, 멀리서는 (3점 슛을 던지는 기술인 것 같다.) 3pt가 많았다. 골대 근처에서는 shooting, driving layup, lay up, hook 등을 이용하였다.

- 모든 팀 (LA Lakers와 LAL가 아닌 팀)이 가장 많이 이용한 기술은 shot 이다. 그리고 두 팀의 shot 기술은 모두 최대 포인트는 3점이다. 위의 시계열 그래프와 비교하였을 때, shot 기술의 total points가 높을 수록, 그 해당 팀이 이겼을 확률이 크다.

- 상위 10개의 데이터를 살펴 보면, 전체적으로 기술을 가장 많이 사용한 팀은 LA LAkers이고, def를 가장 많이 이용하였고, turnaround jump를 가장 적게 사용하였다.

- 팀 별로 많이 이용한 세부 기술을 알아 보니, 전반적으로 turnaround jump 기술이 가장 많게 분포되어 있다. 각 팀들이 많이 사용하는 기술이다.

- 전체적으로 많이 이용한 세부 기술을 알아 보니, 각 period 별로는 세부 기술의 분포와는 관련 없다. 하지만, LAL 팀의 승률과 세부 기술을 살펴 보니, LAL 팀이 다른 팀들보다 def, jump 등의 기술을 더 잘 사용하기 때문에, 승률이 높음을 알 수 있다.
