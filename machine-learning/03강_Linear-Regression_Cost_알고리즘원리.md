## 3강 

### Linear Regression의 cost최소화 알고리즘의 원리 설명

#### What cost(W) looks like ?

- x = [1, 2, 3]
- y = [1, 2, 3]

- W = 1
 - ((1 * 1 - 1)^2 + (1 * 2 - )^2 + (1 * 3 - 3)^2)/3 = cost(0)
- W = 0
 - ((0 * 1 - 1)^2 + (0 * 2 - )^2 + (0 * 3 - 3)^2)/3 = cost(4.67)
- W = 2
 - ((2 * 1 - 1)^2 + (2 * 2 - )^2 + (2 * 3 - 3)^2)/3 = cost(4.67)


- [![cost_minimize1](https://github.com/leeplay/study/blob/master/machine-learning/image/cost_minimize1.png)]()

- 이걸 찾는 알고리즘이 Gradient(경사) descent(내려가다) algorithm

- [![gradient1](https://github.com/leeplay/study/blob/master/machine-learning/image/gradient1.png)]()
- a는 그래프에서 기울기를 구할 때 구한 미분 값이 + 라면 w를 양/음 둘 중 하나로 이동시키기 위한 공식 표현
- [![gradient2](https://github.com/leeplay/study/blob/master/machine-learning/image/gradient2.png)]()
 - 미분은 온라인 사이트에서 구해줌
- [![gradient3](https://github.com/leeplay/study/blob/master/machine-learning/image/gradient3.png)]()

#### Convex function

- 3차원 차트에서 아래와 같은 상황이면 알고리즘이 잘 동작하지 않음
- [![convex1](https://github.com/leeplay/study/blob/master/machine-learning/image/convex1.png)]()
- 아래와 같은 상황이 되야 gradient 알고리즘이 제대로 동작하고 항상 값을 찾음
- cost function을 설계할 때 아래와 같은 상황이 되어야 함
- 아래 상황을 convex function 이라 함
- [![convex2](https://github.com/leeplay/study/blob/master/machine-learning/image/convex2.png)]()

### Linear Regression의 cost최소화 TensorFlow 구현

```
learning_Rate = 0.1
gradient = tf.reduce_mean((W * X - Y) * X)
descent = W - learning_Rate * gradient
update = W - W.assign(descent)
```

- 위 코드가 아래 한 줄로 표현됨

```
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1)
train = optimizer.minimize(cost)
```

- 부록

```
# gradient 값을 임의로 조절할 수도 있음
gvs = optimizer.compute_gradients(cost)
apply_gradients = optimaier.apply_gradients(gvs)
```
