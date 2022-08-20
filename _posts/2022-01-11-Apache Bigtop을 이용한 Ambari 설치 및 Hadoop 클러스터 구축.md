---
title: "Apache Bigtop을 이용한 Ambari 설치 및 Hadoop 클러스터 구축"
author: Kang HyunGu
date: 2022-01-11 20:36:00 +0900
categories: [Bigdata, Hadoop]
tags: [Bigdata]
toc: true
pin: false
comments: true
---

## 1.Apache Bigtop이란?

Bigtop은 Hadoop Ecosystem을 패키징, 테스트, 가상화를 지원해주는 프로젝트로 3.0 기준으로 아래 Hadoop Ecosystem 패키징을 지원한다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/bigop3.0_release.png" width="900" height="300
"></p>
<center> Bigtop Release3.0(출처:https://cwiki.apache.org/confluence/display/BIGTOP/Bigtop+3.0.0+Release) </center><br/>

위의 패키징에서 주목할점은 Ambari와 bigtop-ambari-mpack인데 Cloudera와 Hortonworks의 합병으로 Ambari rpm 파일은 제공이 중단되었고 소스를 받아 빌드 해야하는데 hdp repo가 막혀 빌드도 어려운 상태이며 hdp 스택역시 repo가 막혀 설치하기가 힘든 상태이다.
이러한 상황에서 Apache Bigtop은 훌륭한 대체재로 사용될 수 있을거 같아 사용해보려 한다.

## 2.Apache Bigtop 설치 전 준비 사항.

Apache Bigtop 자체는 git만 설치되면 clone 받아 사용할 수 있지만 빌드하려는 Hadoop ecosystem에 따라 필요한 패키지들이 있다.
우선 Ambari 빌드를 위해서 아래 리스트의 패키지를 설치한다.
- Maven 설치 (3.6.3 버전 설치 3.8.x 버전 빌드 에러 발생)
- Python-devel 설치
- gcc, gcc++ 설치
- rpm-build 설치
- git 설치
- openjdk 1.8 설치

jdk와 maven 환경 설정 진행 과정은 생략.
<pre><code>
sudo yum install rpm-build
sudo yum install python-devel
sudo yum install git
sudo yum install gcc
sudo yum install gcc-c++
sudo yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel
wget https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
</code></pre>

## 3.Apache Bigtop 설치.
Bigtop은 사실 설치랄게 없고 git clone을 받으면 된다 이전 버전에는 gradle 설치가 필요했던거 같은데 지금은 gradlew가 프로젝트 안에 있어 gradle 설치는 필요없다.
<pre><code>
git clone https://github.com/apache/bigtop.git
# clone후에 아래 명령어로 사용가능한 명령어 리스트 출력.
./gradlew task --all
</code></pre>

## 4.Apache Bigtop을 활용한 빌드.
- Ambari 빌드

<pre><code>
./gradlew task ambari-pkg
</code></pre>

빌드가 완료되면 {bigtop_home}/build/ambar/rpm/RPMS/noarch 디렉토리에 rpm 파일이 생성.

- bigtop-ambari-mpack 빌드
bigtop-ambari-mpack은 Ambari에서 bigtop Stack으로 Hadoop Ecosystem을 설치 할 수 있도록하는 패키징이다.

<pre><code>
./gradlew task bigtop-ambari-mpack-pkg
</code></pre>

빌드가 완료되면 {bigtop_home}/build/bigtop-ambari-mpack/rpm/RPMS/noarch 디렉토리에 rpm 파일이 생성.

- bigtop-utils 빌드
bigtop-ambari-mpack 설치시 필요
<pre><code>
 ./gradlew task bigtop-utils-pkg
</code></pre>

## 5. 빌드된 rpm 설치.
- 빌드 결과
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/bulid.png" width="600" height="150"></p>
<pre><code>

#ambari-server 설치
sudo yum install ambari-server-2.7.5.0-1.el7.noarch.rpm
#ambari-agent 설치 (모든 서버 설치)
sudo yum install ambari-agent-2.7.5.0-1.el7.noarch.rpm
#bigtop-utils 설치
sudo yum install bigtop-utils-3.1.0-1.el7.noarch.rpm
#bigtop-ambari-mpack 설치
sudo yum install bigtop-ambari-mpack-2.7.5.0-1.el7.noarch
#mpack 파일 위치 찾기.
rpm -qs bigtop-ambari-mpack-2.7.5.0-1.el7.noarch
경로 : /usr/lib/bigtop-ambari-mpack/bgtp-ambari-mpack-1.0.0.0-SNAPSHOT-bgtp-ambari-mpack.tar.gz

</code></pre>


## 6.Mysql 설정.
<pre><code>
/etc/my.cnf 파일에 아래 설정 추가.
#UTF8 설정. HIVE 테이블에 한글 깨짐 방지.

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

#비밀번호 관련 보안 설정.

validate_password-policy=LOW
validate_password-length=4

mysql 재시작 : sudo systemctl restart mysqld

</code></pre>

## 7.Mysql ambari Metadata 생성.
<pre><code>
# database 생성.
create database ambari;
create database hive;

# ambari, hive계정 생성
create user 'ambari'@'localhost' identified by 'ambari';
create user 'ambari'@'%' identified by 'ambari';
grant all privileges on *.* to 'ambari'@'localhost' identified by 'ambari' with grant option;
grant all privileges on *.* to 'ambari'@'%' identified by 'ambari' with grant option;
flush privileges;

create user 'hive'@'localhost' identified by 'hive';
create user 'hive'@'%' identified by 'hive';
grant all privileges on *.* to 'hive'@'localhost' identified by 'hive' with grant option;
grant all privileges on *.* to 'hive'@'%' identified by 'hive' with grant option;
flush privileges;

# ambari meta table 생성
use ambari;
SOURCE /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;
</code></pre>

## 8.Ambari 설치

- mysql jdbc 커넥터 설치.
sudo yum install mysql-connector-java*
- mysql 커넥터 경로 설정.
sudo ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
- ambari server setup
sudo ambari-server setup
- ambari server start
sudo ambari-server start
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/server.png" width="900" height="400"></p>
- ambari 접속화면
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/ambari1.png" width="900" height="400"></p>
- 설치 가능한 Stack이 없기 때문에 설치가 불가능하다 그래서 아까 설치했던 bigtop-mpack을 추가 설치 한다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/ambari2.png" width="900" height="400"></p>
- mpack 설치
sudo ambari-server install-mpack --mpack=/usr/lib/bigtop-ambari-mpack/bgtp-ambari-mpack-1.0.0.0-SNAPSHOT-bgtp-ambari-mpack.tar.gz --verbose
- ambari 재실행.
sudo ambari-server restart
ambari를 재실행하면 아래와같이 bigtop 1 버전이 스택으로 선택이 가능한데 설치할수 있는 모듈이 기본 모듈밖에 없어 이것을 추가하는 방법을 찾아봐야겠다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/ambari3.png" width="700" height="300"></p>
