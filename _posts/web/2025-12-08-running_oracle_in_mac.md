---
title: Mac에서 Oracle 실행하기
titleEn: Running Oracle on macOS
author: BabyK
date: 2025-12-08
category: Web
layout: post
tags: [Web, Docker. Oracle]
published: true
---

오라클은 맥을 지원하지 않지만 도커 컨테이너를 사용해 실행가능하다.  

```text
공식 오라클 도커이미지 OCR 주소
container-registry.oracle.com/database/free
container-registry.oracle.com/database/enterprise
container-registry.oracle.com/database/standard
```
<br>
상용버전이아닌 테스트나 개인프로젝트 용도로는 Free 버젼을 사용해야한다.

```text
% docker pull container-registry.oracle.com/database/free:latest

% docker images
REPOSITORY                                    TAG       IMAGE ID       CREATED       SIZE
container-registry.oracle.com/database/free   latest    ef56a6ad07e8   5 weeks ago   13.2GB
```

<br>
다운받은 그대로 컨테이너를 실행해 사용하면 작업내용이 컨테이너 내부에 저장되어 컨테이너와 함께 유실될 위험이 크고  
속도문제와 충돌위험도 증가하기 때문에 도커 스토리지 세팅을 변경해 생성파일 위치를 컨테이너 내부가 아닌 로컬로 분리해줘야 한다.  
DB의 경우 [볼륨 마운트][1]{:target="_blank"}를 사용하고 개발자가 로컬의 개발소스를 컨테이너에서 빌드하고 실행 하는 용도로는 바인드 마운트를 사용한다.
컨테이너와 작업물들을 분리한다는 개념은 같지만 관리주체(컨테이너냐 OS냐), 경로, 권한제어 방식의 차이로 생각하는 것이 좋다.  
* 볼륨 : 도커가 추상화한 '독립영역'
* 바인드 :  호스트와 직접 연결된 '실제 디렉토리'  
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

<br>
볼륨 생성

```text
% docker volume create oradata

% docker run --name oracle \
    -p 1521:1521 -p 5500:5500 \
    -e ORACLE_PWD=password \
    -e INIT_SGA_SIZE=3000 \
    -e INIT_PGA_SIZE=1000 \
    -v oradata:/opt/oracle/oradata \
    -d container-registry.oracle.com/database/free:latest

--name : 컨테이너이름 
-e ORACLE_PWD : 오라클 패스워드
-e INIT_SGA/PGA : 초기 글로벌 공간 용량
-p : 1521: 리스너포트, 
     5500: Em Express 포트 (웹용 관리자 인터페이스)
-v : 로컬 볼륨 저장주소

% docker volume ls
DRIVER    VOLUME NAME
local     oradata
```

<br>
마운트 방식을 확인

```text
% docker ps
... 컨테이너ID 확인 (oracle)

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
<Br>

컨테이너 ID로 접속해 SYS계정 로그인 확인
```text
% docker exec -it oracle bash

컨테이너접속
bash-4.4% sqlplus sys/password as sysdba
```
<br>

로그인 성공
```text
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
오라클은 오래전에 맥을 포기했지만 SQL Developer 툴은 제공하고 있는데 혹시 라고 생각했다면 맞다.  
자바인것이다. 알뜰한 사람들 같으니.  
VSCode용 플러그인을 사용하는 것이 더 편리하니 VSCode의 Extensions 탭에서 검색 후 설치하고  
설치 후 나타나는 SQL Developer 탭을 클릭, Containers 항목에서 컨테이너를 추가, CLI에서 컨테이너 생성시 입력한  
컨테이너명(oracle) 과 패스워드(password)를 입력한다.  

<div class="row" style="display: flex; align-items: center;">
      <img src="/img/2025-12-08-running_oracle_in_mac01.png" style="width: 90%; height: auto;">
</div>
<br>

DB커넥션이 만들어지고 정상 작동 하는것을 볼 수 있다.
<div class="row" style="display: flex; align-items: center;">
      <img src="/img/2025-12-08-running_oracle_in_mac02.png" style="width: 90%; height: auto;">
</div>
<br>

테이블 스페이스를 생성하고 볼륨 위치에 생성된 파일을 확인해보자.  
<div class="row" style="display: flex; align-items: center;">
      <img src="/img/2025-12-08-running_oracle_in_mac03.png" style="width: 90%; height: auto;">
</div>

<br>

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

* Source (호스트 경로)):  Docker 가 사용하는 Linux 내부의 실제 디스크 경로   
* Destination (컨테이너 경로)): 생성한 오라클 컨테이너 내부에서 데이터파일, 제어파일, redo log 등이 저장되는 위치로 오라클 프로세스가 실제로 읽고 쓰는 경로   

예를 들어 오라클이 test.dbf 파일을 생성했다고 가정하면 컨테이너 내부에서 Oracle이 만든 경로는  
/opt/oracle/oradata/ORCL/test.dbf  
  
그러니 실제 물리적 파일은 아래의 로컬위치에 생성된다.  
/var/lib/docker/volumes/oradata/_data/ORCL/test.dbf  
  
컨테이너 내부의 Destination 파일은 실제로 생성되는 것이 아니라 도커에 의해 실제 로컬의 물리위치인 Source에 연결되어 있는것 뿐이다.  

여기서 알아둬야 할 점은  
* 맥에서 구동되는 도커의 경우 리눅스 VM위에서 실행되며 따라서 맥의 로컬에서 파일을 찾을 수는 없다.
* 즉 Linux 네이티브로 돌아가는 도커라면 Source 위치에 실제 물리적 파일이 존재하겠지만  
* 맥은 그 Linux VM에 접속해도 Source 위치에 존재하는 실제 물리적 파일의 확인은 불가능 (완전한 VM이 아님), Source 와 연결된 볼륨 데이터의 확인만 가능하다.  
  
<span style="color:red"><b>이래서 개발자는 리눅스를 써야한다</b></span>  
유닉스에 대한 로망때문에 맥을 사용중이지만 iOS 개발만 아니었다면 진작 리눅스머신을 꾸렸을 듯.  
특히 램에 인색한 맥을 사용하면서 가상화로 인해 탁하게 변해가는 메모리압박 그래프와 스왑용량을 보는것은 기분이 썩 좋지않은 경험이다...  

아무튼 볼륨을 들여다보면 ora_sql_test.dba 파일을 찾을 수 있다. 
```text
% docker run --rm -it -v /var/lib/docker/volumes:/vol alpine sh
% cd vol/oradata/_data
% ls
FREE              dbconfig          ora_sql_test.dba
```
<br>

<br>

[1]: https://docs.docker.com/engine/storage/volumes/