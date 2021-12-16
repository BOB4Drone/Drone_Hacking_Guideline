# 프로젝트 성과 <!-- omit in toc -->

## 목차 <!-- omit in toc -->

- [1. Issue Commits](#1-issue-commits)
- [2. 취약점 제보](#2-취약점-제보)
   - [2.1 PX4 firmware 취약점](#21-px4-firmware-취약점)
      - [2.1.1 msgID #332 BOF](#211-msgid-332-bof)
      - [2.1.2 msgID #147 oob-read](#212-msgid-147-oob-read)
      - [2.1.3 dos attack through mavftp](#213-dos-attack-through-mavftp)
   - [2.2 QGround Control 취약점](#22-qground-control-취약점)
      - [2.2.1 msgID #126 oob-read](#221-msgid-126-oob-read)
      - [2.2.2 msgID #266, #267 oob-read](#222-msgid-266-267-oob-read)
      - [2.2.3 null exception in airmap login function](#223-null-pointer-dereference-in-airmap-login-function)
      - [2.2.4 oob-read in mavlink console](#224-oob-read-in-mavlink-console)
   - [2.3 MAVROS 취약점](#23-mavros-취약점)
       - [2.3.1 msgID #192 oob-read](#231-msgid-192-bof)
       - [2.3.2 msgID #332 oob-read](#232-msgid-332-bof)
- [3. 발견 취약점 코드 패치](#3-발견-취약점-코드-패치)
- [4. 도커화를 통한 발견 취약점 재현](#4-도커화를-통한-발견-취약점-재현)
- [5. 논문 투고](#5-논문-투고)

---

### 1. Issue Commits

| No. | 취약점  | 링크  |
|---|---|---|
| 1 | [PX4] Bug Found in msgID #332 mavlink protocol BOF  | https://github.com/PX4/PX4-Autopilot/issues/18369  |
| 2 | [PX4] Bug Found in msgID #147 mavlink protocol OOB | https://github.com/PX4/PX4-Autopilot/issues/18385  |
| 3 | [PX4] A Bug in MAV_CMD_LOGGING_START (msgID #76 - command 2510) |  https://github.com/PX4/PX4-Autopilot/issues/18580 |
| 4 | [QGC] Bug Found in msgID #126 mavlink protocol OOB read  | https://github.com/mavlink/qgroundcontrol/issues/10020  |
| 5 | [PX4] loop of mavftp protocol crash drone |  https://github.com/PX4/PX4-Autopilot/issues/18651 |
| 6 | [QGC] oob read occurs in the handler function handling msgid 0x10a, 0x10b  |  https://github.com/mavlink/qgroundcontrol/issues/10037 |
| 7 | [QGC] Found a bug in the QGC Airmap function |  https://github.com/mavlink/qgroundcontrol/issues/10035 |
| 8 | [MAVROS] BOF occurs in the handler function handling msgid 192 |  https://github.com/mavlink/mavros/issues/1668 |
| 9 | [MAVROS] BOF occurs in the handler function handling msgid 332 |  https://github.com/mavlink/mavros/issues/1666 |
| 10 | [QGC] OOB READ in mavlink console |  https://github.com/mavlink/qgroundcontrol/issues/10059 |
| 11 | [QGC] use after free occurs in GeoTag Images menu |  https://github.com/mavlink/qgroundcontrol/issues/10068 |
---

## 2. 취약점 제보

### 2.1 PX4 firmware 취약점



#### 2.1.1 msgID #332 BOF 

* msgID #332번은 TRAJECTORY_REPRESENTATION_WAYPOINTS 패킷으로 valid points를 5개까지만 받는다고 프로토콜 문서에 명시되어 있다.
![image](https://user-images.githubusercontent.com/91944211/145849438-b036099c-e19b-4b51-ae16-c5b2bb9c9aeb.png)

* 따라서 실제 코드에서도 waypoint배열은 프로토콜 문서와 같이 크기가 5인 배열로 선언되어 있다.
   ```
   struct vehicle_trajectory_waypoint_s {
   #endif
        uint64_t timestamp;
        uint8_t type;
        uint8_t _padding0[7]; // required for logger
        struct trajectory_waypoint_s waypoints[5];
   ```

* 또한 waypoint필드 타입은 uint8_t이므로 0xff까지 입력이 가능하고 따라서 만약 5가 넘으면 예외처리를 해주는 구문이 필요하다.

* 하지만 패킷 디코딩시 별다른 예외처리를 진행하지 않는다.
   ```
   trajectory_representation_waypoints->valid_points=mavlink_msg_trajectory_representation_waypoints_get_valid_points(msg);
   ```

* 또한 파싱한 데이터값을 사용할때도 다른 예외처리 구문이 존재하지 않고 for문의 iteration 횟수로 사용하기 때문에 waypoints배열을 벗어난 곳에서도 true로 메모리를 덮을 수 있다.
   ```
   for (int i = 0; i < number_valid_points; ++i) {
		trajectory_waypoint.waypoints[i].point_valid = true;
   }
   ```
---

#### 2.1.2 msgID #147 oob-read

* msgID #147번 BATTERY_STATUS 패킷에서 현재 연결되어있는 배터리들의 전압을 더해줄때 취약점이 발생한다.

* 다음과 같이 배터리의 전압을 더해준다.

	```
	while (battery_mavlink.voltages[cell_count] < UINT16_MAX && cell_count < 10) {
		battery_status.voltage_cell_v[cell_count] = (float)(battery_mavlink.voltages[cell_count]) / 1000.0f;
		voltage_sum += battery_status.voltage_cell_v[cell_count];
		cell_count++;
	}
	```

* PX4에서 컴파일시 최적화 옵션이 활성화되어 있기 때문에 컴파일시 while문 내 조건은 다음과 같이 변화한다.
	```
	while (battery_mavlink.voltages[cell_count] < UINT16_MAX && cell_count < 10)
	```
	
	```
	while (1 && cell_count < 10)
	```
	
	```
	while (1 < 10)
	```
	
	```
	while (1)
	```

* 따라서 컴파일시 while문 내 조건은 항상 참이 되도록 변경된다.

* 실제로 ida를 통해 컴파일된 코드를 확인한 결과 while문 내부 코드가 지워진 것을 확인할 수 있었다.
	![image](https://user-images.githubusercontent.com/91944211/145856918-5ec925f8-3e43-4a5f-a4b5-69845f82589c.png)

* while문이 참이 되므로 cell_count가 무한히 증가하여 oob-read가 발생하게 된다

---

#### 2.1.3 dos attack through mavftp

* mavftp를 통해 디렉토리를 생성 가능하다.
	![image](https://user-images.githubusercontent.com/91944211/145863374-99c0ec79-7fa8-436d-aaa2-9aedef56b05c.png)

* 반복적으로 디렉토리 생성시 램 제한 범위를 넘어서게(px4는 vfs를 사용하기 때문에 디렉토리를 생성할시 램에 적재된다.) 되고 드론이 종료된다.

* 따라서 이것을 이용해 드론을 종료시키는 dos공격이 가능하다.

---

### 2.2 QGround Control 취약점

#### 2.2.1 msgID #126 oob-read

* MsgID #126번 serial_control 패킷에서 취약점이 발생한다.

* 프로토콜 문서를 보면 data필드는 최대 70바이트이지만 데이터의 길이를 나타내는 count 필드는 uint8_t로 0xff까지 입력이 가능하다. 따라서 count필드와 data의 실제 길이가 다를때 예외처리가 필요하다.

	![image](https://user-images.githubusercontent.com/91944211/145865023-e2e0909b-8a76-4a89-ba01-15efa1dc68bf.png)

* 하지만 예외처리 없이 바로 count가 사용되는 것을 확인할 수 있었다.
	```
	case MAVLINK_MSG_ID_SERIAL_CONTROL:
    {
        mavlink_serial_control_t ser;
        mavlink_msg_serial_control_decode(&message, &ser);
        emit mavlinkSerialControl(ser.device, ser.flags, ser.timeout, ser.baudrate, QByteArray(reinterpret_cast<const char*>(ser.data), ser.count));
    }
	```

* 따라서 임의로 조작된 ser.data는 QByteArray함수로 들어가게 된다.
	```
	QByteArray(reinterpret_cast<const char*>(ser.data), ser.count)
	```

* QByteArray함수에서는 count 크기만큼 데이터를 복사하므로 count값을 data 길이보다 크게 패킷을 보내면 out-of-bound read가 발생한다.
	```
	QByteArray::QByteArray(const char *data, qsizetype size)
	{
  		if (!data) {
  			d = DataPointer();
  		} else {
   	 	 if (size < 0)
    	 		size = qstrlen(data);
    		 if (!size) {
   	 		d = DataPointer::fromRawData(&_empty, 0);
   	 	 } else {
    	  		d = DataPointer(Data::allocate(size), size);
       			Q_CHECK_PTR(d.data());
          			memcpy(d.data(), data, size);
        			d.data()[size] = '\0';
       		 }
    	}
	}
	```
	
---

#### 2.2.2 msgID #266, #267 oob-read

* LOGGING_DATA, LOGGING_DATA_ACKED 패킷에서 취약점이 발생하며 두 패킷의 구조는 모두 아래와 같다.
	![image](https://user-images.githubusercontent.com/91944211/145867572-d61c2a20-e52d-4957-a2c8-6130186910ac.png)

* Serial_Control 패킷과 같이 data 최대 길이인 249인 반면에 data 길이를 나타내는 length 필드의 최댓값은 256이다. 따라서 length필드와 data의 길이가 다를때 예외처리가 존재해야 한다.

* 하지만 예외처리가 존재하지 않고 바로 QByteArray함수를 호출한다.

	```
	void Vehicle::_handleMavlinkLoggingData(mavlink_message_t& message)
    {
        mavlink_logging_data_t log;
        mavlink_msg_logging_data_decode(&message, &log);
        emit mavlinkLogData(this, log.target_system, log.target_component, log.sequence, log.first_message_offset, QByteArray((const char*)log.data, log.length), false);
	```

* QByteArray함수에서는 count 크기만큼 데이터를 복사하므로 length값을 data 길이보다 크게 패킷을 보내면 out-of-bound read가 발생한다.
	```
	QByteArray::QByteArray(const char *data, qsizetype size)
	{
  		if (!data) {
  			d = DataPointer();
  		} else {
   	 	 if (size < 0)
    	 		size = qstrlen(data);
    		 if (!size) {
   	 		d = DataPointer::fromRawData(&_empty, 0);
   	 	 } else {
    	  		d = DataPointer(Data::allocate(size), size);
       			Q_CHECK_PTR(d.data());
          			memcpy(d.data(), data, size);
        			d.data()[size] = '\0';
       		 }
    	}
	}
	```

---
	
#### 2.2.3 null pointer dereference in airmap login function

* Select Tool > Application Settings > AirMap > Show Flight List 메뉴에서 Refresh시 발생한다.

	![image](https://user-images.githubusercontent.com/91944211/145871047-7d4e5fd8-6d8e-4c91-b0b8-06f3f4f970de.png)

* username, password 등을 인자로 함수를 호출하는데 username, password를 입력하지 않은 채 refresh를 누르므로 이 과정에서 null exception이 발생한다.

	```
    Authenticator::AuthenticateWithPassword::Params params;
    params.oauth.username = _settings.userName.toStdString();
    params.oauth.password = _settings.password.toStdString();
    params.oauth.client_id = _settings.clientID.toStdString();
    params.oauth.device_id = "QGroundControl";
    qCDebug(AirMapManagerLog) << "User authentication" << _settings.userName;
    _client->authenticator().authenticate_with_password(params,
            [this](const Authenticator::AuthenticateWithPassword::Result& result) {
	```

* 따라서 null exception으로 프로그램이 종료되게 되고 dos공격이 가능해진다.

---

#### 2.2.4 oob-read in mavlink console

* QGroundControl에서 Mavlink shell을 사용시 쉘과 통신하기 위해 Serial_control packet을 생성해준다.

* Serial_control 패킷 생성시 패킷 데이터 청크보다 사이즈가 더 크게 되어 다음과 같은 코드에서 out-of-bound read가 발생한다.

	```
	   while(data.size()) {
        QByteArray chunk{data.left(MAVLINK_MSG_SERIAL_CONTROL_FIELD_DATA_LEN)};
        uint8_t flags = SERIAL_CONTROL_FLAG_EXCLUSIVE |  SERIAL_CONTROL_FLAG_RESPOND | SERIAL_CONTROL_FLAG_MULTI;
        if (close) flags = 0;
        auto protocol = qgcApp()->toolbox()->mavlinkProtocol();
        auto link = _vehicle->vehicleLinkManager()->primaryLink();
        mavlink_message_t msg;
        mavlink_msg_serial_control_pack_chan(
                    protocol->getSystemId(),
                    protocol->getComponentId(),
                    sharedLink->mavlinkChannel(),
                    &msg,
	```

---

### 2.3 MAVROS 취약점

#### 2.3.1 msgID #192 bof

* MAG_CAL_REPORT라는 calibration을 보정한 결과를 보내는 패킷에서 취약점이 발생한다.

* 해당 msgID에 해당하는 프로토콜 문서를 보면 compass_id는 uint8_t로 0xff까지 설정이 가능하다.

	![image](https://user-images.githubusercontent.com/91944211/145876102-0d4b1784-3dd7-4eb0-8eee-63a0f836524d.png)

* compass_id는 별도의 예외처리를 가지고 있지 않으며 calibration_show 배열의 index로 접근하고 해당 인덱스를 false로 덮는 코드가 존재한다.

	```
	if (calibration_show[mr.compass_id]) {
      auto mcr = mavros_msgs::msg::MagnetometerReporter();

      mcr.header.stamp = node->now();
      mcr.header.frame_id = std::to_string(mr.compass_id);
      mcr.report = mr.cal_status;
      mcr.confidence = mr.orientation_confidence;
      mcr_pub->publish(mcr);
      calibration_show[mr.compass_id] = false;
    }
	```

* calibration_show배열은 크기 8을 가지는 배열로 되어있지만 index인 compass_id는 0xff까지 입력이 가능하기 때문에 out-of-bound read와 write 모두 가능하다.

	```
	private:
  		rclcpp::Publisher<std_msgs::msg::UInt8>::SharedPtr mcs_pub;
 		rclcpp::Publisher<mavros_msgs::msg::MagnetometerReporter>::SharedPtr mcr_pub;

  		std::array<bool, 8> calibration_show;
  		std::array<uint8_t, 8> _rg_compass_cal_progress;
	```

---

#### 2.3.2 msgID #332 bof

* PX4에서 발견한 [msgID #332 BOF](#211-msgid-332-bof) 취약점과 같으며 ROS에서는 아직 패치가 진행되지 않아 추가로 다시 발견하였다.

---

### 3. 발견 취약점 코드 패치

| No. | 내용  | 링크  |
|---|---|---|
| 1 | [2.2.2 msgID #266, #267 oob-read](#222-msgid-266-267-oob-read)  | https://github.com/mavlink/qgroundcontrol/pull/10038  |
| 2 | [2.3.1 msgID #192 oob-read](#231-msgid-192-bof)  | https://github.com/mavlink/mavros/pull/1675  |
| 3 | [2.3.2 msgID #332 oob-read](#232-msgid-332-bof)  | https://github.com/mavlink/mavros/pull/1667  |

---

### 4. 도커화를 통한 발견 취약점 재현

이후에 드론의 취약점 분석을 시도할 분들을 위해 도커 환경을 구성하여 개발했던 퍼저를 함께 배포하려 한다.
[Docker Hub Repository](https://hub.docker.com/repository/docker/ashinedo/4-drone) 에 배포되며 아래 과정을 통해 이용할 수 있다.

**사용 방법**

이미지 다운로드
```
$ docker pull ashinedo/4-drone:PX4-1.0
```

이미지 실행  
취약점이 검출됐던 PX4 버전이 설치되며 퍼저가 자동으로 실행된다.
```
$ docker run -it --rm ashinedo/4-drone:PX4-1.0
```

만약 자동으로 퍼저가 실행되지 않기를 바란다면 bash 옵션을 통해 직접 명령어를 입력할 수 있다.
```
$ docker run -it --rm ashinedo/4-drone:PX4-1.0 bash
```

도커 이미지들은 많은 용량을 차지하므로 이용 후 제거
```
로컬 이미지 리스팅
$ docker images

태그를 이용해 로컬 이미지 제거
$ docker rmi ashinedo/4-drone:PX4-1.0
```

배포중인 Docker Image

**[PX4-1.0](https://hub.docker.com/layers/178199113/ashinedo/4-drone/PX4-1.0/images/sha256-c1623af96905cf568545083de81363e87fcd66dd38b717dae300b5e84184c711?context=repo)** - MAVLink fuzzer(sysid header)

---

### 5. 논문 투고

본 프로젝트를 진행하면서 1건의 논문 투고를 진행하였다. 논문에 대한 정보는 다음과 같다.

```
논문 제목 : PX4 Autopilot의 MAVLink 모듈에 대한 취약점 분석
저자 : 경규창, 김동현, 최수빈, 이준오, 김지수, 류형호, 조진성
컨퍼런스 : 2021 한국정보보호학회 동계학술대회 (CISC-W'21)
Link : [To Be Announced]
```

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
