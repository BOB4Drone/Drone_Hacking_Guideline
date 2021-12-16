# Drone-Hacking-Guideline

**KITRI BoB 10기 취약점 분석 Track : 프로젝트 '드론에 대한 취약성 연구'**

> 연구 기간: 2021년 9월 ~ 2021년 12월(약 4개월)

본 프로젝트는 KITRI BoB(Best of the Best) 10기에서 진행한 프로젝트로, 3개월동안 드론에 대한 취약성 연구를 진행한 내용을 기록하였습니다. 

저희 팀이 3개월동안 분석한 결과를 공유하고, 새롭게 드론 해킹에 입문하는 분들에게 진입장벽을 낮추고자 합니다.

시장 점유율이 높은 드론 소프트웨어를 대상으로 취약점 분석 실험을 했고, 이를 통해 유의미한 취약점들을 식별했습니다.
또한 해당 내용을 신속하게 조치할 수 있도록 제보하여 해당 이슈에 대한 패치 작업을 완료하였습니다.

※ 4DFUZZER [(PublicRepository Link)](https://github.com/BOB4Drone/4D-Fuzzer)

※ English documents are [here](https://github.com/BOB4Drone/Drone_Hacking_Guideline_ENG)


---

### 저자

> KITRI BoB 10기 취약점분석 트랙 4-Drone 팀

- Mentor: 박천성, 강인욱
- PL(Project Leader): 원요한
- Mentee: 이준오 PM, 경규창, 김동현, 류형호, 최수빈, 김지수

## 카테고리 <!-- omit in toc -->

### 소개 <!-- omit in toc -->
   1. [연구배경 및 목표](/1-intro/about-drone-research.md)
   2. [선행 연구](/1-intro/related-work.md)

### 접근 방법론 <!-- omit in toc -->
   1. [무인항공기(UAV) 소프트웨어](/2-body/1_software-uav.md)
      1. [Flight controller software](/2-body/1_software-uav.md/#1-fcsflight-controller-software)
      2. [MAVROS](/2-body/1_software-uav.md#2-mavros)
   2. [지상 관제(GCS) 소프트웨어](/2-body/2_software-gcs.md/)
   3. [하드웨어](/2-body/3_hardware.md)
       1. [GPS 모듈](/2-body/3_hardware.md/#1-gps-모듈)
       2. [PX4 Optical Flow](/2-body/3_hardware.md/#2-px4-optical-flow)
       3. [PX4 Telemetry Radio](/2-body/3_hardware.md/#3-px4-telemetry-radio)
       4. [Wifi 모듈](/2-body/3_hardware.md/#4-wifi-모듈)

### 결과 <!-- omit in toc -->
   1. [프로젝트 성과](/3-conclusion/result.md)
