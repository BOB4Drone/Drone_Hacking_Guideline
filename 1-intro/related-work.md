# 선행 연구들 <!-- omit in toc -->

## 목차 <!-- omit in toc -->

- [1. RVFuzzer: Finding Input Validation Bugs in Robotic Vehicles through Control-Guided Testing](#1-rvfuzzer-finding-input-validation-bugs-in-robotic-vehicles-through-control-guided-testing)
  - [1.1. 요약](#11-요약)
- [2. Security Analysis of the Drone Communication Protocol: Fuzzing the MAVLink protocol](#2-security-analysis-of-the-drone-communication-protocol-fuzzing-the-mavlink-protocol)
  - [2.1. 요약](#21-요약)
- [3. How to Analyze the Cyber Threat from Drones](#3-how-to-analyze-the-cyber-threat-from-drones)
  - [3.1. 요약](#31-요약)

- - -

## 1. RVFuzzer: Finding Input Validation Bugs in Robotic Vehicles through Control-Guided Testing

### 1.1. 요약

RV(robot vehicle)모델들이 다양해짐에 따라 테스트해야 하는 매개변수들이 매우 많아졌고 점점 퍼징을 진행하기 어려워졌다. 시간이 지남에 따라 이 매개변수들을 만족시켜주는 퍼저들은 등장했지만 퍼저를 통한 input이 오작동을 일으키는지 판단하기 어렵기 때문에 RV 시스템에 바로 적용하기 어려웠다. 따라서 이 논문에서는 로봇의 제어 가능여부를 감지하는 RVF 감지기를 구현하여 쉽게 오작동 여부를 판단하고 기존의 RV 시뮬레이션 프레임워크와 쉽게 결합하도록 만들어 쉽게 취약점을 도출해낼 수 있게 했다. 

## 2. Security Analysis of the Drone Communication Protocol: Fuzzing the MAVLink protocol

### 2.1. 요약

Mavlink에 대한 직접적인 퍼저를 구축한 논문이다. Mavlink 구조 분석부터 시작해서 진행한 퍼징 방법론, 뮤테이션 방식, CRC 계산 등 자신이 퍼저를 만들면서 경험한 내용들을 설명하는 식의 논문이다. 이 퍼저를 통해 저자는 취약점을 발견해내었으며 퍼저를 개선하는 것을 future works로 남겨두었다.

## 3. How to Analyze the Cyber Threat from Drones

### 3.1. 요약
DHS(Department of Homeland Security)의 관점에서 UAS 사이버 보안 위협 영향을 조사한 보고서이다. 주로 사이버 보안과 관련하여 UAS 위협을 분류하는 데 중점을 두고 있으며 완화 및 방어 전략에 대한 노력을 지시하기 위해 이러한 위협을 분석하는 데 도움이 될 수 있는 접근 방식을 제시한다.

본 보고서에서는 UAS를 공격 대상으로 보는 관점과, UAS를 공격 수단으로 보는 관점으로 나누어 총 4개의 시나리오를 제시한다. 

나아가 앞으로도 DHS의 고위 관직자들은 계속해서 더 안전한 UAS system을 구축하기 위해 더 노력해야한다고 제시하며 마무리된다. 


---

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

