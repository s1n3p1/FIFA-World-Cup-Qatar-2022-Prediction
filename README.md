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
$$P(\text{draw}) = \cfrac{1}{\sqrt{2\pi}e}\exp{(-\cfrac{(\frac{dr}{200})^2}{2e^2})}$$  
(단, $dr$은 홈 팀의 Elo Rating에서 원정 팀의 Elo Rating을 뺀 값)  