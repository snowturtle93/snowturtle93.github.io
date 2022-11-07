---
title: MapR VS Cloudera
author: Kang HyunGu
date: 2022-11-07 22:34:00 +0900
categories: [Bigdata, Hadoop]
tags: [Bigdata]
toc: true
pin: false
comments: true
---

이번글에서는 Hive에서 작성된 SQL이 어떠한 과정을 통해 작업이 처리되는지 알아보려고 한다.


#### Hive에서 작업을 실행 시키는 단계.
<p align="left"> <img src="{{site.url}}/img/posts/hive_arc.jpg" width="600" height="450"></p> <br/>
1. Hive Driver에 Compiler는 Hive Metastore에서 필요한 읽기/쓰기 정보를 사용하여 쿼리를 구문 분석해 실행 계획을 생성.
2. Driver에 Optimization에 의해 최적화됨.
3. Excution Engine(MapReduce, Tez, Spark)
Hadoop Client로 제출.
4. Hadoop Client는 제출된 MapReduce 작업을 Yarn Resource Manager에게 전달.
5. Yarn Resource Manager는 아래 그림과 같이 Application Master를 실행하고 각각의 Application Master는 Node Manager에게 Container 실행 명령을 전달.
<p align="left"> <img src="{{site.url}}/img/posts/yarn_architecture.gif" width="600" height="450"></p>

[참고 1] https://tsktech.medium.com/apache-tez-2fd0d6a1d4e3<br/>
[참고 2] https://www.analyticsvidhya.com/blog/2020/10/getting-started-with-apache-hive/<br/>
