---
title: Scala 프로젝트 Gradle로 빌드하기
author: Kang HyunGu
date: 2020-08-07 18:31:00 +0900
categories: [Programming, Scala]
tags: [Programming]
toc: false
pin: false
comments: true
---

## 0. IDE는 IntelliJ 를 사용하여 개발합니다.

## 1. Gradle 프로젝트 만들기.
  - New Project 설치 후 Gradle 클릭 후 Java 체크 해지 후 Next 버튼 클릭.

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE1.PNG){: width="600" height="400"}

  - 프로젝트명, Groupid 적어주고 프로젝트 생성.

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE2.PNG){: width=600" height="400"}

## 2. 디렉토리 만들기.
  - src -> main -> scala 순서로 하위프로젝트 만들어준다.

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE3.PNG){: width="600" height="400"}

  - scala 폴더 우클릭 -> Mark Directory as -> Sources Root 로 scala 디렉토리를 소스의 루트로 적용

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE4.PNG){: width="600" height="400"}

## 3. add FrameWorks Support에서 scala 추가해주기.
  - 프로젝트 우클릭 -> add FrameWorks Support -> scala 체크

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE5.PNG){: width="600" height="400"}

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE6.PNG){: width="300" height="400" .aligncenter}

  - scala가 설치되어있지 않다면 아래와 같은 에러가 발생됨.

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE7.PNG){: width="300" height="400" .aligncenter}

  - ok 버튼을 클릭

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE8.PNG){: width="300" height="400" .aligncenter}

  - Download 클릭으로 scala sdk 설치.

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE9.PNG){: width="300" height="400" .aligncenter}

  - ok 버튼 클릭으로 scala 추가 완료

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE10.PNG){: width="300" height="400" .aligncenter}

## 4. scala Class 만들기
  - 3번에서 scala를 정상적으로 추가해줬다면 Scala class 파일을 생성할수 있습니다.

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE11.PNG){: width="600" height="400"}

  - scala (Sources Root)밑에 패키지 만든뒤 (패키지만들기는 선택 사항) bulid.gradle에 아래와 같이 추가.

  <pre><code>
  group 'com.khg.convertor'
  version '1.0-SNAPSHOT'
  apply plugin: 'scala'
  repositories {
    mavenCentral()
  }
  dependencies {
    implementation 'org.scala-lang:scala-library:2.12.11'
    testImplementation 'org.scalatest:scalatest:2.12:3.0.0'
    testImplementation 'junit:junit:4.13'
  }
  </code></pre>

  ![image]({{site.url}}/img/posts/2020-08-07_ScalaGradle/SCALA_GRADLE12.PNG){: width="600" height="400"}
