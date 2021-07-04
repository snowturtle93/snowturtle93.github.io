---
title: "Docker Run 옵션"
author: Kang HyunGu
date: 2021-07-03 17:36:00 +0900
categories: [Infra, Docker&K8s]
tags: [Infra]
toc: true
pin: true
comments: true
---

## 1. Docker Run이란?
Docker Run은 이름에서 알수 있듯이 Docker 이미지로 컨테이너를 만들어 실행 시키는 명령어로 Docker start는 기존에 생성되어있는 컨테이너를 실행시키는 반면 Run은 이미지로부터 컨테이너를 새로 만든다는 차이점이있다.
 <br/>
우선 기본 명령어는 아래와 같다.
```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```
<br/>

## 2. Docker Run 기본  옵션
<br/>
기본적으로 docker run --help 명령어로 docker run 옵션을 알 수 있지만 자주 쓰는 옵션을 정리해보려고 한다.

### 2.1 -it 옵션
옵션 i와 t는 같이 사용하는 경우가 많은데 i는 --interactive 옵션으로 STDIN(표준입력)으로 컨테이너를 생성 하라는 뜻이다. <br/>
t는 --tty 옵션으로 영어로 Allocate a pseudo-TTY 라고 설명이 되어있는데 여기서 pseudo-TTY는 유사 터미널로 컨테이너에 터미널 드라이버를 추가하여 컨테이너를 터미널을 이용하여 연결 할 수있도록 하는 옵션이다. <br>
정리하자면 i옵션으로 표준 입력을 받으며 t옵션으로 터미널로 연결 가능한 컨테이너를 만드는 것이다.

### 2.2 -d 옵션
컨테이너를 백그라운드로 실행시키는 옵션이다.
### 2.3 -h 옵션
컨테이너의 호스트명 지정
### 2.4 -e 옵션
-e, --env=[]: 컨테이너에 환경 변수를 설정.
예를 들어 환경변수에 AIRFLOW_HOME을 설정하고싶다면 아래 처럼 명령어를 실행하면된다.
```
docker run -d -e AIRFLOW_HOME=/home/airflow/airflow my_test_image
```
### 2.5 -rm 옵션
컨테이너안의 프로세스 종료시 컨테이너를 자동으로 삭제하며 -d 옵션과 같이 사용할 수 없다.
### 2.6 -name 옵션
컨테이너의 이름을 지정하여 실행시키는 옵션.
### 2.7 -v 옵션
로컬과 컨테이너 간의 볼륨을 설정하기 위해 사용되는데 로컬 파일시스템의 특정경로를 컨테이너의 파일시스템에 특정 경로로 마운트 시킨다.
* 아래 명령어는 컨테이너를 실행시키는 현재 디렉토리를 컨테이너안의 /home/airflow에 마운트시켜 컨테이너가 종료되어도 /home/airflow안의 파일이 삭제되지않게 해주는 명령어이다.
```
docker run -d -v -v $PWD:/home/airflow my_test_image
```

## 3. 네트워크 옵션
### 3.1 -p 옵션
컨테이너의 포트를 포워딩 하는 옵션으로 아래와 같이 옵션을 적용하면 컨테이너의 80포트를 로컬의 8080포트로 포워딩한다.
```
docker run -d -p 8080:80 my_test_image
```
### 3.2 --net 옵션
Docker에서 아래 명령어로 네트워크를 만들면 만들어진 네트워크안에 속하는 컨테이너를 만들어 컨테이너간의 통신을 가능하게 한다.
```
docker network create [OPTIONS] [NETWORK NAME]
```
여기서 네트워크 옵션의 주요 옵션은 아래와 같다.
* --driver or -d : 네트워크 드라이버 설정 옵션 기본값은 bridge이다.
* --ip-range : 네트워크 범위 설정.
* --subnet : subnet 설정.
* --gateway : gatewat 설정.
* --ipv6 : ipv6네트워크를 사용할지 기본은 false이다.
* -label : 네트워크에 라벨을 설정할 수 있다.
```
docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  my_network
```
&#8251; 네트워크 드라이버의 경우 4개의 종류가 있어 따로 정리할 필요가 있어보인다.

## 4. 자원 관리 옵션
### 4.1 --cpu 옵션
CPU 사용 갯수 제한 옵션.
### 4.2 --cpus 옵션
CPU 사용 비율 제한 옵션.
여기서 특이한점은  CPU 를 1개 사용하면 값의 지정 범위는 0~1이고 4개를 사용하면 0~4이다.
따라서 cpu 옵션을 4로 주고 cpus를 1로주면 4개의 cpu를 25%씩 사용한다는 의미이다.
### 4.3 --cpu-shares
CPU 컨테이너간 가중치 부여 옵션.
* 예를 들어 A컨테이너의 cpu-shares를 1024를 부여하고 B컨테이너에는 512를 부여하면 두 컨테이너가 같은 코어를 사용하려고 경합하게 되면 A컨터이너에게 우선권이 부여된다.
* 아래의 명령어는 3개의 cpu를 33%씩 사용하며 cpu 사용 가중치는 1024로 컨테이너를 생성하라는 명령어이다.
```
docker run -d --cpus=1 --cpu=3 --cpu-shares 1024 my_test_image
```
### 4.4 --memory or -m
컨테이너가 사용할 최대 메모리 설정 옵션으로 최소값은 4m이다.
### 4.5 --memory-swap
memory-swap 옵션이 0이상으로 설정되어있다면 memory옵션과 쌍으로 같이 설정되어야하는데 그 이유는 memory-swap 옵션이 1G에 memory옵션이 300M라면 실제로 메모리가 스왑되는건 700M이다.
