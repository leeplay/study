## 2강 

### Linear Regression와 Hypothesis와 cost 설명

#### Regression

- Linear한 Hypothesis(가설)을 세움
- 세상의 많은 모델이 Linear하기 때문이다. (e.g. 학생이 공부를 많이 할수록 시험을 잘본다. 연습을 많이 할수록 능숙해진다.)
- 어떤 Linear한 선이 우리 상황에 맞을까 찾는 것이 학습을 하는 것
- 이 2차원 선을 수학공식으로 표현하면 H(x) = Wx + b, 가설은 가중치 * x에 bias를 더한 것
- 여러 가설을 세운 후 2차원 공식으로 표현함, 그러면 이 중에 어떤 가설이 제일 좋은지 알아야 함
- 실제 데이터와 가설이 나타내는 데이터와 거리를 표현함, 가까울 수록 좋음
- 이것을 linear regression에서는 cost(lost) functino이라 부름, 우리가 세운 가설과 실제 데이터가 얼마나 다른지 증명함

#### Cost function

- 트레이닝 데이터가 얼마나 적합한지 확인
- (H(x) - y)^
- 가장 작은 cost를 구하는 것

[![cost_function1](https://github.com/leeplay/study/blob/master/machine-learning/image/cost_function1.png)]()
[![cost_function2](https://github.com/leeplay/study/blob/master/machine-learning/image/cost_function2.png)]()
[![cost_function3](https://github.com/leeplay/study/blob/master/machine-learning/image/cost_function3.png)]()


### TensorFlow로 간단한 linear regression을 구현


#### Build graph using TF operations


```
x_train = [1, 2, 3]
y_train = [1, 2, 3]


# W, b를 알 수 없으니 tf가 인식할 수 있도록 rank 값을 인자로 랜덤하게 줌
W = tf.Variable(tf.random_normal([1]), name='weight')
b = tf.Variable(tf.random_normal([1]), name='bias')

hypothesis = x_train * W + b

#cost/loss function
#reduce_mean은 tensor들의 평균을 구하는 함수 (공식의 시그마가 평균임), t = [1, 2, 3, 4] = 2.5
cost = tf.reduce_mean(tf.square(hypothesis - y_train))


#GradientDesent, cost의 최소값을 구함, learning_rate는 미분값을 구하기 위해서 W를 얼만큼 단계별로 이동시킬지 정하는 알파값
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)
train = optimizer.minimize(cost)
```

#### Run/update graph and get results

```
# Launch the graph in a session
sess = tf.Session()

# Initializes global variable in the graph
sess.run(tf.global.variables_initializer())

# Fit the line, 2001번 반복, 20번당 1번씩 출력 
for step in range(2001):  
	sess.run(train)
	if step * 20 = 0:
	  print(step, sess.run(cost), sess.run(W), sess.run(b))
```

- train을 실행한다는 말은 train -> cost -> hypotyesis -> wx+b를 구하는 과정


#### Placeholders

- 값을 정하지 않고 세션을 실행할 때 넘겨줌
- graph를 미리 만들어놓고 값을 줄 수 있음, 재사용

```
X = tf.placeholder(tf.float32, shape=[None])
Y = tf.placeholder(tf.float32, shape=[None])

...

for step in range(2001):  
	cost_val, W_val, b_val, _= sess.run(cost, W, b, train), feed_dict={X: [1,2,3], Y: [1,2,3]]})
	
	if step * 20 = 0:
	  print(step, cost_Val, W_val, b_val)
```


```
# hypothesis를 구한 후 결과를 예측해봄

print(sess.run(hypothesis, feed_dict={X: [5]}))
print(sess.run(hypothesis, feed_dict={X: [2.5]}))
print(sess.run(hypothesis, feed_dict={X: [1.5, 3.5]}))
```
