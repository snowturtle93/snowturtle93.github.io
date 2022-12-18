---
title: Spark가 Lazy Evaluation을 사용하는 이유
author: Kang HyunGu
date: 2022-12-18 14:10:00 +0900
categories: [Bigdata, Spark]
tags: [Bigdata]
toc: true
pin: false
comments: true
---
#Spark가 Lazy Evaluation을 사용하는 이유

<p align="left"> <img src="{{site.url}}/img/posts/lazy_spark.jpeg" width="600" height="450"></p> <br/>

[이미지 출처](https://www.facebook.com/cheongjeul/posts/1607810786091040/?comment_id=1610467795825339)

##Lazy Evaluation이란?
우선 Lazy Evaluation이란 개념은 Spark에서 생긴 개념은 아니며 프로그래밍에서 불필요한 연산을 줄이고자 생긴 연산 전략이다.

##Transformation과 Action 함수
Spark에서 Lazy Evaluation를 사용하는 이유를 설명하기 전에 [이전 포스팅](https://snowturtle93.github.io/posts/Spark%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4%EC%84%9C-%EA%B2%BD%ED%97%98%ED%95%9C-%ED%8A%9C%EB%8B%9D%ED%8F%AC%EC%9D%B8%ED%8A%B8/)에서 언급한 [Transformation](https://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations)함수와 [Action](https://spark.apache.org/docs/latest/rdd-programming-guide.html#actions)함수를 알아야 한다.

###Transformation 함수
Spark에서 Transformation함수는 DataFrame을 원하는 방식으로 수정하는 데 사용하는 함수로 대표적으로는 filter, map, join, group by 함수들이 있으며 실제 데이터 처리가 수행되지는 않는다.

###Action 함수
Spark에서 실제 데이터 처리가 수행되는 함수로 대표적으로는 count, reduce, save, write 함수들이 있다.

이처럼 Spark는 Action 함수가 호출 되기 전까지는 아무리 많은 Transformation함수를 호출 하더라고 데이터 처리를 하지않는 Lazy Evaluation을 사용한다.

## Catalyst Optimizer
Spark에서 Lazy Evaluation를 사용하는 이유를 설명하기 전에 추가로 하나 더 알면 좋은 Catalyst Optimizer에 대해서 짧게 설명 하도록 한다.

Spark에서 Transformation을 수행하게 되면 Spark는 프로그래밍 되어있는 데이터 처리 순서에 맞게 DAG(비순환 그래프)를 생성하며 Catalyst Optimizer는 생성된 DAG를 분석하여 비용 또는 규칙 기반 최적화를 수행하여 논리적 실행 계획과 물리적 실행 게획은 세우게 된다.
<p align="left"> <img src="{{site.url}}/img/posts/Catalyst-Optimizer-diagram.png" width="600" height="450"></p> <br/>

##Lazy Evaluation을 사용하는 이유
Lazy Evaluation 무엇인지 알았으면 이제 Spark에서는 왜 사용하는지 알아볼 차례이다.
Lazy Evaluation을 사용하는 이유는 아래와 같다.

###1. 불필요한 데이터 처리를 생략할 수 있다
만약 Spark에서 Transformation 함수 호출 시에 즉시 데이터 처리를 하게 된다면 아래와 같이 비효율적인 상황이 발생할 수 있다.

가령 데이터 프레임을 처리하면서 map 함수를 사용하여 100만 건의 데이터를 모두 20을 더한 뒤에 다음 코드에서 4를 추가로 더한다면 Spark가 즉시 처리하게 된다면 2번 데이터 처리가 발생하지만, Lazy Evaluation을 한다면 [Catalyst Optimizer](databricks.com/blog/2015/04/13/deep-dive-into-spark-sqls-catalyst-optimizer.html)가 1번의 데이터 처리할 수 있도록 물리적 실행계획을 세워주게 된다.

###2. 메모리 관리의 효울성
Spark의 Transformation 함수 호출시에 즉시 처리하게 된다면 모든 중간 데이터 프레임/RDD를 어딘가에 저장해야하는데 반해
Lazy Evaluation을 사용하면 Spark는 실제로 필요한 중간 결과값만 저장하기 때문에 필요하지 않은 데이터를 삭제하는 데 발생하는 GC 비용을 줄일 수 있다.

###3. Overhead를 줄일 수 있다.
꼭 필요한 연산만 처리할 수 있기 때문에 드라이버와 클러스터 간의 데이터 이동이 절약되므로 프로세스를 빠르게 할 수 있다.


[참고 1] https://towardsdatascience.com/3-reasons-why-sparks-lazy-evaluation-is-useful-ed06e27360c4


[참고 2] https://data-flair.training/blogs/apache-spark-lazy-evaluation/


[참고 3] https://medium.com/analytics-vidhya/being-lazy-is-useful-lazy-evaluation-in-spark-1f04072a3648
