# 구조 

1. 전력을 생산하는 가정(isgenerator=1)은 우선적으로 생산한 전력을 자신의 수요를 충족시키는데 사용한다
2. PV로 부터 전력 수요를 충족하고 남은 전력은 ESS에 저장한다. 
3. ESS controller 는 3가지 종류의 행동이 가능하다 
    + 한전으로 부터 전력을 공급받아야 하는 가정에 전력을 공급
    + SMP를 기반으로 한전에 전력을 판매
    + 전력을 Hold 
4. 한전으로 부터 전력을 구매하는 모든 집은 누진제도를 적용받는다.
    + 따라서 각 가정이 한전으로 부터 구매한 전력량을 누적하여 기록한다 (백터 형태로 기록)

## 누진세 계산 방법 

+ 3~5월은 <기타계절>에 해당된다.


||구간  |기본요금(원/호)|전력량 요금(원/kWh)|
|------|---|---|---|
|1     |200kWh이하        |910    |98.1|
|2     |201~400kWh이하    |1,600  |192.7|
|3     |400kWh초과        |7,300  |285.4|



+ ex) 월간 전력사용량이 250kWh일때


|      |항목  |요금(원)|설명|
|------|---|---|---|
|1     |기본요금         |1,600    |200kWh 초과 기본요금|
|2     |전력량요금       |29,255  |200kWh $\times$ 98.1(원) + 50kWh $\times$ 192.7(원)|
|3     |기후환경요금     |1,325  |5.3(원) $\times$ 250|
|4     |연료비조정       |-750  |-3(원) $\times$ 250|
|5     |전기요금        |**31,430**  |1+2+3+4|
|6     |부가세          |3,143  |5 $\times$ 0.1 (원단위 미만 반올림)|
|7     |전력기금        |1,162  |5 $\times$ 0.037 (10단위 이하 절사)|
|8     |청구요금        |**35,735**  |5+6+7|



# Notation


+ $X_t$ : t 시점의 ESS SOC
+ $C^{n}_t$ : t 시점의 n번째 집의 전력수요
+ $G^{n}_t$ : t 시점의 n번째 집의 전력 생산량
+ $diff_t$ : t 시점의 전력상황 벡터 ($C^{n}_t-G^{n}_t$)  
+ $AC_t$ : t 시점의 한전에서 지금까지 구매한 전력량 벡터 ($diff_t$를 매 스텝 누적, 누진세 적용여부에 사용) 
+ $P_t$ : t 시점의 각 집별 누진세 적용구간 값(전력단가) 벡터
+ $A^{prc}_t$ : t 시점의 전력공유가격 설정 액션
+ $A^{ess}_t$ : t 시점의 ess control 액션
+ $R_t$ : t 시점의 보상

# State

# Action

# Reward

+ $R_t$ = SMP판매금액 + 한전사용대비 절약금액 + 공유전력 판매금액 - (공유전력 구매금액) - (한전에서 구매한 전력비용) -(ESS 감가상각)

    + SMP 판매금액:
        + 한전에 판매할 전력량 * SMP 단가    
        + $X_t(1+A^{ess}_t) \times \textrm{SMP단가}$
    + 한전사용대비 절약금액: 
        + 공유전력 사용량 * (전력단가-공유전력판매단가)
    + 공유전력 판매금액:
        + 공유한 전력량 * 공유전력 판매단가
    + 공유전력 구매금액:
        + 공유한 전력량 * 공유전력 판매단가 
    + 한전에서 구매한 전력비용:
        + 누진세 전력단가 * 한전으로부터 구매한 전력량    
    + ESS 감가상각:
 
+ $R_t$ = SMP판매금액 + 한전사용대비 절약금액 - (한전에서 구매한 전력비용) -(ESS 감가상각)


# Transition


## $diff_t$ transition
 
+ Example 1)
    +  $diff_t$ = [10,-5,4,7,8,19,-6,10]
    +  '남는양 < 필요량' ,남는양을 할당하는 방법은 다음과 같음 
        +  if min(할당 받아야 할 양들) > $\frac{할당가능한 양}{할당받을 집의 수}$ == min(10,4,7,8,19,10) > $\frac{5}{6}$ 
        +  $\frac{5}{6}$ 만큼을 모든 할당받을 수 있는 모든 집에 할당. 그다음 -6에 대해서도 동일한 방법으로 할당   
    + 결과 = $diff_t$ = [8.17,0,2.17,5.17,6.17,17.17,0,8.17]
+ Example 2)
    + $diff_t$ = [10,-5,-3,-5,-2,9,-4,-4]
    + 남는양 :23 > 필요량:19 
    + $diff_t$ = [0,?,?,?,?,0,?,?] ?가 무엇이든 지 상관없음. 왜냐하면 남는 양을 무조건 ESS에 저장할 것이며, 이는 전력구매량 누적합에 영향을 미치지 않기 때문
+ Example 3)
    + $diff_t$ = [10,4,-7,-10,-3,9,-2,5]
    + Example 1에서 했던 방식대로 진행, 단 min(할당 받아야 할 양들) < $\frac{할당가능한 양}{할당받을 집의 수}$ 라면, min(할당 받아야 할 양들)값을 각 집에 할당하며, $\frac{할당가능한 양}{할당받을 집의 수}$ - min(할당 받아야 할 양들) 의 차이에 할당받은 집의 수 만큼을 곱하여 다시 할당되어야 할 양에 추가한다. 


==============================Step 1========================================================
+ 할당받아야할 리스트 = [10,4,9,5]
+ 할당되어야할 리스트 = [-7,-10,-3,-2]
+ min(10,4,9,5) >= $\frac{7}{4}$, $diff_t=[8.25,2.25,7.25,3.25]$



==============================Step 2========================================================
+ 할당받아야할 리스트 = [8.25,2.25,7.25,3.25]
+ 할당되어야할 리스트 = [-10,-3,-2]
+ min(8.25,2.25,7.25,3.25) < $\frac{10}{4}$, $diff_t=[6,5,1]$, $2.25 \times 4=1$을 할당되어야 할 값에 추가    

==============================Step 3========================================================
+ 할당받아야할 리스트 = [6,5,1]
+ 할당되어야할 리스트 = [-3,-2,-1]
+ min(6,5,1) >= $\frac{3}{3}$, $diff_t=[5,4,0]$
    
==============================Step 4========================================================
+ 할당받아야할 리스트 = [5,4]
+ 할당되어야할 리스트 = [-2,-1]
+ min(5,4) >= $\frac{2}{2}$, $diff_t=[4,3]$
    
==============================Step 5========================================================
+ 할당받아야할 리스트 = [4,3]
+ 할당되어야할 리스트 = [-1]
+ min(4,3) >= $\frac{1}{2}$, $diff_t=[3.5,2.5]$
    

## $diff_t$ 결정이후

1. if $\sum diff_t$의 합 <0 :
    
    + $X_{t}$ = $X_t+\sum diff_t$ 

2. else:
    
    + 전력구매 비용: $\sum diff_t \times P_t$ 
    + $AC_{t+1} = AC_t+ diff_t$
     

## ESS transition

+ if $A^{ess}_t  \subset [-1,0)$ :
    
    + SMP 판매
    + $X_{t+1} = X_t(1+A^{ess}_t)$
    
    
+ elif $A^{ess}_t  ==0$ :
   
    + 전력 Hold
    + $X_{t+1} = X_t$
    
+ elif $A^{ess}_t  \subset (0,1]$ :

    + 전력 부족 집에 전력 제공  
    + $X_{t+1} = X_t(1-A^{ess}_t)+f(X_t \times A^{ess}_t)$
    + $f(X_t \times A^{ess}_t)$은 $diff_t$를 transition하는 함수 


## 공유전력 판매금액 설정 

+ min range : a
+ max range : b
+ 가격 : $(a+b)A^{prc}_t$
