---
title: Mac에서 Oracle 실행하기
titleEn: Running Oracle on macOS
author: BabyK
date: 2025-12-08
category: Web
layout: post
tags: [Web, Docker, Oracle]
published: true
---

### 도커 설정
오라클은 맥을 지원하지 않지만 도커 컨테이너를 사용해 실행할 수 있다.  

공식 오라클 도커이미지 OCR 주소
* container-registry.oracle.com/database/free
* container-registry.oracle.com/database/enterprise
* container-registry.oracle.com/database/standard

<br>
상용버전이 아닌 테스트나 개인프로젝트 용도이므로 Free 버젼을 사용해야 한다.

```text
% docker pull container-registry.oracle.com/database/free:latest

% docker images
REPOSITORY                                    TAG       IMAGE ID       CREATED       SIZE
container-registry.oracle.com/database/free   latest    ef56a6ad07e8   5 weeks ago   13.2GB
```

<br>

### 도커 볼륨생성
작업내용이 컨테이너 내부에 저장되면 유실위험이 크고 딜레이와 충돌위험도 증가하기 때문에 도커 스토리지 세팅을 변경해 생성파일 위치를 컨테이너 내부가 아닌 로컬로 분리해줘야 한다. (후술 하겠지만 맥에서 이 로컬은 도커VM 이 된다)
  
DB운영용도로는 [볼륨 마운트][1]{:target="_blank"}를 사용해야하고 일반적인 개발용도로는 로컬의 개발소스를 컨테이너에서 빌드하고 실행 할 수 있는 바인드 마운트를 사용한다.  
컨테이너와 작업물들을 분리한다는 개념은 같지만 관리주체, 경로, 권한제어 방식의 차이로 생각하면 된다.  
* Volume Mount  : 도커가 추상화한 '독립영역' (컨테이너가 관리)
* Bind Mount    :  호스트와 직접 연결된 '실제 디렉토리' (호스트가 관리)
<table style="font-size: 14px;">
    <tr>
        <th></th>    
        <th>Volume Mount (도커볼륨)</th>
        <th>Bind Mount (직접마운트)</th>
    </tr>
    <tr>
        <td>생성위치</td>
        <td>Docker 관리 영역 /var/lib/docker/volumes/...</td>
        <td>로컬 PC의 지정 경로</td>
    </tr>
    <tr>
        <td>관리방식</td>
        <td>Docker가 생성/관리</td>
        <td>사용자(OS)가 직접 디렉토리 생성 및 관리</td>
    </tr>
    <tr>
        <td>백업.이동</td>
        <td>Docker 명령으로 쉽게 가능</td>
        <td>직접 파일 복사해야 함</td>
    </tr>
    <tr>
        <td>보안</td>
        <td>상대적으로 안전 (호스트 구조와 분리)</td>
        <td>호스트 파일 시스템 전체가 노출될 수 있음</td>
    </tr>
    <tr>
        <td>용도</td>
        <td>DB, 지속 저장소, 컨테이너 데이터 보관</td>
        <td>개발환경 공유, 소스코드 수정 반영</td>
    </tr>
    <tr>
        <td>성능</td>
        <td>최적화됨</td>
        <td>환경에 따라 성능이 떨어질 수 있음</td>
    </tr>
    <tr>
        <td>안정성</td>
        <td>매우 높음</td>
        <td>로컬 FS 의존, 오류 위험</td>
    </tr>
</table>

```text
% docker volume create oradata

% docker volume ls
DRIVER    VOLUME NAME
local     oradata
```
<br>

### 도커 컨테이너 생성
* --name : 컨테이너이름 
* -e ORACLE_PWD : 오라클 패스워드
* -p : 1521: 리스너포트, 5500: Em Express 포트 (웹용 관리자 인터페이스)
* -v : 컨테이너 내부에서 오라클이 데이터를 저장하는 주소  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<로컬의 볼륨 마운트 이름 : 컨테이너주소>, `--volume` 플래그와 동일)    
* -d : detached 모드로 실행 : 터미널을 차지하지 않고 백그라운드 실행

-v 옵션 이후에 < 볼륨명 : 컨테이너경로> 로 지정하면 볼륨마운트,  
<호스트의 로컬경로 : 컨테이너경로> 로 지정하면 해당 경로로 바인드 마운트가 된다. (파일로도 지정가능)
```text
% docker run --name oracle \
    -p 1521:1521 -p 5500:5500 \
    -e ORACLE_PWD=password \
    -v oradata:/opt/oracle/oradata \
    -d container-registry.oracle.com/database/free:latest
```

마운트 방식을 확인해보자

```text
% docker ps
... 실행중인 컨테이너ID 확인 (oracle)
... ps -a 옵션의 경우 모든 컨테이너 출력

% docker inspect oracle | grep Mounts -n5
      },
      "GraphDriver": {
          "Data": null,
          "Name": "overlayfs"
      },
      "Mounts": [
          {
              "Type": "volume",
              "Name": "oradata",
              "Source": "/var/lib/docker/volumes/oradata/_data",
              "Destination": "/opt/oracle/oradata",
```

도커 컨테이너로 접속해 SYS계정으로 로그인해보자
```text
% docker exec -it oracle bash

bash-4.4% sqlplus sys/password as sysdba

SQL*Plus: Release 23.26.0.0.0 - Production on Mon Dec 8 10:20:51 2025
Version 23.26.0.0.0

Copyright (c) 1982, 2025, Oracle.  All rights reserved.


Connected to:
Oracle AI Database 26ai Free Release 23.26.0.0.0 - Develop, Learn, and Run for Free
Version 23.26.0.0.0

SQL> select user from dual;

USER
----------------------------------------------------
SYS
```
<br>
오라클은 오래전에 맥을 포기했지만 SQL Developer 툴은 제공하고 있는데 혹시 라고 생각했다면 맞다 자바인것이다.  
VSCode용 플러그인도 있으니 VSCode의 Extensions 탭에서 검색 후 설치하고 설치 후 나타나는 SQL Developer 탭을 클릭, Containers 항목에서 컨테이너를 추가한 뒤 컨테이너 생성시 입력했던 정보들을 기입해준다.

<div class="row" style="display: flex; align-items: center;">
      <img src="/img/2025-12-08-running_oracle_in_mac01.png" style="width: 90%; height: auto;">
</div>
<br>

DB 커넥션이 만들어지고 정상 작동하는 것을 볼 수 있다.
<div class="row" style="display: flex; align-items: center;">
      <img src="/img/2025-12-08-running_oracle_in_mac02.png" style="width: 90%; height: auto;">
</div>
<br>

### 오라클 PDB 설정
설치된 오라클의 버전은 12C 이상인 멀티테넌트 방식이므로 CDB 컨테이너 안에 PDB를 생성해서 사용한다.  

```sql
-- PDB 생성
-- FILE_NAME_CONVERT = (pdbseed , destination)
-- pdbseed : PDB의 템플릿파일(.dbf)
-- destination : 템플릿 파일을 사용해 생성되는 PDB 데이터파일들이 저장되는 위치

CREATE PLUGGABLE DATABASE mypdb
  ADMIN USER mypdb_admin IDENTIFIED BY "1234"
  FILE_NAME_CONVERT = ('/opt/oracle/oradata/FREE/pdbseed', '/opt/oracle/oradata/FREE/mypdb');

-- CDB 에 존재하는 PDB 조회. MYPDB가 MOUNTED 상태임을 확인
SELECT name, con_id, open_mode FROM v$pdbs; 

-- PDB OPEN_MODE를 OPEN 으로 전환해야 접속할 수 있다
ALTER PLUGGABLE DATABASE mypdb OPEN;
-- ALTER PLUGGABLE DATABASE ALL OPEN; -- CDB 전체의 모든 PDB를 OPEN 할때는

-- CDB 기동시 PDB는 자동으로 열리지 않기 때문에 OPEN 상태로 유지한다
ALTER PLUGGABLE DATABASE mypdb SAVE STATE;


-- 생성된 PDB 를 확인
SELECT username, common, con_id
FROM cdb_users
ORDER BY con_id, username;

-- CDB -> PDB 컨테이너 전환
ALTER SESSION SET CONTAINER = mypdb;

-- 컨테이너를 PDB 로 전환하고 실행해야 MYPDB_ADMIN 유저가 검색됨.
-- MYPDB_ADMIN 의 common(공통유저) 컬럼 값은 NO 
SELECT username, common FROM dba_users;

-- MYPDB_ADMIN 에게 접속권한 부여
GRANT CREATE SESSION TO MYPDB_ADMIN;

-- 유저생성 권한
GRANT CREATE USER TO MYPDB_ADMIN;

-- 1. MYPDB_ADMIN 에게 롤 수여 권한만 줌(권장).
GRANT CREATE ROLE TO MYPDB_ADMIN;
GRANT GRANT ANY PRIVILEGE TO MYPDB_ADMIN;

-- 2. MYPDB_ADMIN 에게 DDL 권한도 함께 줌. (아래 권한을 직접 갖지 않아도 1번으로 개발자들에게 수여는 가능)
GRANT CREATE TABLE     TO MYPDB_ADMIN WITH ADMIN OPTION;
GRANT CREATE VIEW      TO MYPDB_ADMIN WITH ADMIN OPTION;
GRANT CREATE SEQUENCE  TO MYPDB_ADMIN WITH ADMIN OPTION;
GRANT CREATE PROCEDURE TO MYPDB_ADMIN WITH ADMIN OPTION;

-- 테이블스페이스 생성권한 (CDB가 생성해도 되지만 밑에서 MYPDB_ADMIN 이 직접생성)
GRANT CREATE TABLESPACE TO MYPDB_ADMIN; 
GRANT UNLIMITED TABLESPACE TO MYPDB_ADMIN;  -- 테이블스페이스 용량제한 해제
GRANT ALTER TABLESPACE TO MYPDB_ADMIN;
-- GRANT DROP TABLESPACE TO MYPDB_ADMIN;

-- PDB -> CDB 빠져나옴
ALTER SESSION SET CONTAINER = CDB$ROOT
```
<br>

<br>
터미널에서 컨테이너로 접속해 생성한 MYPDB_ADMIN 계정으로 로그인해 보자.

```text
% docker exec -it oracle bash
ash-4.4$ sqlplus mypdb_admin/1234@localhost:1521/mypdb;

SQL*Plus: Release 23.26.0.0.0 - Production on Wed Dec 10 04:59:26 2025
Version 23.26.0.0.0

Copyright (c) 1982, 2025, Oracle.  All rights reserved.

Last Successful login time: Wed Dec 10 2025 04:57:33 +00:00

Connected to:
Oracle AI Database 26ai Free Release 23.26.0.0.0 - Develop, Learn, and Run for Free
Version 23.26.0.0.0

SQL> show user;
USER is "MYPDB_ADMIN"

SQL> SHOW con_name;

CON_NAME
------------------------------------
MYPDB

SQL>select username from dba_users where username like 'MYPDB_ADMIN'; -- 생성된 PDB ADMIN 유저가 조회된다.

```

<br>

MYPDB_ADMIN 유저로 고정길이의 테이블 스페이스를 추가해보자.
```text
SQL>
CREATE TABLESPACE ORA_SQL_TEST_TS DATAFILE '/opt/oracle/oradata/FREE/mypdb/ora_sql_test.dbf' SIZE 10G
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;

-- 생성한 테이블 스페이스 삭제
-- DROP TABLESPACE ORA_SQL_TEST_TS INCLUDING CONTENTS AND DATAFILES CASCADE CONSTRAINTS;
```

MYPDB_ADMIN 자신의 디폴트 테이블 스페이스는 SYSTEM으로 조회된다.
```text
SQL> select username, default_tablespace from user_users;

USERNAME
--------------------------------------------------------------------------------
DEFAULT_TABLESPACE
------------------------------
MYPDB_ADMIN
SYSTEM

```

CDB 관리자인 SYS로 접속하여 MYPDB_ADMIN의 기본테이블 스페이스를 변경해준다.
```sql
SQL> ALTER USER MYPDB_ADMIN DEFAULT TABLESPACE ORA_SQL_TEST_TS;

User MYPDB_ADMIN altered.

SQL>
SELECT username, default_tablespace FROM dba_users where username like 'MYPDB_ADMIN' ORDER BY username;
```
<br>

오라클 컨테이너에서 ora_sql_test.dbf 파일을 확인해보자.
```text
% docker exec -it oracle bash
bash-4.4$ cd /opt/oracle/oradata/FREE/mypdb
bash-4.4$ pwd
/opt/oracle/oradata/FREE/mypdb
bash-4.4$ ls
ora_sql_test.dbf  sysaux01.dbf	system01.dbf  temp01.dbf  undotbs01.dbf
```

```text
% docker inspect oracle | grep Mounts -n5
      },
      "GraphDriver": {
          "Data": null,
          "Name": "overlayfs"
      },
      "Mounts": [
          {
              "Type": "volume",
              "Name": "oradata",
              "Source": "/var/lib/docker/volumes/oradata/_data",
              "Destination": "/opt/oracle/oradata",
```
<br>

* Type (마운트타입) : 볼륨 마운트  
* Source (호스트 경로):  Docker 가 사용하는 로컬호스트 내부의 실제 디스크 경로   
* Destination (컨테이너 경로): 생성한 오라클 컨테이너 내부에서 데이터파일, 제어파일, redo log 등이 저장되는 위치로 오라클 프로세스가 실제로 읽고 쓰는 경로   

예를 들어 컨테이너 내부에서 오라클이 test.dbf 파일을 생성했다고 가정하면 경로는  
`Destination : /opt/oracle/oradata/FREE/test.dbf`   

그러나 실제 물리적 파일은 컨테이너 외부의 로컬위치에 생성된다.  
`Source : /var/lib/docker/volumes/oradata/_data/FREE/test.dbf`

<br>
컨테이너 내부 Destination 경로에 위치한 파일들은 실제 로컬 source 경로의 파일과 연동되어 있다.  
이 파일들은 컨테이너 내부의 오라클에 의해 관리되고 있으니 직접 조작할 수 없고 DB 명령어를 통해서만 수정되어야 한다.  

여기서 알아둬야 할 점    
* 맥에서 구동되는 Docker는 내부적으로 리눅스 VM 위에서 실행되므로, VM 내부 Source 경로(/var/lib/docker/volumes/...)는 macOS에서 직접 탐색불가 (경량화된 VM이므로 완전한 독립 VM과는 다름)
* Linux 네이티브 환경에서는 Source 경로가 호스트 파일 시스템에 실제로 존재함  
* 따라서 볼륨 마운트된 VM Source 경로를 확인하려면 Alpine 컨테이너와 연결해서 간접 확인만 가능함 (볼륨 방식이 아닌 바인드 마운트 방식은 맥 호스트에서도 리눅스 환경과 동일하게 작동함)

<span style="color:red"><b>이래서 개발자는 리눅스를 써야한다</b></span>  
유닉스에 대한 로망 때문에 맥을 사용중이지만 iOS 개발만 아니었다면 진작 리눅스머신을 꾸렸을 것 같다.  
특히 램에 인색한 맥을 사용하면서 가상화로 인해 탁하게 변해가는 메모리압박 그래프와 스왑용량을 보는것은 기분이 썩 좋지않은 경험이다...  

데이터전용 컨테이너를 만들어서 오라클에서 생성되는 데이터를 별도의 컨테이너로 관리하는 멀티 컨테이너 구조를 사용할 수도 있을 듯 하지만 개인 프로젝트 용도로는 이정도면 충분하다.  아니면 도커 볼륨 (리눅스 VM) 에 생성되는 데이터를 정기적으로 맥으로 백업해 보관하는 방식도 생각해 볼 수 있고 이러한 경우 맥에서 파일을 읽기 전용으로 설정해둘 필요가 있을 것이다.  
컨테이너 내부의 Destination 경로에 있는 ora_sql_test.dbf 파일을 확인했으니 이번에는 호스트의 Source 경로에서 해당 파일을 확인해보자.  
Alpine 컨테이너 볼륨을 통해 리눅스VM 호스트에 위치한 ora_sql_test.dbf 파일을 확인할 수 있다. 
```text
// -v <VM Source 경로>:<Alpine 컨테이너 경로>

% docker run --rm -it -v /var/lib/docker/volumes:/vol alpine sh
/ # cd vol/oradata/_data/FREE/mypdb
/vol/oradata/_data/FREE/mypdb # ls
ora_sql_test.dbf  sysaux01.dbf      system01.dbf      temp01.dbf        undotbs01.dbf
/vol/oradata/_data/FREE/mypdb #
```

운영 서버가 아닌 개인작업시 작업이 끝나면 오라클 컨테이너를 종료해 데이터 유실을 방지하고  
`% docker stop oracle`

다시 컨테이너를 재시작할때는 간단하게 아래와 같이 입력해 실행.  
`% docker start oracle`
<br>

MYPDB_ADMIN 계정도 SQL Developer 커넥션을 추가해주자.
<div class="row" style="display: flex; align-items: center;">
      <img src="/img/2025-12-08-running_oracle_in_mac03.png" style="width: 90%; height: auto;">
</div>
<br>

Service Name 은 오라클을 로컬에서 설치할때 흔히 설정하던 SID 값 (인스턴스의 고유명) 을 말한다.  
`CREATE DATABASE ORCLCDB ...` 로 새로운 데이터베이스 생성시 ORCLCDB ,
PDB 생성 명령어인 `CREATE PLUGGABLE DATABASE mypdb` 의 mypdb 가 해당된다.   
로컬에서 수행하는 테스트가 아닌 상용시스템이나 원격 연결, 도커 사용시에는 Service name 사용이 권장된다.  
도커로 다운받은 FREE 버젼의 CDB 인스턴스명은 기본적으로 FREE 로 설정 되어있다.  

터미널 로그인 예시  
`sqlplus sys/password@localhost:1521/FREE as sysdba`     
`sqlplus mypdb_admin/1234@localhost:1521/mypdb`  

이제 생성된 MYPDB_ADMIN 으로 접속, 사용할 업무용 유저를 생성, 업무용 유저로 테이블을 생성해서  
오라클 11g 이전 버전처럼 동일하게 사용하면 된다.  MYPDB_ADMIN 은 PDB 의 SYS 인셈.  
참고로 SYS, SYSTEM 계정은 CDB 뿐만이 아니라 PDB 생성시 공통유저로 존재하며   
별도의 어드민을 MYPDB_ADMIN 과 같이 생성해 사용하는 것이 권장된다.  

공통유저 : 컨테이너 마다 존재하지만 하나의 유저. 다만 컨테이너 단위로만 영향력을 가짐.  
        건들지 말자. 쓸일이 없다....

MYPDB_ADMIN 로 PDB에 로그인하고 개발자용 유저를 생성하고 관련 권한을 부여하자.  
```sql
-- 개발자 생성
CREATE USER DEV_USER1 IDENTIFIED BY DEV_USER1;

-- 로그인 권한
GRANT CREATE SESSION TO DEV_USER1;


GRANT CREATE SESSION TO 
-- MYPDB_ADMIN 이 개발자에게 수여할 DDL 롤과 권한
CREATE ROLE DEV_DDL_ROLE;
GRANT CREATE TABLE 
    ,CREATE VIEW
    ,CREATE SEQUENCE
    ,CREATE PROCEDURE TO DEV_DDL_ROLE;

GRANT DEV_DDL_ROLE TO DEV_USER1;
```

<br>


[1]: https://docs.docker.com/engine/storage/volumes/