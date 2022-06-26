---
title: "Spark로 Parquet파일 새로운 컬럼을 맨앞에 추가하는 방법"
author: Kang HyunGu
date: 2022-06-26 21:38:00 +0900
categories: [Bigdata, Spark]
tags: [Bigdata]
toc: true
pin: true
comments: true
---
이번에 샤딩된 데이터베이스에서 유입되는 데이터베이스에 어떤 데이터베이스 인스턴스에서 데이터가 유입되었는지 파악하기 위한 컬럼이 필요해 스파크를 활용해 컬럼을 추가해 보았다.
인터넷에 컬럼 추가 방법을 검색하면 withColumn 함수를 사용하는게 많이 검색이 되었는데
withColumn 함수를 그냥 사용하면 Parquet파일 맨뒤에 컬럼이 추가가 되어 문제가 발생했다.
테이블 컬럼이 추가가되면 Parquet파일에 맨뒤에 추가가 될텐데 이렇게 되면 기존 Parquet파일구조와 달라져 이슈가 발생하기 때문에 컬럼을 맨앞에 추가 할 필요성이 있었다.

## Pyspark 샘플코드
```
df = spark.read.parquet(dir)
current_column =df.schema.names
addedColDf = df.withColumn('addCol',lit(1))
#컬럼 맨앞자리로 되어있는 리스트 생성
columns = ['addCol']+current_column
addedColDf = addedColDf.select(columns)
```
Select문을 검색하고 싶은 순서대로 추가해주는 간단한 방법이다.
