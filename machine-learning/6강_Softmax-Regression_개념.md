## 6강

### Softmax Regression 개념

- logistic regression
- [![softmax1](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax1.png)]()
- X라는 입력에 W을 주면 Z가 나오고 sigmoid하면 Y^ 예측값(0/1)이 나옴
- 실제 Y hat은 ^ 으로 표현함 (동영상에서 잘못 표기됨)

#### Multinomial classification

- binary classificaton으로 multinomial classification 이 가능함
- [![softmax2](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax2.png)]()
- 여러 개의 classfication에서 나눠서 계산하기 번거로워서 하나로 합쳐서 계산함
- [![softmax3](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax3.png)]()
- 이제 입력값에 대해서 각각 classifier 를 생성해 따로 계산하지 않고 한번의 계산으로 각각 classifier에 해당하는 예측값을 구할 수 있게 되었다. 


#### Where is sigmoid ?

- 하나의 입력값 X에 대해 여러 개의 classifier를 염두해두고 계산했기 때문에 이제 값이 classifier 수만큼의 벡터로 나오게 됨
- 나온 Y hat을 0 ~ 1로 표현하기 위해서 sigmoid하는 것을 softmax라 함
- 어떤 classifier에 해당될지 1 이하의 값으로 표현하고 총합이 1이 되기 때문에 확률값이 된다.
- [![softmax4](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax4.png)]()
- one-hot encoding은 여러 개의 classifier에 분류될 확률을 무시하고 가장 높은 확률을 가진 classifier를 1로 표현하고 나머지는 0으로 표현하는 것


#### Cost function

- 예측값과 실제값의 차이를 확인해야 함
- Cross-Entropy
- [![softmax5](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax5.png)]()
- [![softmax6](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax6.png)]()
- L이 label 실제값, S가 예측값
- log를 한 값을 L element에 곱하고 나온 결과를 합함(시그마가 그 뜻임)

#### Logistic cost VS cross entropy

- 사실상 이전 cost 함수가 entropy임 (y가 1일 경우 logistic에서는 뒷 식이 무시되고 앞식만 남게되고 entropy에서는 y가 0일 경우는 곱해봤자 0이기에 1만 의미있다.)
- [![softmax7](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax7.png)]()
- [![softmax8](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax8.png)]()
 
### tensor flow Softmax Classification 구하기

- [![softmax9](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax9.png)]()
- [![softmax10](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax10.png)]()
- [![softmax11](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax11.png)]()
- class가 많아 y값이 정하기 어려워짐, 3 종류가 올 수 있고 표기는 one-hot encoding 방식

### tensor flow Fancy Softmax Classification 구하기

#### softmax_cross_entropy_with_logits

- 이전 entropy를 구하는 함수가 복잡함, 텐서플로우에서는 더 간단한 표현을 가진 함수를 제공해줌

```
logits = tf.matmul(X, W) + b
hypothesis = tf.nn.softmax(logits)

# 기존
cost = tf.reduce_mean(-tf.reduce_sum(Y*tf.log(hypothesis), axis=1))

# fancy
cost_i = tf.nn.softmax_cross_entropy_with_logits(logits=logits, labels=Y_one_hot)
cost = tf.reduce_mean(cost_i)
```

#### reshape

- 분류값을 one_hot으로 표현해야 할 때
- one_hot 함수는 rank를 +1로 만들어서 돌려줌, 1차원을 주면 2차원을 주게됨 -> 여전히 복잡함
- 예를 들어 [[0], [3]]에 nb_classes를 7을 줬다면 one_hot으로 표현하면 [[[1000000]], [[0001000]]] 이렇게 표현됨
- 이럴때 reshape 함수를 사용 하게됨
- 그러면 위의 예에서 [[1000000]], [[0001000]] 이렇게 표현됨, shape이 동일하게 (n, 7)이 됨
- [![softmax12](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax12.png)]()
- [![softmax13](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax13.png)]()
- [![softmax14](https://github.com/leeplay/study/blob/master/machine-learning/image/softmax14.png)]()
- flattern -> [[1], [0]] -> [1, 0]
