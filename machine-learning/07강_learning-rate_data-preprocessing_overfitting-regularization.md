## 7강 학습 

### learning-rate, data-preprocessing, Overfitting-regularization

#### rate

- cost function을 정의하고 최소화하기 위한 알고리즘에 learning-rate이란 알파값을 주게됨

- Large learning rate: overshooting
  - step이 너무 크면 cost를 구할 수가 없음, cost 함수를 벗어날 수도 있음, cost값이 줄어들지 않고 이상하게 나온다면 learning_rate를 의심해야함
  - 이럴 경우 learning_rate 수치를 낮춰야함

- Small learning rate: takes too long, stops at local minimum
  - 너무 오래 걸림, 이럴 경우 learning_rate 수치를 올려야함
  - 위의 두 경우는 정답이 없으므로 관찰하면서 적절히 조절해야 함

#### data preprocessing

- learning rate을 잘 잡았는데 데이터가 이상하게 나올때
- 2차원으로 그래프를 그렸을 경우
- [![appendix1](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix1.png)]()
- 왜곡된 데이터 등고선이 그려질 수 있음
- [![appendix2](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix2.png)]()
- 데이터를 nomalize 해야함
- [![appendix3](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix3.png)]()
- zero-centered : 데이터의 중심을 0에 위치하게
- normalized data : 일정값 범위 안으로 들어오게
- [![appendix4](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix4.png)]()


#### overfitting

- 머신러닝의 가장 큰 문제
- 학습데이터를 통해 학습을 하게 되는데 학습데이터를 기준으로 맞는 모델을 만들기 때문에 실제 리얼과 맞지 않는 경우
- 아래 모델의 경우 왼쪽이 실제 세상과 더 적합하지만 학습 데이터 기준으로 보면 우측이 더 적합하다. 
- [![appendix5](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix5.png)]()

#### overfitting solution

- 많은 학습 데이터
- 중복된 feature를 제거
- Regularization

#### Regularization

- 선을 구부리지 말고 일정하게 만들자. (weight이 큰 값을 가지지 말자)
- cost 함수 뒤에 term을 추가
- [![appendix6](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix6.png)]()


### Training Validation and test sets

- 학습 데이터를 구분해 학습/검증/테스트 용으로 구분
- [![appendix7](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix7.png)]()

#### online learning

- 지속적으로 학습하는 방법
- MINIST Dataset : 유명한 데이터 사례, 사람들이 흘려쓰는 우편번호를 컴퓨터가 인지하기 위해 시간이 흐를수록 데이터의 변형을 입력

### Tensor flow 

- [![appendix8](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix8.png)]()
- 잘못된 learning rate 예
- [![appendix9](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix9.png)]()
- [![appendix10](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix10.png)]()

#### NaN

- non-normalized inputs
- [![appendix11](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix11.png)]()

### MNIST Dataset

- tensorflow에 기본 테스트 데이터가 있음
- [![appendix12](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix12.png)]()
- [![appendix13](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix13.png)]()
- 학습데이터가 커져서 epoch/batch를 사용해야 함
- n epoch : 전체 data set을 n번 읽어들인 것
- batch size : 한번에 읽어들일 데이터 row
- [![appendix14](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix14.png)]()
- [![appendix15](https://github.com/leeplay/study/blob/master/machine-learning/image/appendix15.png)]()
