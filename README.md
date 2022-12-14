# FIFA World Cup Qatar 2022 결과 예측 by Elo Rating
Elo Rating을 통해 `FIFA World Cup Qatar 2022`의 결과를 예측하였습니다.

# Introduction
## Elo Rating 소개
[킹무위키](https://namu.wiki/w/Elo%20%EB%A0%88%EC%9D%B4%ED%8C%85)를 참조하는 것이 가장 좋습니다.  
간단한 수식을 통해 계산할 수 있으며, 가장 중요한 점은
* 이기면 점수가 오르고, 지면 점수가 내려간다.
* 매 경기마다  팀의 점수 상승폭과 진 팀의 점수 하락폭은 같다.
* 강팀이 약팀에게 이기면 점수가 적게 오른다.
* 약팀이 강팀에게 이기면 점수가 크게 오른다.

정도로 요약할 수 있습니다.  
마찬가지로 간단한 수식을 통해 경기 전에 각 팀의 승리 확률을 예측할 수 있습니다.  
다만, 본래 승패가 결정나는 종목에 대해서 쓰이는 기법이므로 축구처럼 무승부가 많은 종목에는 적합하지 않습니다.  
하지만 FIFA도 2018년부터는 Elo Rating을 변형하여 FIFA Ranking을 산정하고 있습니다.

## 축구로의 접목
이러한 이유로 무승부 확률을 계산하기 위해서는 따로 공식을 제작해야 하는데, [Xiong et al. (2016)](https://ieeexplore.ieee.org/document/7810980)에서는 한 가지 방법을 제시하고 있습니다.  
공식은 다음과 같습니다.  
$$P(\text{draw}) = \cfrac{1}{\sqrt{2\pi}e}\exp{(-\cfrac{(dr/200)^2}{2e^2})}$$  
(단, $dr$은 홈 팀의 Elo Rating에서 원정 팀의 Elo Rating을 뺀 값)  
다만 해당 논문에서는 위 식에 의해 계산된 무승부 값을 반으로 나누어, 기존 두 플레이어의 승리 확률에서 각각 빼주는 방식을 취하고 있습니다.  
이 경우, 두 플레이어의 Elo Rating이 크게 차이나는 경우 약팀의 승리 확률이 0보다 작아질 수 있습니다.  
따라서 본 연구에서는 1에서 무승부 확률을 뺀 값을 '승부가 날 확률'로 정리하고, 그 값에 대해서 기존의 승리 확률을 곱하여 사용하겠습니다.  
또한 해당 논문에서 이용한 정규분포 곡선을 이용하되, 상수들은 새로 계산하였습니다.  
계산에는 Stochastic Gradient Descent를 이용했고, 자세한 계산 과정은 제 졸업논문에 기술되어 있지만 놀랍게도 제 졸업논문은 아마 어디에도 공개되지 않을 것 같습니다...  
최종적으로 무승부 확률은 다음과 같습니다.
$$P(\text{draw}) = \cfrac{c}{\sqrt{2\pi}\sigma}\exp{(-\cfrac{(dr/400)^2}{2\sigma^2})}, \quad c =  0.527, \sigma = 0.693$$  
홈 팀에게는 홈 어드밴티지가 부여되는데, 이 역시 SGD를 통해 계산했고 약 86.4점 정도의 Elo Rating을 부여받게 됩니다.   
SGD 과정에서는 1872년 이후에 벌어진 모든 A매치 경기 데이터가 사용되었고, 이는 [martj42의 Github](https://github.com/martj42/international_results)에서 구할 수 있었습니다. 감사합니다.  

# 예측 결과
## Data
[World Football Elo Ratings](eloratings.net)에서 제공하는 Elo Rating을 사용하였습니다.  
2022년 9월 28일 현재 월드컵 진출국 Elo Rating은 다음과 같습니다.  
| 국가 | Elo Rating |
|:----:|:---:|
| 브라질 | 2169 |
| 아르헨티나 | 2140 |
| 스페인 | 2045 |
| 네덜란드 | 2040 |
| 벨기에 | 2025 |
| 프랑스 | 2005 |
| 포르투갈 | 2004 |
| 덴마크 | 1971 |
| 독일 | 1960 |
| 우루과이 | 1936 |
| 스위스 | 1929 |
| 크로아티아 | 1922 |
| 잉글랜드 | 1920 |
| 세르비아 | 1893 |
| 에콰도르 | 1840 |
| 멕시코 | 1821 |
| 이란 | 1817 |
| 폴란드 | 1809 |
| 미국 | 1798 |
| 일본 | 1798 |
| 웨일스 | 1790 |
| 대한민국 | 1783 |
| 캐나다 | 1770 |
| 모로코 | 1754 |
| 코스타리카 | 1737 |
| 카타르 | 1650 + 86.4 |
| 호주 | 1719 |
| 튀니지 | 1687 |
| 세네갈 | 1687 |
| 사우디 | 1632 |
| 카메룬 | 1613 |
| 가나 | 1541 |

* 카타르는 홈 어드밴티지 적용

## 결과 예측 예시
먼저 확률 계산기입니다.  
우리 팀이 상대 팀보다 Elo Rating이 400점 높다면 확률은 다음과 같이 계산됩니다.
```{python}
expect_result(400)
>> (0.812, 0.107, 0.081, 2.54)
```
순서대로 승리 확률, 무승부 확률, 패배 확률, 기대 승점입니다.

## 조별 기대 승점 순위
각 조마다 각 팀이 3경기동안 얻을 수 있는 기대 승점을 계산해보았습니다.
### A조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 네덜란드 | 6.71 |
| 에콰도르 | 4.29 |
| 카타르 | 3.08 |
| 세네갈 | 2.55 |
### B조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 잉글랜드 | 5.15 |
| 이란 | 3.88 |
| 미국 | 3.66 |
| 웨일스 | 3.57 |
### C조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 아르헨티나 | 7.45 |
| 멕시코 | 3.83 |
| 폴란드 | 3.70 |
| 사우디 | 1.86 |
### D조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 프랑스 | 6.03 |
| 덴마크 | 5.63 |
| 호주 | 2.68 |
| 튀니지 | 2.34 |
### E조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 스페인 | 6.05 |
| 독일 | 5.01 |
| 일본 | 3.09 |
| 코스타리카 | 2.42 |
### F조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 벨기에 | 6.02 |
| 크로아티아 | 4.75 |
| 캐나다 | 2.96 |
| 모로코 | 2.78 |
### G조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 브라질 | 7.13 |
| 스위스 | 4.47 |
| 세르비아 | 4.07 |
| 카메룬 | 1.25 |
### H조
| 국가 | 기대 승점 |
|:------:|:-----------:|
| 포르투갈 | 6.27 |
| 우루과이 | 5.51 |
| 대한민국 | 3.76 |
| 가나 | 1.30 |

## 대한민국 경기 결과 예측
대한민국의 경기 결과 예측은 다음과 같습니다.
```
vs 우루과이
승리 확률 : 21.67%  
무승부 확률 : 26.05%  
패배 확률 : 52.28%   
기대 승점 : 0.91점  

vs 가나
승리 확률 : 63.51%  
무승부 확률 : 20.72%  
패배 확률 : 15.77%  
기대 승점 : 2.11점  

vs 포르투갈
승리 확률 : 17.06%  
무승부 확률 : 22.08%  
패배 확률 : 60.87%  
기대 승점 : 0.73점  
```

## 대한민국의 16강 진출 확률 시뮬레이션
이런 식으로 매 경기 시뮬레이션을 돌리면 됩니다.  
각 조에서 열리는 6경기에 대해 모두 시뮬레이션을 돌리고, 결과를 종합하면 되는데, 문제가 있습니다.  
Elo Rating으로는 득점과 실점까지 예측할 수 없다는 것입니다. (마음 먹으면 가능하겠지만..)  
그래서 '승점이 동률이면 Elo Rating이 높은 팀을 높은 순위로 배치한다'는 룰을 그냥 만들었습니다.  
(분명 대한민국에게는 불리한 룰입니다.)  
아무튼 100,000번 시뮬레이션 해본 결과는 다음과 같습니다.  
```
1위 확률 : 9.66%  
2위 확률 : 21.65%  
3위 확률 : 51.74%  
4위 확률 : 16.95%  
```

## 전체 토너먼트 계산
토너먼트에서는 승부를 가리지 못한 경우 승부차기가 진행됩니다. (120분간 진행되니 무승부 확률이 조금 더 낮아지겠지만 패스..)  
승부차기에 돌입하면 승률은 50%가 되는 룰을 도입하였습니다.  
각 국가별로 10,000번 시뮬레이션 해본 결과는 다음과 같습니다.
### 대한민국
16강 진출 확률 : 31.44%  
8강 진출 확률 : 7.48%  
4강 진출 확률 : 2.45%  
결승 진출 확률 : 0.63%  
우승 확률 : 0.15%  
### 일본
16강 진출 확률 : 27.14%  
8강 진출 확률 : 10.07%  
4강 진출 확률 : 3.06%  
결승 진출 확률 : 1.09%  
우승 확률 : 0.23%  
### 아르헨티나
16강 진출 확률 : 95.56%  
8강 진출 확률 : 68.33%  
4강 진출 확률 : 47.97%  
결승 진출 확률 : 28.82%  
우승 확률 : 18.55%  
### 브라질
16강 진출 확률 : 93.28%  
8강 진출 확률 : 69.19%  
4강 진출 확률 : 48.17%  
결승 진출 확률 : 31.49%  
우승 확률 : 21.88%  
### 프랑스
16강 진출 확률 : 84.35%  
8강 진출 확률 : 48.47%  
4강 진출 확률 : 29.93%  
결승 진출 확률 : 15.49%  
우승 확률 : 7.06% 
### 포르투갈
16강 진출 확률 : 87.04%  
8강 진출 확률 : 44.09%  
4강 진출 확률 : 24.11%  
결승 진출 확률 : 13.67%  
우승 확률 : 6.50%  
### 우루과이
16강 진출 확률 : 76.21%  
8강 진출 확률 : 30.08%  
4강 진출 확률 : 14.10%  
결승 진출 확률 : 6.39%  
우승 확률 : 2.48%  
### 잉글랜드
16강 진출 확률 : 72.11%  
8강 진출 확률 : 38.71%  
4강 진출 확률 : 17.01%  
결승 진출 확률 : 7.33%  
우승 확률 : 2.68%  

## 우승 확률 계산
50000번 시뮬레이션 해본 결과는 다음과 같습니다.
| 국가 | 우승 확률 (%) |
|:----:|:---:|
| 브라질 | 21.660 |
| 아르헨티나 | 18.980 |
| 네덜란드 | 8.482 |
| 스페인 | 7.996 |
| 프랑스 | 6.954 |
| 벨기에 | 6.872 |
| 포르투갈 | 5.850 |
| 덴마크 | 4.528 |
| 독일 | 3.422 |
| 잉글랜드 | 2.634 |
| 우루과이 | 2.622 |
| 스위스 | 2.160 |
| 크로아티아 | 2.046 |
| 세르비아 | 1.158 |
| 에콰도르 | 1.082 |
| 멕시코 | 0.758 |
| 이란 | 0.558 |
| 폴란드 | 0.480 |
| 미국 | 0.406 |
| 웨일스 | 0.294 |
| 일본 | 0.198 |
| 대한민국 | 0.170 |
| 캐나다 | 0.168 |
| 카타르 | 0.154 |
| 모로코 | 0.102 |
| 코스타리카 | 0.080 |
| 호주 | 0.072 |
| 세네갈 | 0.058 |
| 튀니지 | 0.034 |
| 사우디 | 0.016 |
| 카메룬 | 0.004 |
| 가나 | 0.002 |