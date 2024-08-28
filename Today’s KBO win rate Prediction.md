# Today’s KBO win rate Prediction

S_AI training

---

# **-  subject**

- 당일날 공개되는 라인업을 입력했을때 어떤 팀이 이길지 승률 예측하는 모델

- input : 2023년 투수별 타자 상대 전적 데이터
- output : win rate

---

# -  Crawler

- MLB는 데이터셋을 api로 제공해주지만, KBO는 그렇지 않다.
- 스탯티즈 ( [https://statiz.sporki.com/](https://statiz.sporki.com/))의 데이터를 크롤링

---

# - Input Data

- 득점과 상관성이 높은 특성을 찾아 필요한 데이터 구분
- 그래프를 활용해 상관성이 높은 특성들의 우선순위 판단

ex) 투수별 타자의 타율(AVG)과 승리 확률 기여도(WPA)가 승률에 영향을 많이 미칠 것이다

---

# - Model

![model structure.png](Today%E2%80%99s%20KBO%20win%20rate%20Prediction%20036244eb85b34ff78205ac297aa0e806/model_structure.png)