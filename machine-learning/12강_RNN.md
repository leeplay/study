## 12강

### NN의 꽃 RNN 이야기

#### Sequence data

- 연속된 데이터를 해석해야 할 때
- 문장을 이해하려면 이전 단어의 의미를 알아야 함
- 이전의 어떤 결과가 다음에 영향을 미쳐야 함
- [![rnn1](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn1.png)]()
- [![rnn2](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn2.png)]()

#### Recurrent Neural Network

- [![rnn3](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn3.png)]()


#### (Vanilla) Recurrent Neural Network

- 기본적인 RNN
- [![rnn4](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn4.png)]()
- [![rnn5](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn5.png)]()
- weight 값은 동일
- [![rnn6](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn6.png)]()
- [![rnn7](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn7.png)]()
- [![rnn8](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn8.png)]()
- 적용 분야
  - Language Modeling
  - Speech Recognition
  - Machine Translation
  - Conversation Modeling/Question Answering
  - Image/Video Captionaing
  - Image/Music/Dance Generation
- RNN을 어떻게 활용하냐에 따라 달라짐
- [![rnn9](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn9.png)]()
- 이 RNN은 기본모델이여서 실제로는 아래 두 가지 모델들을 더 많이 사용함
  - LSTM (Long Short Term Memory) 모델
  - GRU 모델


### RNN Basics

#### RNN in TensorFlow

- 이번에 아웃풋이 다음 Cell로 연결됨
- Cell을 생성함, lstm이나 GRU로 만들어도 됨
- 출력크기 = hidden_size

```
cell = tf.contrib.rnn.BasicRNNCell(num_units=hidden_size)
```


- 만든 cell과 입력 데이터를 넘겨줌
- 결과와 마지막 states 값을 돌려줌

```
outputs, _states = tf.nn.dynamic_rnn(cell, x_Data, dtype=tf.float32)
```

- [![rnn10](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn10.png)]()


#### One node: 4(input-dim) in 2(hidden_size)

- one hot encoding
- output-dimension은 hidden_size를 정의하기 나름
- [![rnn11](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn11.png)]()
- [![rnn12](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn12.png)]()
- 출력하면 값이 2개 나오는데 hidden size 를 2개로 줬기 때문이다. 초기값은 랜덤이다.
- [![rnn13](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn13.png)]()
- sequence_length : cell을 몇 번 펼칠 것인가
- [![rnn14](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn14.png)]()
- 학습을 시킬 때 문자열 하나씩 학습시키면 느리고 비효율적임, 효율적으로 만들고 싶으면 시퀀스를 여러 개 줘야함, 이 말을 batch_size라 함
- [![rnn15](https://github.com/leeplay/study/blob/master/machine-learning/image/rnn5.png)]()


