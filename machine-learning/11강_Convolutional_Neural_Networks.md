## 11강

### Convolutional Neural Networks의 Conv 레이어 만들기

- 고양이 뇌분석에서 아이디어를 얻음
- 이미지 전체를 보고 인지하는게 아닌 부분을 나눠서 인지하는 것을 발견
- 이미지를 잘라서 입력받음
- CONV를 함
- RELU를 함
- POOL(샘플링)을 함
- 이 과정을 반복함
- FC(Fully Connected)를 구성함
- [![cnn1](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn1.png)]()
- [![cnn1](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn2.png)]()
- 32*32*3 size 에서 5*5*3 만큼 한 번에 읽어들임
- 칼라이기 때문에 *3(RGB)임
- [![cnn3](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn3.png)]()
- 하나의 값으로 표현함
- 순회하면서 동일 weight로 값을 읽어들임
- [![cnn4](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn4.png)]()
- [![cnn5](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn5.png)]()

- stride : 이동하는 칸 수, 1이면 1칸씩 이동
- stride 값이 커질 수록 이미지가 작아져 데이터 손실이 발생해 pad 를 사용
- pad는 입력과 아웃의 같은 사이즈를 만들기 위해 테두리를 사이즈만큼 증가시킴
  - 이미지가 작아지는 것을 방지효과와 모서리를 알려주는 효과가 있음
- [![cnn6](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn6.png)]()


#### Swiping the entire image

- [![cnn7](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn7.png)]()
- 필터를 적용해도 이미즈 사이즈가 줄지 않음
- 필터를 여러 개 생성해 적용함, 각각의 필터는 weight이 다름
- [![cnn8](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn8.png)]()
- weight의 수는 5*5*3*6 만큼 생성

#### Convolution layers

- [![cnn9](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn9.png)]()
- 고양이 실험에서 시작


### Max pooling 과 Full Network

#### Pooling

- 샘플링
- [![cnn10](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn10.png)]()

#### MAX Pooling

- [![cnn11](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn11.png)]()
- 필터에서 가장 큰 숫자를 출력

#### Fully connected layer

- [![cnn12](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn12.png)]()


### CNN 활용예

- 많은 응용이 가능함
- Lecun 교수가 처음 적용함
- [![cnn13](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn13.png)]()
- [![cnn14](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn14.png)]()
- [![cnn15](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn15.png)]()
- [![cnn16](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn16.png)]()
- [![cnn17](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn17.png)]()
- ResNet
  - layer가 너무 많아 fastforwad 개념을 사용
  - layer의 수는 많지만 건너뛰어 학습양은 줄어듬
  - 왜 잘 나오는지 이해를 못 하고 있음
- [![cnn18](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn18.png)]()
- 이미지가 아닌 텍스트 처리에 도전함
- [![cnn19](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn19.png)]()

### Tensor Flow CNN Basic

- [![cnn20](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn20.png)]()

#### Simple Convolution Layer 

- [![cnn21](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn21.png)]()
- [![cnn22](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn22.png)]()
- [![cnn23](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn23.png)]()
- [![cnn24](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn24.png)]()
- [![cnn25](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn25.png)]()
- [![cnn26](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn26.png)]()
- [![cnn27](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn27.png)]()


### MNIST 99% with CNN

- 이건 나중에 실습해보자

### CNN Class, Layers, Ensemble

#### Python Class

- 파이썬 개발팁 (객체지향 표현임)
- [![cnn28](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn28.png)]()
- 지원 api 패키지
- [![cnn29](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn29.png)]()
- [![cnn30](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn30.png)]()

#### Ensemble

- 새로운 데이터가 들어왔을 때 하나의 모델로 예측하는 것이 아닌 다양한 모든 모델에서 예측해보고 결과를 합쳤을 경우 성능이 더 좋게 나옴
- [![cnn31](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn31.png)]()
- [![cnn32](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn32.png)]()


#### Ensemble prediction

- [![cnn33](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn33.png)]()
- [![cnn34](https://github.com/leeplay/study/blob/master/machine-learning/image/cnn34.png)]()

