## 8장 머신러닝: MLlib

### 8.1 MLlib 개요


[![ml-image01](https://github.com/leeplay/study/blob/master/image/ml-image01.png)]()

#### 통계처리, 머신러닝이란 ?

- 통계처리 : 특정 데이터로부터 수학적 기법을 이용하여 그 성질을 끄집어내는 처리
- 머신러닝 : 과거 데이터의 정량적 관계를 파악함으로써 미래 데이터의 대한 예측을 하는 처리

#### ml step 

- 1단계 : Raw 데이터 정련 및 로딩 (Train/Validation/Test Data)
- 2단계 : 특징 추출 (Reaturization => Feature Vectors)
- 3단계 : 학습 (Training => Model)
- 4단계 : 평가 및 튜닝 (Model Evaluation => Best Model)
- 5단계 : 모델 저장
- 6단계 : 서비스

[![ml-image01-1](https://github.com/leeplay/study/blob/master/image/ml-image01-1.png)]()

### 8.2 MLlib의 기초와 제공 알고리즘

- spark 공식 사이트서 다양한 api와 알고리즘을 확인해보세요

[![ml-image02](https://github.com/leeplay/study/blob/master/image/ml-image02.png)]()


#### 기본 요소

- MLlib의 데이터 타입 (2.0 기준)
  - Vector
    - sparse, dense
  - LabeledPoint
    - supervised learning 에서 사용
  - Rating
    - rating of a product by a user
  - Matrix
    - Local Matrix
    - Distributed Matrix
      - RowMatrix
      - IndexedRowMatrix
      - CoordinateMatrix
      - BlockMatrix
  - Various Models 


#### PMML(Predictive Model Markup Language)

- 데이터마이닝그룹이라는 표준화 단체가 규정한 XML로 기술하는 학습 모델 언어
- 스파크 이외의 툴에서 이 모델을 활용한 기능을 쉽게 구현

- KMeansModel ClusteringModel
- LinearRegressionModel RegressionModel (functionName="regression")
- RidgeRegressionModel  RegressionModel (functionName="regression")
- LassoModel  RegressionModel (functionName="regression")
- SVMModel  RegressionModel (functionName="classification" normalizationMethod="none")
- Binary LogisticRegressionModel  RegressionModel (functionName="classification" normalizationMethod="logit")

```
# Kmeans-Sample 

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<PMML xmlns="http://www.dmg.org/PMML-4_2" version="4.2">
    <Header description="k-means clustering">
        <Application name="Apache Spark MLlib" version="2.3.0"/>
        <Timestamp>2018-06-04T21:04:47</Timestamp>
    </Header>
    <DataDictionary numberOfFields="3">
        <DataField name="field_0" optype="continuous" dataType="double"/>
        <DataField name="field_1" optype="continuous" dataType="double"/>
        <DataField name="field_2" optype="continuous" dataType="double"/>
    </DataDictionary>
    <ClusteringModel modelName="k-means" functionName="clustering" modelClass="centerBased" numberOfClusters="2">
        <MiningSchema>
            <MiningField name="field_0" usageType="active"/>
            <MiningField name="field_1" usageType="active"/>
            <MiningField name="field_2" usageType="active"/>
        </MiningSchema>
        <ComparisonMeasure kind="distance">
            <squaredEuclidean/>
        </ComparisonMeasure>
        <ClusteringField field="field_0" compareFunction="absDiff"/>
        <ClusteringField field="field_1" compareFunction="absDiff"/>
        <ClusteringField field="field_2" compareFunction="absDiff"/>
        <Cluster name="cluster_0">
            <Array n="3" type="real">0.1 0.1 0.1</Array>
        </Cluster>
        <Cluster name="cluster_1">
            <Array n="3" type="real">9.099999999999998 9.099999999999998 9.099999999999998</Array>
        </Cluster>
    </ClusteringModel>
</PMML>
```


### 8.3 MLlib 입문

#### K-means 개요

```
[root@CentOS7-14 mllib]# cat kmeans_data.txt
0.0 0.0 0.0
0.1 0.1 0.1
0.2 0.2 0.2
9.0 9.0 9.0
9.1 9.1 9.1
9.2 9.2 9.2
```

```
import org.apache.spark.mllib.clustering.KMeans
import org.apache.spark.mllib.linalg.Vectors

// Load and parse the data
val data = sc.textFile("/kikang/spark-2.3.0-bin-hadoop2.7/data/mllib/kmeans_data.txt")
val parsedData = data.map(s => Vectors.dense(s.split(' ').map(_.toDouble))).cache()

// Cluster the data into two classes using KMeans
val numClusters = 2
val numIterations = 20
val clusters = KMeans.train(parsedData, numClusters, numIterations)

// Export to PMML to a String in PMML format
println(s"PMML Model:\n ${clusters.toPMML}")

// Export the model to a local file in PMML format
clusters.toPMML("/tmp/kmeans.xml")

// Export the model to a directory on a distributed file system in PMML format
clusters.toPMML(sc, "/tmp/kmeans")

// Export the model to the OutputStream in PMML format
clusters.toPMML(System.out)

```

### 8.6 spark.ml 패키지의 ML 파이프라인

#### 왜 별도로 존재하는가 ?

- 여전히 ml 작업은 복잡하고 귀찮고 모델의 평가 방법에 따라 순서도 바뀌게 됨
- 처리 전체를 구성하는 API가 표준으로 지원되길 원함
- spark ml은 mllib와 마찬가지로 머신러닝 알고리즘 API를 제공하며 나아가 데이터를 성형하고 평가하는 고수준 API도 제공
- 이 API를 이용하면 머신러닝의 학습과 예측 이외의 처리를 모듈로 다룰 수 있고 그 모듈을 조합해서 일련의 처리를 포괄적으로 다룰 수 있음
- 이 일련의 처리를 ML 파이프라인이라고 한다.


#### spark.ml vs spark.mllib

- 설명충
  - 기능적으로 충실해지고 있어 앞으로 주목도가 더 높다고 볼 수 있음
  - mllib와는 다른 개념으로 개발되었고 통계/머신러닝을 더 쉽게 개발할 수 있도록 고안되었음

- 잘 모르겠어 더 정확히
  - spark.mllib contains the original API built on top of RDDs.
  - spark.ml provides higher-level API built on top of DataFrames for constructing ML plielines
  - Pileplie is a series of algorithms
  - Easy ML workflow construction

[![ml-image03](https://github.com/leeplay/study/blob/master/image/ml-image03.png)]()
[![ml-image04](https://github.com/leeplay/study/blob/master/image/ml-image04.png)]()

- spark 2.0 Announcement
  - DataFrame-based API is primary API
  - RDD-bases API is noe in maintenance mode
  - RDD-bases API in spark.mllib will be depreated in spark 2.3
  - RDD-bases API in spark.mllib will be removed in spark 3.0

- 그럼 왜 DataFrame-based API를 써야하죠 ? 단순히 버전 때문에 ?
  - provides a more user-friendly API than RDDs.
  - The many benefits of Dataframes include Spark DAtasources, SQL/DataFrame queries, Tungsten and Catalyst optimizations.
  - provided a uniform API across ML algorithms and across multiple languages.
  - facilitate practical ML Pipelines, particularly feature transformations

[![ml-image05](https://github.com/leeplay/study/blob/master/image/ml-image05.png)]()


### 8.6.2 파이프라인의 핵심 컴포넌트

#### Transfomer (learned model)

- 데이터를 변환처리 하는 컴포넌트
- DataFrame 형식의 입력을 특정 처리를 거쳐 새로운 DataFRame 형식으로 출력한다.
- DataFrame -> Transformer.transform() -> DataFrame

#### Estimator (learnig algorithm)

- 트랜스포머 생성
- 알고리즘을 사용해 데이터를 트레이닝 시킴
- DataFrame -> Estimator.fit() -> Model(Transfomer)
- 에를 들어 학습 알고리즘으로 선형회귀를 택하면 Estimator는 선형회귀모델로 train 시킴

#### Pipeline

- Chain of Tranfomers and Estimators
- Pipeline itself is an Estimator
- 한번 정의하면 같은 단계로 반복해 처리할 수 있음

#### Param

- 트랜스포머와 에스티메이터 각각 가지는 매개변수


### 우리가 ML을 바라보는 자세

[![ml-image06](https://github.com/leeplay/study/blob/master/image/ml-image06.png)]()
[![ml-image07](https://github.com/leeplay/study/blob/master/image/ml-image07.png)]()

#### Example: Estimator, Transformer, and Param

```
import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.ml.linalg.{Vector, Vectors}
import org.apache.spark.ml.param.ParamMap
import org.apache.spark.sql.Row

val training = spark.createDataFrame(Seq(
  (1.0, Vectors.dense(0.0, 1.1, 0.1)),
  (0.0, Vectors.dense(2.0, 1.0, -1.0)),
  (0.0, Vectors.dense(2.0, 1.3, 1.0)),
  (1.0, Vectors.dense(0.0, 1.2, -0.5))
)).toDF("label", "features")

val lr = new LogisticRegression()
println(s"LogisticRegression parameters:\n ${lr.explainParams()}\n")

lr.setMaxIter(10).setRegParam(0.01)

val model1 = lr.fit(training)
println(s"Model 1 was fit using parameters: ${model1.parent.extractParamMap}")

val paramMap = ParamMap(lr.maxIter -> 20).put(lr.maxIter, 30).put(lr.regParam -> 0.1, lr.threshold -> 0.55)
val paramMap2 = ParamMap(lr.probabilityCol -> "myProbability")
val paramMapCombined = paramMap ++ paramMap2

val model2 = lr.fit(training, paramMapCombined)
println(s"Model 2 was fit using parameters: ${model2.parent.extractParamMap}")

val test = spark.createDataFrame(Seq(
  (1.0, Vectors.dense(-1.0, 1.5, 1.3)),
  (0.0, Vectors.dense(3.0, 2.0, -0.1)),
  (1.0, Vectors.dense(0.0, 2.2, -1.5))
)).toDF("label", "features")

model2.transform(test).select("features", "label", "myProbability", "prediction").collect().foreach { case Row(features: Vector, label: Double, prob: Vector, prediction: Double) =>
    println(s"($features, $label) -> prob=$prob, prediction=$prediction")
  }
```


#### Example: Pipeline

```
import org.apache.spark.ml.{Pipeline, PipelineModel}
import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.ml.feature.{HashingTF, Tokenizer}
import org.apache.spark.ml.linalg.Vector
import org.apache.spark.sql.Row

val training = spark.createDataFrame(Seq(
  (0L, "a b c d e spark", 1.0),
  (1L, "b d", 0.0),
  (2L, "spark f g h", 1.0),
  (3L, "hadoop mapreduce", 0.0)
)).toDF("id", "text", "label")

val tokenizer = new Tokenizer().setInputCol("text").setOutputCol("words")
val hashingTF = new HashingTF().setNumFeatures(1000).setInputCol(tokenizer.getOutputCol).setOutputCol("features")
val lr = new LogisticRegression().setMaxIter(10).setRegParam(0.001)
val pipeline = new Pipeline().setStages(Array(tokenizer, hashingTF, lr))

val model = pipeline.fit(training)
model.write.overwrite().save("/tmp/spark-logistic-regression-model")

pipeline.write.overwrite().save("/tmp/unfit-lr-model")

val sameModel = PipelineModel.load("/tmp/spark-logistic-regression-model")

val test = spark.createDataFrame(Seq(
  (4L, "spark i j k"),
  (5L, "l m n"),
  (6L, "spark hadoop spark"),
  (7L, "apache hadoop")
)).toDF("id", "text")

model.transform(test).select("id", "text", "probability", "prediction").collect().foreach { case Row(id: Long, text: String, prob: Vector, prediction: Double) =>
    println(s"($id, $text) --> prob=$prob, prediction=$prediction")
  }
```


### 숙제

#### Word2Vec으로 한국어 벡터화하기

#### 응용편:회귀에 의한 매출분석


## Spark with Deeplearning

### Deep learning and Apache Spark

#### Spark binding 

- caffe
- keras
- mxnet
- paddle
- tensorflow
- CNTK

#### Spark Native

- BIGDL
- DeepDist
- DeepLearning4J
- MLLib
- SparkCL
- SparkNet


### 2016년 동향

- integrate with Spark
- build on Spark
- modify Spark
- Official Spark MLlib support is limited(perceptron-like networks)
- 딥러닝이 Feature Engineering을 대신해주고 Hyper paramemter를 대신 최적화 해주고 병렬 인프라를 


### Databricks의 전망

- hosted Spark platform on public cloud
- Customers use many DeepLearning frameworks
- 2017년도 까지는 접목/실험 단계
[![ml-image08](https://github.com/leeplay/study/blob/master/image/ml-image08.png)]()


### 결합방식

[![ml-image09](https://github.com/leeplay/study/blob/master/image/ml-image09.png)]()
[![ml-image10](https://github.com/leeplay/study/blob/master/image/ml-image10.png)]()
[![ml-image11](https://github.com/leeplay/study/blob/master/image/ml-image11.png)]()

### 결론 

- Spark에 딥러닝을 접목하기 위한 모든 시도가 많이 이루어지고 있다.
- GPU와 Deeplearning Framework를 대체하기 보다는 시너지를 가지는 방향으로 발전될 것이다.

