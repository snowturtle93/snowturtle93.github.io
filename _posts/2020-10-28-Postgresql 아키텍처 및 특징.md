---
title: "Postgresql 아키텍처 및 특징."
author: Kang HyunGu
date: 2020-10-28 18:32:00 +0900
categories: [Database, RDBMS]
tags: [Database]
toc: true
pin: true
comments: true
---
Informatica PowerCenter가 MetaDB로 Postgresql을 지원하면서 관세청에 PowerCenter가 납품 될 때 Postgresql과 같이 납품이 되어 Postgresql을 설치했다.


Postgresql 설치를 하면서 조금 공부를 한 내용을 정리한다.

## 0. 특징.
- 프로세스 기반의 DBMS이다.

- 1개의 connection 마다 1개의 backend 프로세스 생성(Postmater 프로세스에 의해 fork)한다.

- Autovacuumlauncher/worker 프로세스이다.

## 1. 아키텍처
- PostgreSQL의 아키텍처는 아래와 같이 구성되어있다.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Shared Memory
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Background processes
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Data directory structure / Data files

<p align="center"><img src="{{site.url}}/img/posts/2020-10-28-Postgresql 아키텍처 및 특징/PostgreSQL-Architecture1.jpg" width="600" height="400"></p>
<br/>

###1.1 Shared Memory
Shared Memory는 트랜잭션 및 다른 로그 캐치용으로 할당 된 메모리로 Shared Buffers, WAL Buffers, Work Memory, Maintenance Work Memory로 구성되어있다.
이 설정은 Postgresql의 data 폴더의 postgresql.conf 파일에서 설정 가능하다.

####1.1.1 Shared Buffers
PostgreSQL도 여느 RDBMS와 마찬가지로 데이터베이스에 엑세스를 시도하면, 먼저 디스크에서 필요한 데이터를 공유 버퍼로 먼저 읽어 들인다.(Oracle의 SGA와 같은 개념인듯.)  그리고, 공유 버퍼에서 데이터를 읽고, 쓰기를 처리한다. 이후, 동일한 데이터 엑세스에 대해서는 공유버퍼에서 읽어 들여, 느린 디스크 엑세스를 줄여 성능을 확보 할 수 있는 기본적인 튜닝이다.
기본 값은 128MB 이며, 최소 128KB 이상으로 설정하여야 한다.
shared_buffers 설정은 서버 메모리의 1/4 또는 1/2정도이다. (공식 페이지에서는 40%를 넘지 않도록 권장한다.)
설정값은 MB, GB 등 단위별로 설정 가능하다.
이렇게 Shared Buffers 메모리를 사용하는 이유는 Disk I/O를 최소하 하기 위해서 사용한다.

####1.1.2 WAL Buffers
데이터를 업데이트 할 때 어떤 변경을 할 것 인지를 남기는 로그이며, PostgreSQL에서는 WAL(Write Ahead Log)이라 명칭함
손실된 데이터 복원에 아주 중요한 역할.
config파일에서 wal_buffers 값 변경, 일반적으로 shared_buffers의 1/32 크기로 지정.
WAL(Write Ahead Logging) Buffer의 변경사항이 WAL 파일로 작성되는데 이 WAL 파일을  StandBy 서버에 전송하는 방식을 이용하여 Posgresql HA 구성을 할 수있다.


####1.1.3 Work Memory
내부 정렬 및 해시 테이블에 사용항 메모리 양을 설정한다.
정렬 작업은 ORDER BY, DISTINCT 및 병합 조인에 사용되고 해시 테이블은 해시 조인, 해시 기반 집계 및 IN 하위 쿼리의 해시 기반 처리에 사용된다.
기본값은 4MB이다.
여러 세션이 병렬로 메모리를 사용할 수 있으므로 전체 메모리는 설정한 값의 몇배가 될 수 있다.
work_mem의 설정은 1 / (max_connections * 2)으로 설정하는 것이 적당하다.
예를들어 1GB의 서버 메모리에 max_connections가 100이면 1024/200 = 5.12로 약 5MB로 설정하면 된다.
설정을 튜닝한다고 이 메모리를 무작정 늘리게되면 out of memory 오류가 발생.

####1.1.4 Maintenance Work Memory
VACUUM, ANALYZE, ALTER TABLE, CREATE INDEX 및 ADD FOREIGN KEY 등과 같은 데이터베이스 유지 관리 작업을위한 메모리.
9.3 및 이전 버전의 Maintenance Work Memory의 기본값은 9.4에서 16MB (16MB)이고 이후 Maintenance Work Memory의 기본값은 64MB이다.
Work Memory에 비해 큰 Maintenance Work Memory를 설정하는 것이 안전하다고 한다. 설정이 클수록 유지 관리 (VACUUM, ANALYZE, ALTER TABLE, CREATE INDEX 및 ADD FOREIGN KEY 등) 작업의 성능이 향상됨.

여기서 VACCUM은 Postgresql MVCC(다중 버전 동시성 제어)를 구현하기 위한 방법으로 데이터를 UPDATE 또는 DELETE시에 디스크에 있던 기존 정보를 갱신하거나 삭제하지 않고 기존 정보는 변경되었다는 표시를 남기고 새롭게 디스크에 UPDATE된 정보를 기록하는데 이 과정에서 DELETE 했어도 디스크 용량은 줄어들지 않으며 UPDATE 시에는 새로운 행이 추가되기 때문에 디스크 용량이 증가한다 이때 이 낭비된 공간을 제거해주는 명령어가 VACCUM이다.
(DBMS별 MVCC 구현 방법은 다음 포스팅에서 공부해서 올려야겠다.)
1) Postmaster 프로세스는 제일 앞단에서 Client와 맞닫는 프로세스입니다.

- PostgreSQL 기동할 때 가장 먼저 시작되는 프로세스이다.

- 초기 복구 작업, 메모리 초기화, Background 프로세스 기동작업 수행한다.

- 데몬 프로세스로 Client 프로세스의 접속 요청을 받아 Backend 프로세스를 생성한다.


###1.2 Background Processes
2) Background Processes는 로그, 통계 등 백그라운드에서 배치성으로 돌아가는 프로세스입니다.

- logger : 에러 메시지를 로그 파일에 기록한다.
- checkpointer : 체크 포인트 발생시 dirty 버퍼를 데이터파일에 기록한다.
- Background writer : 로그 및 백업 정보를 최신 상태로 유지.
- wal writer : WAL 버퍼를 WAL 파일에 기록한다.
- autovacuum launcher : Vacuum 이 필요한 시점에 Postmaster 프로세스에 autovacuum worker 프로세스 기동 요청한다.
- archiver : Archive mode 사용시 WAL 파일을 지정된 디렉토리로 복사한다.
- stats collector 세션 수행 정보, 테이블 사용 통계 정보 같은 DBMS 통계 정보 수집한다.

###1.3 Data directory structure / Data files
PostgreSQL은 데이터베이스 Cluster(Cluster는 하나의 서버 인스턴스에 의해 관리되는 여러 Database의 모음)라고 불리는 여러 데이터베이스로 구성된다. Postgre를 초기화할 때SQL 데이터베이스 template0, template1 및 Postgres 데이터베이스가 생성됨
Template0 및 Template1은 시스템 카탈로그 테이블을 포함하는 새로운 데이터베이스 사용자 작성을 위한 템플리트 데이터베이스다.
사용자 데이터베이스는 template1 데이터베이스를 복제하여 생성된다.

Cluster에 대한 구분은 아래 3가지로 할 수 있다.
- Data directory
- TCP Port
- Set of processes

PGDATA 디렉터리에는 여러 하위 디렉터리가 포함되어 있으며 제어 파일은 다음과 같습니다.

<p align="center"><img src="{{site.url}}/img/posts/2020-10-28-Postgresql 아키텍처 및 특징/PostgreSQL-Architecture2.jpg" width="600" height="400"></p>
<br/>

- pg_version : 데이터베이스 버전 정보를 포함합니다.
- base : 데이터베이스 하위 디렉토리를 포함합니다.
- global : pg_user와 같은 클러스터 현명한 테이블 포함.
- pg_clog : 트랜잭션 포함은 상태 데이터를 커밋합니다.
- pg_multixact : 다중 트랜잭션 상태 데이터 포함 (공유 행 잠금에 사용됨).
- pg_notify :  LISTEN / NOTIFY 상태 데이터 포함.
- pg_serial : 커밋 된 직렬화 가능한 트랜잭션에 대한 정보 포함.
- pg_snapshots :  내 보낸 스냅 샷 포함.
- pg_stat_tmp : 통계 서브 시스템을위한 임시 파일을 포함합니다.
- pg_subtrans : 하위 트랜잭션 상태 데이터 포함.
- pg_tblspc : 테이블 스페이스에 대한 심볼릭 링크 포함.
- pg_twophase :  준비된 트랜잭션을위한 상태 파일 포함.
- pg_xlog :  WAL (Write Ahead Log) 파일 포함.
- pid :  현재 포스트 마스터 프로세스 ID (PID)를 포함하는이 파일.

출처 [1]: https://www.educba.com/postgresql-architecture/
출처 [2]: https://waspro.tistory.com/146
출처 [3]: http://www.gurubee.net/lecture/2889
