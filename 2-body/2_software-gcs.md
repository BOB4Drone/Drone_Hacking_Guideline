
# GCS(Ground Control System) <!-- omit in toc -->

## 목차 <!-- omit in toc -->

- [1. 개요](#1-개요)
    - [1.1. GCS 개념](#11-GCS-개념)
    - [1.2. GCS 기능](#12-GCS-기능)
    - [1.3. 드론별 GCS 현황](#13-드론별-gcs-현황)
    - [1.4. ELF vs AppImage](#14-elf-vs-appimage)
- [2. 분석 환경 구축](#2-분석-환경-구축)
    - [2.1. QGC 빌드](#21-qgc-빌드)
      - [2.1.1. QT 설치](#211-qt-설치)	
      - [2.1.2. GUI를 이용한 빌드](#212-gui를-이용한-빌드)
      - [2.1.3. CLI를 이용한 빌드](#213-cli를-이용한-빌드)
    - [2.2. Sanitizer 사용](#22-sanitizer-사용)
      - [2.2.2. GUI 이용](#221-gui를-이용한-sanitizer-사용)
      - [2.2.3. CLI 이용](#222-cli를-이용한-sanitizer-사용)
- [3. 취약점 분석 방법론](#3-취약점-분석-방법론)
    - [3.1. 소스코드 오디팅을 통한 취약점 분석](#31-소스코드-오디팅을-통한-취약점-분석)
      - [3.1.1. MAVLINK Handle 함수 오디팅](#311-mavlink-handle-함수-오디팅)
      - [3.1.2. QGC GUI 기능 함수 오디팅](#312-qgc-gui-기능-함수-오디팅)
    - [3.2. Fuzzing을 통한 취약점 분석](#21-qgc-빌드)
      - [3.2.1. Mavlink Fuzzer](#321-mavlink-fuzzer)
      - [3.2.2. QGC With AFL++](#322-qgc-with-afl)

## 1. 개요

### 1.1. GCS 개념

GCS란 지상 관제 시스템(Ground Control Station)의 줄임말으로 드론의 제어, 미션 등 다양한 운영을 하기 위하여 만든 소프트웨어이다. (PX4에서는 [QGroundControl](https://github.com/mavlink/qgroundcontrol)의 사용이 권장된다.)


### 1.2. GCS 기능

  - 자율비행 및 실시간 비행경로 변경 관제 가능
  - 지도 데이터 처리, 비행경로 사전 분석 기능
  - 지오펜스 기능 (필요 시 비행금지 지역 생성)

### 1.3. 드론별 GCS 현황
|ardupilot|px4|dji|
|------|---|---|
|mission planner|qground control|ground station|


### 1.4. ELF vs AppImage
우리는 QGC의 취약점 분석에 앞서 메모리 보호기법을 확인하기 위해 Release파일인 [QGroundControl.AppImage](https://docs.qgroundcontrol.com/master/en/releases/release_notes.html)를 checksec.sh을 이용해 확인하였다.   
![Untitled](https://user-images.githubusercontent.com/91944211/145853059-f851a947-f62a-4850-8d73-2fb56cf68cea.png)

NX를 제외한 모든 보호기법이 꺼져있는것을 볼 수 있는데, 이는 AppImage확장자의 특성 때문이다.
![Untitled (1)](https://user-images.githubusercontent.com/91944211/145853378-2a6ecd6c-15ab-4e3a-b997-85b34383104e.png)

다음 그림과 같이 AppImage는 실행 바이너리와 종속성 충족을 위한 라이브러리들이 squashfs형태로 함께 저장되어 있다. 따라서 우리는 `sudo mount ./qgroundcontrol.AppImage mountpoint -o offset=188392` 명령어를 통해 실제로 빌드된 바이너리를 확인하였고, 다시 메모리 보호기법을 확인했다.   
![Untitled (2)](https://user-images.githubusercontent.com/91944211/145854132-68ed37d7-623e-47ce-803f-bf079a7cef12.png)
![Untitled (3)](https://user-images.githubusercontent.com/91944211/145854403-426d88d4-87e9-46fb-b1d0-ef62d9f02d1d.png)

## 2. 분석 환경 구축

### 2.1. QGC 빌드

#### 2.1.1. QT 설치

QGC는 QT 라이브러리를 Cross-Platform 라이브러리로 사용하기 때문에 QT를 다운받아주고 QT Creator를 통해 컴파일한다.

1. [Qt 온라인 설치 프로그램](https://www.qt.io/download-open-source) 다운로드 및 실행

2. QT 버전(5.15.2) 및 구성 요소 설정
![image](https://user-images.githubusercontent.com/91944211/145797702-876c007d-83f1-481f-a01d-ef3b9fded745.png)

3. 추가 패키지 설치
    ```
    sudo apt-get install speech-dispatcher libudev-dev libsdl2-dev patchelf
    ```
4. 서브모듈을 포함해 레포지토리를 클론
    ```
    git clone --recursive -j8 https://github.com/mavlink/qgroundcontrol.git
    ```
2. 서브모듈 업데이트
    ```
    git submodule update --recursive
    ```
#### 2.1.2. GUI를 이용한 빌드

1. QT Creator로 qgroundcontrol project open

2. 망치 모양을 클릭해 빌드 진행

    ![image](https://user-images.githubusercontent.com/91944211/145805224-3b2287ba-9d1f-4baa-85c4-71be54e58e66.png)

#### 2.1.3. CLI를 이용한 빌드
1. makefile 생성
    ```
    cd qgroundcontrol
    mkdir build
    cd build
    ~/Qt/5.15.2/gcc_64/bin/qmake ../
    ```
2. make를 통한 빌드
    ```
    make
    ```
### 2.2. Sanitizer 사용

#### 2.2.1. GUI를 이용한 Sanitizer 사용

1. qt creator에서 Projects > Build > Build Steps > qmake > Additional arguments 진입

2. 아래와 같아 Argument 수정

    ![image](https://user-images.githubusercontent.com/91944211/145806129-c2a81e84-2da1-4b68-9b99-10b8a1663711.png)


#### 2.2.2. CLI를 이용한 Sanitizer 사용

1. makefile 생성
    ```
    cd qgroundcontrol
    mkdir build
    cd build
    ~/Qt/5.15.2/gcc_64/bin/qmake ../
    ```
2. makefile의 CC, CXX, LINK에 Sanitizer 옵션 추가

    ![image](https://user-images.githubusercontent.com/91944211/145806760-400381d7-a375-491d-b071-091c9bef09e5.png)


## 3. 취약점 분석 방법론

### 3.1. 소스코드 오디팅을 통한 취약점 분석

#### 3.1.1. MAVLINK Handle 함수 오디팅
QGC 소스 디렉토리에서 `src/Vehicle/Vehicle.cc`의 [593line](https://github.com/mavlink/qgroundcontrol/blob/9718d3c71a57034fdaedc1a0aea355c4fd6c2506/src/Vehicle/Vehicle.cc#L593)를 보면 Vehicle::_mavlinkMessageReceived() 라는 함수에서 mavlink packet을 핸들링한다. 따라서 해당 함수의 흐름을 따라가며 오디팅을 진행하였다.


#### 3.1.2. QGC GUI 기능 함수 오디팅
QGC 프로그램에서 GUI로 구현된 기능들의 함수를 분석하여 취약점을 찾을 수 있었다. 
- [#Issue10035](https://github.com/mavlink/qgroundcontrol/issues/10035) 
- [#Issue10059](https://github.com/mavlink/qgroundcontrol/issues/10059)
- [#Issue10068](https://github.com/mavlink/qgroundcontrol/issues/10068)

### 3.2. Fuzzing을 통한 취약점 분석

#### 3.2.1. Mavlink Fuzzer
QGC도 PX4와 같은 mavlink protocol을 이용하기 때문에 [기존에 만든 퍼저](https://github.com/BOB4Drone/4D-Fuzzer)를 이용해 3개의 함수에서 취약점을 찾을 수 있었다.   

#### 3.2.2. QGC With AFL++
코드커버리지 기반의 퍼징을 위해 AFL++을 사용하였다. 먼저 AFL++은 기본적으로 파일 I/O를 이용해 퍼징을 수행하기 때문에 `afl-forkserver.c`를 다음과 같이 UDP를 사용하도록 바꿔주었다.   
```
int client_socket;
    struct sockaddr_in serverAddress;



    memset(&serverAddress, 0, sizeof(serverAddress));
    

    serverAddress.sin_family = AF_INET;
    inet_aton("127.0.0.1", (struct in_addr*) &serverAddress.sin_addr.s_addr);
    serverAddress.sin_port = htons(14540);
    serverAddresss.sin_family = AF_INET;


    // socket create
    if ((client_socket = socket(PF_INET, SOCK_DGRAM, 0)) == -1)
    {
        printf("socket create fail\n");
        exit(0);
    }


    sendto(client_socket, buf, len, 0, (struct sockaddr*)&serverAddress, sizeof(serverAddress));

    // socket close
    close(client_socket);
```   
이후 Makefile을 수정해 컴파일러를 afl-g++로 바꿔주어 퍼징을 진행했다. 하지만 이렇게 진행할경우 세 가지 문제점이 발생한다.   
1. QGC가 켜지기 전에 AFL에서 먼저 패킷을 보낸다.
2. HeartBeat를 받지않으면 다른 패킷을 받지 않는다.
3. AFL은 프로그램이 종료되어야 계측을 할 수 있다.

우리는 문제들을 해결하기 위해 다음과 같은 방법을 이용했다.   

![Untitled Diagram drawio (1) (1)](https://user-images.githubusercontent.com/91944211/145868099-68bcb159-9b80-442d-838c-db0df9d6bfa5.png)   
먼저 첫번째 문제 해결을 위해 다음과 같은 UDP Server의 중계를 거쳐 QGC의 소켓이 로드된 후에 하트비트 패킷과 생성된 패킷을 보내도록 하였다.   

```
if (heatbeatFlag){exit(0);}
heatbeatFlag = 1
```
다음으로 두번째와 세번째 문제 해결을 위해 src/Vehicle/Vehicle.cc에서 `_uas->receiveMessage(message);` 밑에 해당 코드를 추가하여 하트비트 및 생성된 패킷을 받으면 QGC가 종료되도록 하였다.

![Untitled (5)](https://user-images.githubusercontent.com/91944211/145869720-87e71283-3b88-4a4b-92d4-c45491cb6555.png)   
위의 두가지 방법으로 퍼징을 진행한 결과 다음과 같이 120개의 uniq행을 찾을 수 있었지만, 크래시는 나오지 않았다. 왜냐하면 해당 방식은 full packet을 뮤테이트 하기 때문에 mavlink의 crc검증 루틴을 통과할 수 없다.   

![GCS_fuzzing drawio](https://user-images.githubusercontent.com/91944211/145870675-c787b919-a846-40fa-afb0-906f3c042b93.png)   
때문에 우리는 UDP 중계서버의 코드를 수정해 다음과 같은 구조로 퍼징을 다시 진행했다.   
해당 구조는 AFL++에서 Mavlink의 'payload' 부분만 뮤테이트해, UDP Server에서 CRC값이 포함된 full packet을 전송한다.   

![Untitled (6)](https://user-images.githubusercontent.com/91944211/145870720-2ba6f5fe-1ce6-47aa-a177-1ec67268d062.png)   
그 결과 다음과 같이 많은 crash를 얻을 수 있었다. 해당 crash들은 모두 위의 4dfuzzer에서 찾은 취약점이었지만, 해당 방식을 응용해 future work로써 유용히 쓰일 수 있을것으로 보인다.


## 카테고리 <!-- omit in toc -->

### 소개 <!-- omit in toc -->
   1. [연구배경 및 목표](/1-intro/about-drone-research.md)
   2. [선행 연구](/1-intro/related-work.md)

### 접근 방법론 <!-- omit in toc -->
   1. [무인항공기(UAV) 소프트웨어](/2-body/1_software-uav.md)
      1. [Flight controller software](/2-body/1_software-uav.md/#1-fcsflight-controller-software)
      2. [Nuttx RTOS](/2-body/1_software-uav.md/#2-nuttx-rtos)
   2. [지상 관제(GCS) 소프트웨어](/2-body/2_software-gcs.md/)
   3. [하드웨어](/2-body/3_hardware.md)
       1. [GPS 모듈](/2-body/3_hardware.md/#1-gps-모듈)
       2. [PX4 Optical Flow](/2-body/3_hardware.md/#2-px4-optical-flow)
       3. [PX4 Telemetry Radio](/2-body/3_hardware.md/#3-px4-telemetry-radio)
       4. [Wifi 모듈](/2-body/3_hardware.md/#4-wifi-모듈)

### 결과 <!-- omit in toc -->
   1. [프로젝트 성과](/3-conclusion/result.md)
   2. [프로젝트 후기](/3-conclusion/conclusion.md)

