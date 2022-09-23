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
2022년 9월 27일 현재 다음과 같습니다.

## 결과 예측 예시
먼저 확률 계산기입니다.  
```{python}
def expect_result(differ):
    differ = differ / 400
    draw = 2 * np.exp(-(differ * 4) ** 2 / (2 * np.exp(1) ** 2)) / ((2 * np.pi) ** 0.5 * np.exp(1))
    lose = 1 / (10 ** differ + 1) * (1 - draw)
    win = (1 - 1 / (10 ** differ + 1)) * (1 - draw)
    point = 3 * win + 1 * draw
    return win, draw, lose, point
```
400점의 차이가 나면 확률은 다음과 같이 계산됩니다.
```{python}
expect_result(400)
>> (0.819, 0.099, 0.082, 2.55)
```
순서대로 강팀의 승리 확률, 무승부 확률, 약팀의 승리 확률, 강팀의 기대 승점입니다.
