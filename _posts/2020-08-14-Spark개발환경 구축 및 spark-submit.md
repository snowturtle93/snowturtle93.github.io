---
title: Spark 개발환경 구축 및 spark-submit
author: Kang HyunGu
date: 2020-08-14 19:28:00 +0900
categories: [Bigdata, Spark]
tags: [Bigdata]
toc: false
pin: true
comments: true
---

poc 서버에서 scala 프로그램을 개발하기 힘들기 때문에 로컬에서 개발한 프로그램을 jar파일로 묶어 배포하는 방식이 필요하여 IntelliJ + Maven + Scala 환경을 구축한 뒤 spark-submit 테스트를 한다.

## 1. Project 생성
1.1 IntelliJ에서 New Project 클릭 후 Maven 프로젝트를 만들어준다.

이때 주의사항으로는 JDK 호환성 문제로 현재 사용하는 Scala 버전에서는 1.8 버전의 JDK로 프로젝트를 만들어준다.

 ![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/JDK선택.PNG){: width="700" height="300"}

1.2 Create from archetype 체크박스 선택후 Add Archetype 버튼 클릭.

 ![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/Archetype추가.PNG){: width="700" height="300"}

1.2 GroupId, ArtifactId, Version 입력
```
GroupId:net.alchim31.maven
ArtifactId:scala-archetype-simple
Version:1.6
```
![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/Archetype추가2.PNG){: width="700" height="300"}

1.3 프로젝트 정보 입력후 프로젝트 생성

![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/Project설정.PNG){: width="700" height="300"}

![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/Project설정2.PNG){: width="700" height="300"}

## 2. pom.xml 설정

2.1 Test 관련 설정 삭제.

![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/pom_test삭제.PNG){: width="700" height="300"}

2.2 testSourceDirectory, goal, arg 주석 or 삭제
![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/pom_test삭제2.PNG){: width="700" height="300"}

2.3 surefire 플러그인 삭제.
![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/surefire삭제.PNG){: width="700" height="300"}

2.4 shade 플러그인 추가.
``` xml
<plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-shade-plugin</artifactId>
     <version>3.0.0</version>
     <executions>
         <execution>
             <phase>package</phase>
             <goals>
                 <goal>shade</goal>
             </goals>
         </execution>
     </executions>
     <configuration>
         <filters>
             <filter>
                 <artifact>*.*</artifact>
                 <excludes>
                     <exclude>META-INF/*.SF</exclude>
                     <exclude>META_INF/*.DSA</exclude>
                     <exclude>META_INF/*.RSA</exclude>
                 </excludes>
             </filter>
         </filters>
         <finalName>${project.artifactId}-khg-${project.version}</finalName>
     </configuration>
 </plugin>
```
![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/shade추가.PNG){: width="700" height="300"}

2.5 scala 관련 디펜던시 추가.
``` xml
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>2.4.4</version>
        </dependency>

        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>2.4.4</version>
        </dependency>
```
![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/spark설정추가.PNG){: width="700" height="300"}

## 3. 코드 작성 및 컴파일.
3.1 코드작성
```
package com.khg
import org.apache.spark.sql.SparkSession
import org.apache.hadoop.conf._
import org.apache.hadoop.fs._

object csvToORC {
  case class data(ADT_CODE: String, ADT_YYMM: String, ADT_DIV: String, LOC_CODE: String, ADT_LOC: String, ADT_AREA: String, ADT_NUM: Int, DEATH_NUM: Int, SERIOUS_INJURED_NUM: Int, SLIGHTH_INJURED_NUM: Int, INJURED_NUM: Int)

  def main(args: Array[String]): Unit = {
    val conf = new Configuration()
    val fs = org.apache.hadoop.fs.FileSystem.get(conf)
    val targetPath= new Path("/data/test.orc")
    val spark = SparkSession.builder
      .appName("csvToORC")
      .getOrCreate()

    val df = spark.read
      .option("header", "true") // Use first line of all files as header
      .option("encoding", "UTF-8")
      .csv("/data/test.csv")

    df.createOrReplaceTempView("test")
    val sqlDF = spark.sql("SELECT * FROM test")
    if (fs.exists(targetPath)) {
      fs.delete(targetPath, true)
    }
    sqlDF.write.format("orc").save(targetPath.toString)
  }
}

```
3.2 컴파일

컴파일은 우측 메이븐 -> Lifecycle -> clean, complie, package 순으로 진행한다.


![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/컴파일.PNG){: width="700" height="300"}

3.3 jar 생성

target에 jar 파일 생성 완료.


![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/jar생성.PNG){: width="300" height="400"}

## 4. spark-submit

서버에 jar 파일 및 테스트데이터 업로드 후 spark-submit 진행.

```
spark-submit --master yarn --class com.khg.csvToORC --name "csvToORC"  /home/mapr/sparkConv-1.0.jar
```

![Alt text]({{site.url}}/img/posts/2020-08-14-ScalaMaven/완료.PNG){: width="750" height="200"}

## 5. pom.xml
``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.khg</groupId>
    <artifactId>sparkConv</artifactId>
    <version>1.0</version>
    <name>${project.artifactId}</name>
    <description>My wonderfull scala app</description>
    <inceptionYear>2015</inceptionYear>
    <licenses>
        <license>
            <name>My License</name>
            <url>http://....</url>
            <distribution>repo</distribution>
        </license>
    </licenses>
    <repositories>
        <repository>
            <id>mapr-releases</id>
            <url>https://repository.mapr.com/maven/</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>
    </repositories>
    <properties>
        <maven.compiler.source>1.6</maven.compiler.source>
        <maven.compiler.target>1.6</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11.5</scala.version>
        <scala.compat.version>2.11</scala.compat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>
        <!-- hadoop 설정 추가 -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.0-mapr-1808</version>
        </dependency>
        <!--Spark 설정 추가 -->
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>2.4.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>2.4.4</version>
        </dependency>
        <!--Spark 설정 추가 끝 -->
        <!--    &lt;!&ndash; Test &ndash;&gt;
            <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.11</version>
              <scope>test</scope>
            </dependency>
            <dependency>
              <groupId>org.specs2</groupId>
              <artifactId>specs2-core_${scala.compat.version}</artifactId>
              <version>2.4.16</version>
              <scope>test</scope>
            </dependency>
            <dependency>
              <groupId>org.scalatest</groupId>
              <artifactId>scalatest_${scala.compat.version}</artifactId>
              <version>2.2.4</version>
              <scope>test</scope>
            </dependency>-->
    </dependencies>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <!-- <testSourceDirectory>src/test/scala</testSourceDirectory>-->
        <plugins>
            <plugin>
                <!-- see http://davidb.github.com/scala-maven-plugin -->
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <!--<goal>testCompile</goal>-->
                        </goals>
                        <configuration>
                            <args>
                                <!--   <arg>-make:transitive</arg>
                                   <arg>-dependencyfile</arg>
                                   <arg>${project.build.directory}/.scala_dependencies</arg>-->
                            </args>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <filters>
                        <filter>
                            <artifact>*.*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META_INF/*.DSA</exclude>
                                <exclude>META_INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                    <finalName>${project.artifactId}-khg-${project.version}</finalName>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```
