## 9강


### 미분 기본

- back propagation이 미분개념임
- 델타 x를 아주 작은 값(0)으로 보낼 때 순간변화율은 얼마인가 ? 
- x가 f에 끼치는 영향이 얼마인가 ? 
- [![xor4](https://github.com/leeplay/study/blob/master/machine-learning/image/xor4.png)]()
- 상수함수는 미분하면 항상 0임
- x는 미분하면 1
- 2x는 미분하면 2

#### Partial derivative

- 관심있는 미분값을 제외하고 나머지는 상수로 보는 개념, 편미분
- [![xor5](https://github.com/leeplay/study/blob/master/machine-learning/image/xor5.png)]()
- chain rule g(x)가 f에 미치는 영향
- 복합 함수의 미분은 각각 미분해 곱한다.
- f(g(x)) -> df/dx -> df/dg*dg/dx (체인룰)
- [![xor6](https://github.com/leeplay/study/blob/master/machine-learning/image/xor6.png)]()



### XOR 문제 딥러닝으로 풀기

#### XOR

- one logistic regression으로는 풀 수 없다.
- 여러 개의 logistic을 사용하면 풀 수는 있지만 학습이 불가능하다.
- [![xor1](https://github.com/leeplay/study/blob/master/machine-learning/image/xor1.png)]()
- 여러 개의 sigmoid를 하나로 연결함, 이게 neural newtork 이다.

#### NN

- 여러 개의 sigmoid를 하나로 표현할 수 있다고 배웠다. 
- [![xor2](https://github.com/leeplay/study/blob/master/machine-learning/image/xor2.png)]()
- [![xor3](https://github.com/leeplay/study/blob/master/machine-learning/image/xor3.png)]()
- 이런 W, b를 가지면 문제를 해결을 할 수 는 있는데 이런 조합이 너무 많다. 가장 작은 regulation을 구할 수 있을까 ?


### 딥네트워 학습시키기(backpropagation)

- 어떤 점에서 미분값이 필요함
- 노드가 많아지는 NN에서 복잡해짐
- 이 두 가지가 너무 많아 계산하기 너무 어려움
  - 최종적인 결과가 Y일 때 X1이 Y에 끼치는 영향을 알아야 W을 조절함
  - 각각 layer가 Y에 끼치는 영향을 알아야 함
- 위의 문제를 Backpropagation 알고리즘으로 해결함


#### Back propagation (chain rule))

- 미분값은 비례해서 영향을 미침, 미분값이 5면 x값이 1이 바뀌면 5의 영향을 미침
- [![xor7](https://github.com/leeplay/study/blob/master/machine-learning/image/xor7.png)]()
- [![xor8](https://github.com/leeplay/study/blob/master/machine-learning/image/xor8.png)]()
- 미분값을 다 알 수 있기 때문에 tensor flow는 다 그래프로 가지는 것
- [![xor9](https://github.com/leeplay/study/blob/master/machine-learning/image/xor9.png)]()

### Neural Net for XOR

- [![xor10](https://github.com/leeplay/study/blob/master/machine-learning/image/xor10.png)]()
- [![xor11](https://github.com/leeplay/study/blob/master/machine-learning/image/xor11.png)]()
- 위의 방식으로 xor는 실패함

#### Neural Net

- xor 문제를 풀기 위해선 sigmoid를 연속으로 구성해줘야 함
- [![xor12](https://github.com/leeplay/study/blob/master/machine-learning/image/xor12.png)]()
- [![xor13](https://github.com/leeplay/study/blob/master/machine-learning/image/xor13.png)]()

#### Wide NN for XOR

- [![xor14](https://github.com/leeplay/study/blob/master/machine-learning/image/xor14.png)]()
- 작은 값은 더 작아지고 큰 값은 더 커짐, 모델일 잘 나왔다는 걸 증명함

#### Deep NN for XOR

- [![xor15](https://github.com/leeplay/study/blob/master/machine-learning/image/xor15.png)]()


### Tensorboard p Neural Net for XOR


- 그래프 시각화 툴
- 기존에 루프가 돌 때마다 화면에 출력했는데 그럴 필요가 없음

- [![xor16](https://github.com/leeplay/study/blob/master/machine-learning/image/xor16.png)]()
- 값이 하나일 때
- [![xor17](https://github.com/leeplay/study/blob/master/machine-learning/image/xor17.png)]()
- 값이 여러 개 일 때 
- [![xor18](https://github.com/leeplay/study/blob/master/machine-learning/image/xor18.png)]()
- 그래프 보고 싶을 때 
- [![xor19](https://github.com/leeplay/study/blob/master/machine-learning/image/xor19.png)]()
- 실행화면
- [![xor20](https://github.com/leeplay/study/blob/master/machine-learning/image/xor20.png)]()
- [![xor21](https://github.com/leeplay/study/blob/master/machine-learning/image/xor21.png)]()
- 비교화면
- [![xor22](https://github.com/leeplay/study/blob/master/machine-learning/image/xor22.png)]()
