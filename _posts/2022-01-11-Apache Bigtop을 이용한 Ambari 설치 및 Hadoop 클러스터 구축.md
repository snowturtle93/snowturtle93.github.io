---
title: "Apache Bigtop을 이용한 Ambari 설치 및 Hadoop 클러스터 구축"
author: Kang HyunGu
date: 2022-01-11 20:36:00 +0900
categories: [Bigdata, Hadoop]
tags: [Bigdata]
toc: true
pin: true
comments: true
---

## 1.Apache Bigtop이란?
Bigtop은 Hadoop Ecosystem을 패키징, 테스트, 가상화를 지원해주는 프로젝트로 3.0 기준으로 아래 Hadoop Ecosystem 패키징을 지원한다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/bigop3.0_release.png" width="900" height="400"></p>
<center> Bigtop Release3.0(출처:https://cwiki.apache.org/confluence/display/BIGTOP/Bigtop+3.0.0+Release) </center><br/>

위의 패키징에서 주목할점은 Ambari와 bigtop-ambari-mpack인데 Cloudera와 Hortonworks의 합병으로 Ambari rpm 파일은 제공이 중단되었고 소스를 받아 빌드 해야하는데 hdp repo가 막혀 빌드도 어려운 상태이며 hdp 스택역시 repo가 막혀 설치하기가 힘든 상태이다.
이러한 상황에서 Apache Bigtop은 훌륭한 대체재로 사용될 수 있을거 같아 사용해보려 한다.

## 2.Apache Bigtop 설치 전 준비 사항.

## 4.Mysql 설정.
<pre><code>
#UTF8 설정. HIVE 테이블에 한글 깨짐 방지.

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

#비밀번호 관련 보한 설정.

validate_password-policy=LOW
validate_password-length=4
</code></pre>
