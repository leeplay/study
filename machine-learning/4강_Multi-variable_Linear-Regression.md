## 4강 

### multi-variable linear regression

- 입력 값이 복수라면 ?
- [![multi_cost_function1](https://github.com/leeplay/study/blob/master/machine-learning/image/multi_cost_function1.png)]()
- 수식이 너무 복잡해짐
- maxtrix 등장

#### Matrix multiplication

- [![matrix1](https://github.com/leeplay/study/blob/master/machine-learning/image/matrix1.png)]()
- [![matrix2](https://github.com/leeplay/study/blob/master/machine-learning/image/matrix2.png)]()
- [5, 3]
 - 5는 instance 개수
 - 3은 element의 개수
- X[5, 3] * W[3, 1] = H(X)[5, 1]


#### WX vs XW

- 이론 적으로는 H(x) = Wx + b
- 구현적으로는 H(X) = XW
- 데이터의 형태는 대부분 메트릭이기 때문이다.
- 수학적의미로는 두 개가 동일하다.

### multi-variable linear regression TensorFlow에서 구현하기 


- learning_rate에서 1e-5 이런 표현이 많이 쓰이는데 scientific notation 표현법이며 0.00001을 의미함, 양수면 0보다 커지면서 제곱임
- 코드가 너무 복잡해짐
- [![matrix3](https://github.com/leeplay/study/blob/master/machine-learning/image/matrix3.png)]()
- Matrix로 표현해야 함
 - shape을 조심해야 함
 - Node은 n개의 데이터를 의미함
- [![matrix4](https://github.com/leeplay/study/blob/master/machine-learning/image/matrix4.png)]()

#### Loading Data from file

- 데이터가 너무 많아지니 소스코드로 표현 불가함
- 파이썬의 slice를 사용함
- [![loading_data1](https://github.com/leeplay/study/blob/master/machine-learning/image/loading_data1.png)]()

#### Queue Runners

- 파일을 메모리에 올리는게 부담일 때
- [![loadig_data2](https://github.com/leeplay/study/blob/master/machine-learning/image/loadig_data2.png)]()
- batch를 통해서 구현
- [![loadig_data3](https://github.com/leeplay/study/blob/master/machine-learning/image/loadig_data3.png)]()
- shuffle_batch (batch의 순서를 조절하고 싶을때)
