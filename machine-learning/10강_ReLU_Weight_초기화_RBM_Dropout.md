## 10강

### Sigmoid 보다 ReLU가 더 좋아


 - [![relu1](https://github.com/leeplay/study/blob/master/machine-learning/image/relu1.png)]()
 - [![relu2](https://github.com/leeplay/study/blob/master/machine-learning/image/relu2.png)]()
 - [![relu3](https://github.com/leeplay/study/blob/master/machine-learning/image/relu3.png)]()



#### Poor results

- deep/wide 하게 구성했는데 정확도가 떨어짐
 - [![relu4](https://github.com/leeplay/study/blob/master/machine-learning/image/relu4.png)]()

#### Backpropagation

- 레이어가 많아질수록 학습을 못 하게 됨
- Backpropagation + sigmoid 알고리즘 특성상 값이 커도 0 ~ 1사이로 나오게 되고 로컬미분을 곱하게 되면서 소수점 숫자가 계속 곱해져 숫자가 계속 작아지게 됨
- vanishing gradient : 등차 기울기가 사라지는 현상, 초기 부분에 경사도가 사라져 초기값이 최종값에 영향을 주지 않는다고 나오게됨
- NN winter2 : 2차 겨울이 오게됨 ㅠㅠ
 - [![relu5](https://github.com/leeplay/study/blob/master/machine-learning/image/relu5.png)]()

#### ReLU

- Hinton 이 제안함
- Rectified Linear Unit
- 0 보다 작으면 꺼버리고 0보다 크면 갈 때까지 가라
- [![relu6](https://github.com/leeplay/study/blob/master/machine-learning/image/relu6.png)]()
- [![relu7](https://github.com/leeplay/study/blob/master/machine-learning/image/relu7.png)]()
- 마지막 레이어는 sigmoid사용해야 함 0~1로 만들기 위해
- [![relu8](https://github.com/leeplay/study/blob/master/machine-learning/image/relu8.png)]()
- Leaky ReLU
  - 0보다 작은 값을 좀 더 주자
- ELU
- tanh : sigmoid 개선
- [![relu9](https://github.com/leeplay/study/blob/master/machine-learning/image/relu9.png)]()
- [![relu10](https://github.com/leeplay/study/blob/master/machine-learning/image/relu10.png)]()


### Weight 초기화 잘해보자

- vanishing gradient 의 하나의 원인이 됨
- w를 0으로 줄 때 문제
  - x가 0이 되버려서 전부 0이 되버림
  - 초기값을 0을 주면 안됨
- [![weight1](https://github.com/leeplay/study/blob/master/machine-learning/image/weight1.png)]()


#### Deep Belief Nets
- Restricted Boadman Machine(RBM)
- RBM을 사용해서 초기화시킴
- [![weight2](https://github.com/leeplay/study/blob/master/machine-learning/image/weight2.png)]()
- 앞뒤로만 연결되고 같은 레이어에선 연결 안됨
- [![weight3](https://github.com/leeplay/study/blob/master/machine-learning/image/weight3.png)]()
- Forward(encode)한 값을 구하고 그 값에서 Backward(decode) 했을 때 최저차가 나오도록 weight을 조절함
- [![weight4](https://github.com/leeplay/study/blob/master/machine-learning/image/weight4.png)]()
- 하나에서 계산하고 그 뒤로 계속 전파하면서 구함
- [![weight5](https://github.com/leeplay/study/blob/master/machine-learning/image/weight5.png)]()
- 초기값을 구하는 과정에서 어느정도 학습이 되서 Fine tunning이라고 부름
- 초기값을 구하고 label된 결과를 학습함


#### Xavier/He initialization

- RBM이 복잡하다.
- [![weight6](https://github.com/leeplay/study/blob/master/machine-learning/image/weight6.png)]()
- [![weight7](https://github.com/leeplay/study/blob/master/machine-learning/image/weight7.png)]()
- 초기값 설정은 계속 연구가 진행 중인 분야


### Dropout과 앙상블


#### overfitting 문제 

- [![dropout1](https://github.com/leeplay/study/blob/master/machine-learning/image/dropout1.png)]()
- 학습을 더하거나 복잡하게 구성할 수록 문제에 빠질 수 있음
- Regularization을 해야 하는데 그 방법 중 하나가 Dropout 임

#### Dropout

- [![dropout2](https://github.com/leeplay/study/blob/master/machine-learning/image/dropout2.png)]()
- 랜덤하게 노드를 강제로 끊음, 상당히 정확도가 올라감
- [![dropout3](https://github.com/leeplay/study/blob/master/machine-learning/image/dropout3.png)]()
- 학습할 땐 일부만 참여시키고 평가할 때는 전부 참여시킴

#### TensorFlow implementation

- [![dropout6](https://github.com/leeplay/study/blob/master/machine-learning/image/dropout6.png)]()
- [![dropout4](https://github.com/leeplay/study/blob/master/machine-learning/image/dropout4.png)]()

#### Ensemble

- 기계나 학습데이터가 많을 때 뉴럴 네트웍을 깊게 구성할 때 같은 모델(초기값 정도 다른)을 여러개 만들고 마지막에 합침
- 독립된 전문가 여러 명에게 문의하는 형식
- 성능(2~5%)을 더 높여줌
- 실제로 이 모델을 많이 사용함



### 레고처럼 네트웍 모듈을 마음껏 쌓아보자

- 어떤 형태의 네트웍을 구성할 수 있다.

#### Fast forward

- 2015년도에 He가 발견
- 시그널을 잡아서 앞으로 그래프 노드 2단 앞으로 이동시킴


#### Split & Merge

#### Recurrent network

- graph 이동을 앞 뿐이 아닌 옆으로도 이동함



### NN, ReLu, Xavier, Dropout and Adam (사용팁)


- [![dropout5](https://github.com/leeplay/study/blob/master/machine-learning/image/dropout5.png)]()
