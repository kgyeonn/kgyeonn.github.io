---
layout : single
title : "[Tableau] 미국주식데이터 시각화하기-(2)"
categories : Tableau
---



# 3. 데이터 시각화(Tableau)

![07_tableau](../../images/2022-11-30-yfinance_tableau/07_tableau.png)

## 3.1 증감현황 나타내기

> 1.   특정달을 선택하여 날짜별로 증가한 종목 수를 달력에 보여주세요
>
> 2.   특정달을 선택하여 날짜별로 감소한 종목 수를 달력에 보여주세요
>
>  3.   특정 날의 증감 현황을 보여주세요
>        (단, 달력에서 선택한 날짜가 나타나도록 보여주세요)
>
>  4.   해당 날의 현황을 자세하게 테이블로 보여주세요
>       (단, 해당 일의 증감 현황 중 증가 또는 감소를 선택하였을 때 테이블이 변화하도록 보여주세요)

- **증감율 구하기**

```sql
[Close]-[Open] #종가-시가
```

- **T/F 증감율**

```sql
[증감율]>0 #True/False
```

### 3.1.1 특정달을 선택하여 날짜별로 증가한 종목수를 달력에 보여주세요.

```sql
# 증가종목 : 증감율이 T인경우 ticker의 개수
count(IF [T/F 증감율] THEN [Ticker] END) 
```

![08_increase](../../images/2022-11-30-yfinance_tableau/08_increase.png)

### 3.1.2 특정달을 선택하여 날짜별로 감소한 종목수를 달력에 보여주세요.

```sql
# 감소종목 : 증감율이 F인경우 ticker의 개수
count(IF not([T/F 증감율]) THEN [Ticker] END)
```

![08_decrease](../../images/2022-11-30-yfinance_tableau/08_decrease.png)

### 3.1.3 특정날의 증감현황 (단, 달력에서 선택한 날짜가 나타나도록 보여주세요)

- **증감현황 캘린더 시트**

![10_calander](../../images/2022-11-30-yfinance_tableau/10_calander.png)

- **증감현황 시트**

![11_status](../../images/2022-11-30-yfinance_tableau/11_status.png)




💡 증감현황캘린더 시트의 9월 20일 클릭 시 9월 20일의 증감현황시트가 나타나야함

### * 시트로 이동 동작 (워크시트→ 동작)

![12_sheetmove](../../images/2022-11-30-yfinance_tableau/12_sheetmove.png)

- 동작추가 클릭 → 시트로 이동

- 원본 시트를 증감현황캘린더 대상시트를 증감현황 동작 실행조건은 선택으로 선택

### *  필터 동작 (워크시트→ 동작)

![13_filtermove](../../images/2022-11-30-yfinance_tableau/13_filtermove.png)

- 증감현황캘린더에 걸려있는 날짜 필터를 선택하면 해당 날짜 클릭시 해당 일자 증감현황이 보여짐

![14_calanderstatus](../../images/2022-11-30-yfinance_tableau/14_calanderstatus.gif)

### 3.1.4 대시보드로 보여주기

![15_dashboard](../../images/2022-11-30-yfinance_tableau/15_dashboard.png)

![16_dashboard](../../images/2022-11-30-yfinance_tableau/16_dashboard.gif)



----

## 3.2 회사별 주식 현황을 캔들차트로 시각화 하기

### 3.2.1 Apple Inc. 캔들차트 그리기

```sql
#C_시가종가변경
[Close]-[Open]

#C_고가저가변경
[Low]-[High] #high-low를 빼면 양수만 나오는 경향이 있음 
```

- **SUM(High)와 SUM(Open) 이중축 만들기 (간트차트로 변경)**
  - 왜 고가와 시가를 행에 두었는가? 고가 저가 축 중심을 high로 잡고, 시가 종가 축을 open으로 잡음
  - 축편집에서 0포함 제거해주기!!

![17_gantchart](../../images/2022-11-30-yfinance_tableau/17_gantchart.png)

- **SUM(High)**
  - C_고가저가변경 측정값을 크기로 둠
  - C_고가저가변경 측정값을 세부정보로 둠
  - Low 측정값을 세부정보로 둠

![18_sum(high)](../../images/2022-11-30-yfinance_tableau/18_sum(high).png)

- **SUM(Open)**
  - C_시가종가변경 측정값을 크기로 둠
  - C_시가조가변경 측정값을 세부정보로 둠
  - Close 측정값을 세부 정보로 둠


![19_sum(open)](../../images/2022-11-30-yfinance_tableau/19_sum(open).png)

- **상승은 빨간색, 하강은 파란색 색상 표시**

```sql
#C_색상 : C_시가조가변경 측정값이 0 이상인 경우 상승 미만인 경우 하강 
IF [C_시가종가변경] > 0 THEN "상승" ELSE "하강" END
```

![20_color](../../images/2022-11-30-yfinance_tableau/20_color.png)

### 3.3.2 대시보드 그리기

- 회사선택
- 연도/분기별 선택 가능

![21_candlechart](../../images/2022-11-30-yfinance_tableau/21_candlechart.gif)