---
title: Mac에서 Docker 도커를 사용해 빌드하기
titleEn: 
author: BabyK
date: 2025-12-25
category: Web
layout: post
tags: [Web, Docker]
published: true
---
<br>
### 도커를 사용한 운영배포 시뮬레이션
도커를 사용해 리눅스 컨테이너 안에서 빌드 및 테스트를 진행하여 운영서버 반영전의 개발단계를 시뮬레이션 해보자.   
* 운영 환경과 똑같은 리눅스 이미지를 사용한 로컬 컨테이너 사용
* 맥에서는 코드 오류를 확인하는 컴파일까지 진행하고 빌드는 컨테이너에서 실행하여 로컬 테스트를 진행  
* 개발 호스트에서 빌드이후 바이너리 소스나 빌드 결과물만 운영 서버로 배포하고 실행함
* 빌드 소스들은 도커 이미지로 변환후 도커허브를 통해 운영 배포
<br>

### 주의할 점
* 바인드 마운트된 파일들은 컨테이너를 통해 호스트 파일 시스템에 쓰기 권한을 갖게 되므로 보안 취약점이 될 수 있다.  
* 리모트 환경에서 도커 데몬을 사용하고 있다면 당연히 클라이언트가 아닌 데몬이 실행되고 있는 리모트 호스트 서버 위치의 경로와 컨테이너가 사이에 바인드가 생성된다.
* 운영환경에서는 바이너리 파일이나 설정파일의 조작을 막기위해 read-only로 컨테이너를 실행할 수 있다(빌드 결과물만 신뢰한다는 구조가 됨).
<br>

### 도커 설정
바인드 마운트 생성시 `--mount` `--volume` 두 가지 플래그를 사용할 수 있는데
일반적으로 `--mount` 명령어와 각종 옵션을 사용한 사용이 권장된다.  
참고로 `docker run -v` 와 `docker run --volume` 은 같은 의미이고 *Volume* 은 [볼륨 마운트] [2]{:target="_blank"} 와 바인드 마운트에 상관없이 컨테이너 외부의 스토리지를 뜻한다.  

서비스를 배포할때 사용하는 리눅스 이미지는 Alpine 이나 Debian Slim, Minimal Ubuntu 와 같은 경량 리눅스가 주로 사용되는데 Alpine 과 나머지 둘은 사용하는 라이브러리상에 차이점이 존재한다.  
RHEL or DEBIAN 계열의 리눅스는 전통적인 GNU C Library 를 사용하며 하위 호환성과 안정성에 강점이 있지만 코드 규모가 크고 보안적인 취약점도 늘어나는 단점이 있다. Alpine 이 사용하는 musl libc 는 코드 규모가 작은반면 glibc 전용 바이너리 실행이 불가하기 때문에 명확하고 예측가능한 목표의 서비스를 위해 사용해야 한다. 따라서 Alpine 은 언어 자체에 라이브러리를 포함해 OS 와 분리되어 있는 (Static Library) GO 나 RUST 같은 언어를 사용한 서비스에서 강점을 갖는다.  
<br>

 [Minimal Ubuntu][1]{:target="_blank"} 의 최신버전을 사용해보자.  
 
```text
... 우분투 이미지 다운로드
% docker pull ubuntu:latest
% docker images
REPOSITORY                                    TAG       IMAGE ID       CREATED        SIZE
ubuntu                                        latest    c35e29c94501   2 months ago   141MB

... 로컬에 개발경로 생성
% mkdir docker_dev
% pwd
/Users/babyk/workspace/docker_dev

... 로컬 개발경로를 컨테이너상의 /workspace 와 연결
    --name 컨테이너명을 지어줌
    --rm 컨테이너를 종료시 자동삭제 (볼륨 마운트와 달리 컨테이너를 남겨둘 필요가 없기때문에)
    --it 명령어 입력과 셸 접속 가능 (컨테이너 사용시 사실상 필수 옵션)
    -v <현재 로컬 작업경로 : 컨테이너 경로>
    -w 컨테이너에 초기 진입 경로

% docker run --name ubuntu_dev --rm -it -v "$(pwd)":/workspace -w /workspace ubuntu:latest bash

... 컨테이너안의 /workspace 로 진입
% root@744a5b166823:/workspace# pwd
/workspace

... 터미널을 하나더 열어서 로컬의 작업경로에서 텍스트 파일을 하나 생성해주면
/Users/babyk/workspace/docker_dev % echo "Hello from docker_dev path" > hello.txt

... 컨테이너 상에서도 파일확인 
% Ls
hello.txt
```
<br>

컨테이너에서 운영 빌드환경을 시뮬레이션 해보자.
```text
... 로컬에서 유저의 UID 와 GID 를 출력하는 코드를 작성하고
/Users/babyk/workspace/docker_dev % cat test.c

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    uid_t uid = getuid();   // 현재 사용자 UID
    gid_t gid = getgid();   // 현재 그룹 GID

    printf("Current UID: %d\n", uid);
    printf("Current GID: %d\n", gid);

    return 0;
}

...minimal ubuntu 컨테이너에는 gcc가 없기 때문에 설치
% root@744a5b166823:/workspace# apt update
% root@744a5b166823:/workspace# apt install -y gcc

...설치가 끝나면 컴파일한다. 바이너리 파일 생성. 실행.
% root@744a5b166823:/workspace# gcc -o test test.c
% root@744a5b166823:/workspace# ./test
Current UID: 0
Current GID: 0
```
<br>

컨테이너에서 사용하는 명령어는 도커 데몬이 실행되고 있는 호스트의 커널을 이용하기 때문에  
UID 와 GID 가 0 번 root 권한을 갖게되면 컨테이너를 통한 호스트의 시스템을 조작이 가능하게 된다.  

readonly 를 사용해서 컨테이너에서 호스트를 조작할 수 있는 보안 문제를 차단할 수 있고
docker run 명령어 입력시 `-u <UID:GID>` 옵션을 사용해 UID:GID 를 설정해 접근을 차단할 수도 있다.  

readonly 로 바인드 마운트를 테스트해 보자.
```text
/Users/babyk/workspace/docker_dev % docker run --name ubuntu_dev --rm -it -v "$(pwd)":/workspace:ro -w /workspace ubuntu:latest bash

root@c0c18294b6f8:/workspace# ls
hello.txt  test  test.c

... 삭제 및 컴파일도 불가
root@c0c18294b6f8:/workspace# rm test
rm: cannot remove 'test': Read-only file system

root@c0c18294b6f8:/workspace# gcc -o test2 test.c
/usr/bin/ld: cannot open output file test2: Read-only file system
```
<br>
<br>


[1]: https://hub.docker.com/_/ubuntu
[2]: /posts/web/2025-12-08-running_oracle_in_mac/#도커-볼륨생성

