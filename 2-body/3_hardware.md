# 하드웨어 <!-- omit in toc -->

## 목차 <!-- omit in toc -->

- [1. GPS 모듈](#1-gps-모듈)
  - [1.1. 개요](#11-개요)
  - [1.2. 취약점 분석 방법론](#12-취약점-분석-방법론)
    - [1.2.1. GPS Spoofing (Jamming)](#121-gps-spoofing-jamming)
    - [1.2.2. GPS Spoofing (Location Change)](#122-gps-spoofing-location-change)
- [2. PX4 Optical Flow](#2-px4-optical-flow)
  - [2.1. 개요](#21-개요)
  - [2.2. 취약점 분석 방법론](#22-취약점-분석-방법론)
    - [2.2.1. IMU Sensor & Ultrasonic Sensor](#221-imu-sensor--ultrasonic-sensor)
    - [2.2.2. Image Sensor](#222-image-sensor)
- [3. PX4 Telemetry Radio](#3-px4-telemetry-radio)
  - [3.1. 개요](#31-개요)
  - [3.2. 취약점 분석 방법론](#32-취약점-분석-방법론)
- [4. Wifi 모듈](#4-wifi-모듈)
  - [4.1. 개요](#41-개요)
  - [4.2. 취약점 분석 방법론](#42-취약점-분석-방법론)


## 개요

본 프로젝트에서는 다음과 같이 4가지 센서에 대해서 취약점 분석을 진행하였다.

![image](https://user-images.githubusercontent.com/91944211/145882093-94ad9ea0-4c00-44b1-b647-e2631babce0f.png)

본 프로젝트에서는 4가지 센서에 대해서 적용할 수 있는 6가지 공격을 선별하였다. 기존 논문/컨퍼런스 발표/기술보고서 분석을 통해 공격을 선별하였으며, 총 8가지 공격 중 4가지 공격을 성공했다.

![image](https://user-images.githubusercontent.com/91944211/145883340-f0a455fb-bf4e-4716-ac81-4e713d2bc0eb.png)


## 1. GPS 모듈

### 1.1. 개요

GPS 모듈은 PX4-Autopilot에서 GPS 정보를 주고받는 모듈이다. 

GPS 모듈을 통해 총 2가지 GPS Spoofing을 시도하였으며, GPS Spoofing은 실패하였으나, Jamming은 성공하였다.

### 1.2. 취약점 분석 방법론

#### 1.2.1. GPS Spoofing (Jamming)

가장 대표적으로 알려진 GPS Spoofing 기법을 응용하여 GPS 정보를 계속해서 보내는 Jamming 공격을 수행하였다.

큰 무리없이 GPS Spoofing을 통한 Jamming에 성공했다.

#### 1.2.2. GPS Spoofing (Location Change)

참고 링크 : https://gpspatron.com/spoofing-a-multi-band-rtk-gnss-receiver-with-hackrf-one-and-gnss-jammer/

위 링크와 같은 방법으로 공격을 시도하였으나, 다른 인공위성 신호(GNSS)의 간섭때문에 실패했다.


## 2. PX4 Optical Flow

### 2.1. 개요

Optical Flow는 쉽게 이야기해 카메라라고 볼 수 있다. Optical Flow는 IMU(Inertial measurement unit) Sensor, Ultrasonic Sensor, 그리고 Image Sensor로 이루어져 있다.

IMU Sensor와 Ultrasonic Sensor에 대해서는 공진주파수 공격을 진행하였으나 실패했고, Image Sensor를 대상으로는 강한 빛을 이용한 센서 무력화에 성공했다.

### 2.2. 취약점 분석 방법론

취약점 분석에 있어서 다음 논문을 많이 참고하였다 : https://ieeexplore.ieee.org/abstract/document/6630805/

#### 2.2.1. IMU Sensor & Ultrasonic Sensor

IMU Sensor와 Ultrasonic Sensor에 대해서는 공진주파수 공격을 진행하였다.

먼저 IMU Sensor의 경우에는 내부 스프링 고유진동수 공격을 시도해 보았다. 그러나 실패하였다.

참고 : https://www.usenix.org/sites/default/files/conference/protected-files/enigma17_slides_kim.pdf

다음으로 Ultrasonic Sensor의 경우 Arduino를 이용하여 공진주파수 공격을 진행하였다.

그러나 Frequency가 맞지 않아 성공하지 못했다. (40KHz → 42KHz)

#### 2.2.2. Image Sensor

Image Sensor에 대해서는 강한 빛을 사용한 센서 무력화 공격을 진행하였다.

큰 무리없이 공격을 성공할 수 있었다.

참고 : https://www.usenix.org/sites/default/files/conference/protected-files/woot_slides_park.pdf


## 3. PX4 Telemetry Radio

### 3.1. 개요

Telemetry Radio는 PX4-Autopilot에서 RF신호를 송/수신하는 센서이다. 이 센서를 통해서는 MAVLink 정보나 Remote Controller 정보를 보낼 수 있다.

따라서 본 프로젝트에서는 각각 MAVLink와 Remote Controller 정보로 나누어 RF Replay Attack을 수행하였고, 결론적으로 Remote Controller 정보를 통한 RF Replay Attack만 성공하였다.

### 3.2. 취약점 분석 방법론

Telemetry Radio에 대해서는 RF Replay Attack을 수행하였다.

MAVLink 혹은 Remote Controller로 나누어 정보를 보낼 수 있어서 각각 RF Replay Attack을 수행하였다.

결론적으로 Remote Controller만 성공하였다. 그 이유는 MAVLink의 경우 FHSS기법을 사용하고, RC는 DSSS기법을 사용하기 때문이었다.

## 4. Wifi 모듈

### 4.1. 개요

Wi-fi 모듈은 말 그대로 와이파이를 통해 정보를 송/수신하는 센서이다. 본 프로젝트에서는 ESP8266 Wi-fi 모듈을 사용하였다. ESP8266 Wi-fi 모듈을 대상으로는 Deauth Attack을 진행하였다.

### 4.2. 취약점 분석 방법론

MAVLink Bridge Wifi를 통해서 Deauth Attack을 진행했고, 큰 무리 없이 성공할 수 있었다.

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

