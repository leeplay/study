## 5강

### Logistic Classification의 가설 함수 정의

- 뉴럴 네트웍과 딥러닝을 이해하기 위한 주요 개념
- binary classification을 분류하는데 유용한 알고리즘
- classification
 - spam, feed, stock, image 종양분석
- linear regression의 문제점은 x값이 엄청나게 증가할 경우 바이너리 표현을 하기 어렵다. 중간값이 계속 증가할 것이기 때문이다. 
- 이런 이유로 잘못된 예측을 할 확률이 증가함, 또한 바이너리에서는 y값을 0~1로 표현하고 싶은데 학습된 데이터로 w와 b값을 구했는데 x값이 엄청나게 커진 경우 y값 또한 증가한다. 
- 그러다 이런 함수를 찾음
 - 가설(H(x))를 z로 표현했을 때 어떤 함수(g)를 사용하면 0과 1로 표현할 수 있을까 ?
 - g(z) = 1 / (1 + e^-z)
 - 분자가 1이여서 분모가 커지면 0에 수렴하고 분모가 작아지면 1에 수렴한다.
 - -^ 승을 하기에 항상 소수점으로 나오며 거기에 1을 더하기 때문에 최소 1을 보장함, 그래서 최대 1이 나오고 분모가 커지면 0으로 수렴함
 - 이 함수를 sigmoid 혹은 logistic function이라 부름
 - z값이 커질수록(소수점의 제곱이라 계속 작아짐) g함수의 값은 1로 수렴하고 작아질수록 0으로 수렴함
 - [![logistic1](https://github.com/leeplay/study/blob/master/machine-learning/image/logistic1.png)]()
 - [![logistic2](https://github.com/leeplay/study/blob/master/machine-learning/image/logistic2.png)]()


### Logistic Regression의 cost 함수 설명

- H(x) = Wx + b 에서는 cost를 구할 때 제곱을 하기 때문에 포물선 형태를 그리고 어디서 시작하든 최소점에 도달 할 수 있음
- sigmod 함수로 0 ~ 1 사이로 y값이 나오게 했는데 이 함수로 cost 를 그려보면 직선이 아닌 울퉁불퉁한 선에 제곱을 했기 때문에 울퉁불퉁한 선이 그려짐
- 새로운 hyphthesis에서는 이제 linear한 선이 아님, linear한 선이 아닌 상태에서 제곱을 하게되니 울퉁불퉁한 그래프가 됨, 어디서 시작하든 최소점 쉽게 구할 수가 없으며 최소점이 달라짐
- local minimum과 global minimum 개념이 등장함, global minimum이 실제 구해야할 값인데 local minimum 만 구하고 멈출 확률이 아주 높음, 그래서 이런 그래프는 사용 못 하게 되고 cost 함수를 바꿔야 함
- [![logistic3](https://github.com/leeplay/study/blob/master/machine-learning/image/logistic3.png)]()

#### new cost function for logistic

- [![logistic4](https://github.com/leeplay/study/blob/master/machine-learning/image/logistic4.png)]()
- log 함수를 사용함, 로그함수의 특징을 잘알아야 함
- 계단형태의 그래프에 상반되는 것이  로그함수이다.  
- 그래서 기본적으로 로그함수를 사용하고 로그함수의 형태가 포물선이여서 적합하다. 
- 예측에 실패하면 cost가 무한대로 커짐
- 바이너리이기때문에 y값은 0/1 중 하나임
- y=1 일 때 cost 함수 -log(H(x)), 가설(x)이 1일 경우 cost는 0이 됨, 가설이 0일 경우 cost는 무한대, 0<a<1인 경우 무한대로 감
- y=0 일 때 cost 함수 -log(1- H(x)), 위와 반대
- [![logistic5](https://github.com/leeplay/study/blob/master/machine-learning/image/logistic5.png)]()
- 위 함수로 그래프의 모향이 포물선 형태로 만들었기 때문에 미분이 가능함

### tensor flow로 구현하기

