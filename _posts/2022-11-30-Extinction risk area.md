---

layout: single
title:  "[Tableau] 도시소멸지수 구하기-(1)"
categories : Tableau

---



## 1. 테이블 확인하기 

![01_table](../../images/2022-11-30-Extinction risk area/01_table.png)

![02_table2](../../images/2022-11-30-Extinction risk area/02_table2.png)

- 데이터를 확인해본 결과 231개 열과 18668개의 행으로 구성되어 있음을 알 수 있다.
- 각 열은 성별연령별로 각각 나누어져서 측정값으로 구분되어 있다.

## 2. 피벗 만들기

![03_pivot](../../images/2022-11-30-Extinction risk area/03_pivot.png)

![04_pivottable](../../images/2022-11-30-Extinction risk area/04_pivottable.png)

- 피벗함수를 사용할 열을 선택하여 피벗 테이블을 만들어 준다.
- 피벗필드명과 피벗 필드값 열이 생성됨을 볼 수 있다.

## 3. 피벗필드명 데이터 정규화

```sql
 # REGEXP_EXTRACT(문자열, 패턴) 
 # ex) REGEXP_EXTRACT(’abc 123’, ‘[a-z]+\s+(\d+)’) = ‘123’
 
 REGEXP_EXTRACT(문자열, 패턴)** 예 : REGEXP_EXTRACT(’abc 123’, ‘[a-z]+\s+(\d+)’) = ‘123’
```

```sql
# REGEXP_MATCH(문자열, 패턴) 
# ex) REGEXP_MATCH(’-([1234].[The.Market ])-’, ‘\[\s*(\w*\.)(\w*\s*\])’) = true

REGEXP_MATCH(문자열, 패턴)
예 : REGEXP_MATCH(’-([1234].[The.Market ])-’, ‘\[\s*(\w*\.)(\w*\s*\])’) = true
```

- 정규화 함수를 통하여 C_연령, C_성별 필드를 새로 생성하였다. 

![05_age sex](../../images/2022-11-30-Extinction risk area/05_age sex.png)

## 4. 가임여성과 고령자 구분하기

> 참고1. 도시소멸지수는 20~39세 여성을 65세 이상 고령자로 나눈 값임.
>
> 참고2. 도시소멸지수가 0.5 이하인 경우를 소멸위험 지역으로 분류

### 가임여성 구분

```sql
#C_여성수 : 25,858,862명
IF [C_성별] = '여성' THEN [피벗 필드 값]
END

# 가임여성 T/F 테이블
[C_성별]=='여성' AND
[C_연령]> 19 and [C_연령]<40

# 가임여성수 : 6,302,207명
IF [C_성별]=='여성' AND
   [C_연령]> 19 and [C_연령]<40 THEN [피벗 필드 값]
END

# C_가임여성수 비율
SUM([가임여성수]) / SUM([C_여성수])

# C_가임여성순위
RANK_UNIQUE([C_가임여성비율])

```

![06_women](../../images/2022-11-30-Extinction risk area/06_women.png)

### 고령자 구분

```sql
# 고령자 T/F 테이블
[C_연령] > 64

# 고령자수 
IF [C_연령] > 64 THEN [피벗 필드 값]
END

# C_고령자수비율
SUM([고령자수]) / SUM([피벗 필드 값])

# C_고령자순위
RANK_UNIQUE([C_고령자수비율])
```

![07_oldman](../../images/2022-11-30-Extinction risk area/07_oldman.png)

## 5. 도시소멸지수 구하기(시도, 시군구)

```sql
# 도시소멸지수 
SUM([가임여성수])/SUM([고령자수])

# T/F_소멸위험지역
[C_도시소멸지수] < 0.5
```

![08_Extinction risk area](../../images/2022-11-30-Extinction risk area/08_Extinction risk area.gif)



## 6. 대시보드화면

![09_dashboard](../../images/2022-11-30-Extinction risk area/09_dashboard-9769784.png)