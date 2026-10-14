# 자율주행 기반 물류 카트 견인 로봇

> **작업자 추종·경로 기록·자율주행을 통합한 ROS 2 기반 견인 로봇**

![ROS 2](https://img.shields.io/badge/ROS_2-Humble-22314E?logo=ros&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Jetson_Orin_Nano-76B900)
![Docker](https://img.shields.io/badge/Environment-Docker-2496ED?logo=docker&logoColor=white)

---

## 1. 프로젝트 개요

### 1-1. 프로젝트 소개

- **프로젝트명:** 자율주행 기반 물류 카트 견인 로봇
- **영문명:** Autonomous Driving-Based Towing Robot
- **프로젝트 정의:** LiDAR 기반 작업자 추종, SLAM 기반 이동 경로 기록, 저장 경로 기반 반복 자율주행을 결합한 ROS 2 기반 물류 카트 견인 로봇

<p align="center">
  <img src="images/robot_prototype.png" alt="제작된 AFTR 로봇과 견인 카트" width="900">
</p>

<p align="center">
  <img src="images/outline.png" alt="작업자 추종, 경로 저장, 반복 자율주행 개념도" width="950">
</p>

### 1-2. 개발 배경 및 필요성

기존 물류 자동화 시스템은 전용 경로, 마커, 중앙관제 등 부가 인프라 구축에 따른 초기 비용과 설치 부담이 있습니다. 특히 작업 환경이 자주 바뀌는 중소규모 사업장에서는 고정형 자동화 설비를 도입하기 어렵습니다. 이를 해결하기 위해 **작업자의 실제 이동 동선을 추종·기록하고, 기록한 경로를 자율주행에 재사용하는 견인 로봇**을 개발했습니다.

### 1-3. 프로젝트 특장점

- **인프라 부담 최소화:** 별도의 마커 및 중앙관제 구축 없이 LiDAR 기반 작업자 추종으로 운반 작업 지원
- **현장 동선 기록:** 작업자를 따라 이동하며 TF 기반 위치·방위각을 수집하고 CSV 경로점으로 저장
- **안전 경로 생성:** 점유 격자 지도와 Distance Transform으로 장애물 이격거리를 평가하고 저장 경로를 보정
- **Wide-Turn 궤적 생성:** 지도 코너 및 회전 방향을 분석하고 Bézier 곡선 기반으로 선회 궤적 구성
- **양방향 재주행:** 보정 경로를 ROS 2 Path로 구성하여 정방향 및 복귀 주행
- **견인 안정성 및 안전 운용:** 물리적 조향각 제한 래치, 낙상 감지에 따른 주행 정지, GUI 상태 모니터링

### 1-4. 주요 기능

| 기능 | 설명 |
| --- | --- |
| 작업자 추종 | 2D LiDAR 하체 클러스터링 및 양다리 후보 기반 대상 추적 |
| 속도 제어 | 작업자와의 거리·각도 오차를 이용한 선속도·각속도 제어 |
| 경로 기록 | map 좌표계의 위치·방위각을 수집해 CSV waypoint 저장 |
| 경로 보정 | 경로 재샘플링, 점유 격자 전처리, 장애물 거리 평가 및 안전 경로 생성 |
| 코너 주행 | Wide-Turn 경로 생성과 충돌 검사를 통한 선회 경로 구성 |
| 반복 자율주행 | Nav2 기반 경로 재생, 정방향·복귀 주행 및 상태 모니터링 |
| 쓰러짐 감지 | RGB 영상 기반 작업자 자세 인식 및 위험 상황 시 주행 정지·알림 |
| 운영자 GUI | 모드 전환, 주행 상태·속도 표시, 조이스틱을 이용한 위치 정렬 |

### 1-5. 기대 효과 및 활용 분야

작업자 추종 운반과 저장 경로 기반 반복 주행을 하나의 시스템에서 수행합니다. 작업 현장의 실제 이동 동선을 경로 데이터로 활용하여 별도의 마커나 고정형 경로 인프라에 대한 의존도를 줄이고, 기존 롤테이너와 연결해 물품 운반을 지원합니다.

**활용 분야:** 중소규모 사업장 물류 운반, 실내 반복 운송, 롤테이너 견인 및 이동 로봇 운용.

### 1-6. 기술 스택

| 구분 | 사용 기술 및 장비 |
| --- | --- |
| 로봇 플랫폼 | ROS 2 Humble, ros2_control |
| 지도 및 자율주행 | SLAM Toolbox, Nav2, AMCL |
| 센서 | RPLIDAR S2, Intel RealSense D435i |
| 구동계 | MD400T BLDC Motor Driver, In-Wheel Motor × 2 |
| 운영자 인터페이스 | Qt 기반 Operator GUI, Audio, Status LED |
| 배포 환경 | Docker, Linux amd64 Laptop, NVIDIA Jetson Orin Nano |

---

## 2. 팀원 소개

| 구분 | 역할 | 담당 영역 |
| --- | --- | --- |
| 팀장 | Leg Tracking 및 Fall Detection | `aftr_tracking`, `aftr_fall_detection` |
| 팀원1 | Path Planning 및 Autonomous Navigation Support | `aftr_path_manager`, `aftr_status_led` |
| 팀원2 | Firmware 및 System Integration | `aftr_audio`, `aftr_bringup`, `aftr_description`, `aftr_gui`, `aftr_hardware`, `aftr_mode_manager`, `aftr_navigation`, `aftr_slam`, `aftr_teleop` |
| 팀원3 | Mechanical Design | 기구부 설계 및 CAD |

---

## 3. 시스템 구성도

### 3-1. 하드웨어 및 시스템 구성

<p align="center">
  <img src="images/system_architecture.png" alt="센서, Mini PC, GUI 및 구동계 시스템 구성도" width="950">
</p>

RPLIDAR S2와 D435i가 센서 입력을 제공하며, ROS 2에서 추종·지도 작성·경로 재현과 GUI 운용을 통합합니다. 구동계는 MD400T 2채널 BLDC 모터 드라이버와 인휠모터 2개로 구성됩니다.

### 3-2. 소프트웨어 운용 흐름

<p align="center">
  <img src="images/operation_workflow.png" alt="초기 설정부터 작업자 추종, 경로 저장 및 자율주행까지의 흐름도" width="1200">
</p>

1. **초기 설정:** 통신 연결 및 센서·구동계 상태 확인, 운용 모드 선택
2. **추종 및 경로 저장:** 작업자 하체 추적, SLAM 지도 작성 및 waypoint 기록
3. **경로 재현 및 자율주행:** 기록 경로 보정, AMCL 위치 추정, 정방향·복귀 주행

### 3-3. 운영자 GUI

<p align="center">
  <img src="images/operator_gui.png" alt="작업자 추종 및 자율주행 운용 GUI" width="1200">
</p>

GUI에서 모드 선택, 경로 저장 설정, 주행 명령, 로봇 상태 및 지도 정보를 확인할 수 있습니다.

### 3-4. 기구 설계

<p align="center">
  <img src="images/robot_components.png" alt="조절식 디스플레이 거치대와 로봇 주요 구성 요소" width="850">
</p>

<p align="center"><sub>모니터 암, 비상 정지 버튼, 상태 표시등 및 카트 탈부착용 래치 구조</sub></p>

<p align="center">
  <img src="images/steering_latch_mechanism.png" alt="작업 카트와 로봇 간 견인 연결 구조" width="850">
</p>

<p align="center"><sub>물리적 조향각 제한 래치: 로봇-카트 간 과조향을 억제하고 선회 안정성을 확보하는 구조</sub></p>

---

## 4. 작품 소개영상

[![자율주행 기반 물류 카트 견인 로봇 — 작품 소개영상](https://img.youtube.com/vi/bFqfijlAvS0/hqdefault.jpg)](https://youtu.be/bFqfijlAvS0)

[YouTube에서 작품 소개영상 보기](https://youtu.be/bFqfijlAvS0)

---

## 5. 핵심 소스코드

프로젝트의 핵심 기술은 **자율주행 경로 보간**, **Human Tracking**, **Motor Driver 제어**로 구성됩니다.

### 5-1. 자율주행 — 경로 보간 및 곡선 생성

[`aftr_path_manager/aftr_path_manager/bidirectional_corner_fillet.py`](aftr_path_manager/aftr_path_manager/bidirectional_corner_fillet.py)

저장 경로의 급격한 방향 전환 구간을 부드럽게 연결하는 경로 보간 기능입니다. 5차 Bézier 전이 곡선의 제어점 가중치를 계산하여 곡선을 일정 간격으로 생성하며, 안전 경로 보정 및 Wide-Turn 처리와 함께 경로 재주행에 활용합니다.

```python
# _quintic_transition_samples() 내부: 5차 Bezier 가중치
sample_count = max(2, int(math.ceil(max(length_m, step_m) / step_m)))
segment_samples = []
for index in range(sample_count + 1):
    t = index / sample_count
    one_minus_t = 1.0 - t
    weights = (
        one_minus_t**5,
        5.0 * one_minus_t**4 * t,
        10.0 * one_minus_t**3 * t**2,
        10.0 * one_minus_t**2 * t**3,
        5.0 * one_minus_t * t**4,
        t**5,
    )
```

안전 경로 생성에서는 점유 격자 지도와 Distance Transform으로 장애물 이격거리를 평가하고, 코너 구간에는 외측 선회 궤적과 충돌 검사를 적용합니다.

### 5-2. Human Tracking — 작업자 탐지 및 추적 대상 확정

[`aftr_tracking/aftr_tracking/tracker_node.py`](aftr_tracking/aftr_tracking/tracker_node.py)

LiDAR scan에서 검출한 작업자 후보 중 거리 조건을 충족하는 대상을 선택합니다. 일정 프레임 동안 후보가 관찰되면 추적 대상을 확정하고 `FOLLOW` 상태로 전환합니다.

```python
def search_target(self, humans):
    """Lock the nearest candidate after the required confirmation frames."""
    valid_candidates = [
        h
        for h in humans
        if 0.3 <= math.hypot(h[0], h[1]) <= self.detection_dist_limit
    ]

    if not valid_candidates:
        self.candidate_count = 0
        return None

    current_nearest = min(valid_candidates, key=lambda h: math.hypot(h[0], h[1]))
    self.candidate_count += 1

    if self.candidate_count >= self.required_frames:
        self.locked_target = current_nearest
        self.lost_frames = 0
        self.candidate_count = 0
        self.mode = "FOLLOW"
        return self.locked_target
```

추적 중에는 이전 대상과의 위치 차이, 일시적 미검출 및 장애물 여부를 고려해 추적 상태와 이동 명령을 관리합니다.

### 5-3. Motor Driver — BLDC 모터 속도 명령

[`aftr_hardware/src/md_driver.cpp`](aftr_hardware/src/md_driver.cpp)

`ros2_control` 하드웨어 인터페이스와 연결된 MD400T Driver에 좌·우 In-Wheel Motor의 RPM 명령을 전송합니다. 아래 함수는 토크 상태에 따른 Enable 값과 모터별 RPM을 PID207 데이터 형식으로 구성합니다.

```cpp
void MDDriver::setVelocity(int32_t rpm_left, int32_t rpm_right)
{
  // PID207: [Enable1][RPM1_L][RPM1_H][Enable2][RPM2_L][RPM2_H][ReqPID]
  std::vector<uint8_t> data;
  uint8_t enable = is_torque_on_ ? 1 : 0;

  data.emplace_back(enable);
  data.emplace_back(rpm_left & 0xFF);
  data.emplace_back((rpm_left >> 8) & 0xFF);

  data.emplace_back(enable);
  data.emplace_back(rpm_right & 0xFF);
  data.emplace_back((rpm_right >> 8) & 0xFF);

  data.emplace_back(2);
  sendPacket(PID::VEL_CMD, data);
}
```

패킷 헤더, Checksum 생성과 응답 데이터 처리는 [`md_packet.cpp`](aftr_hardware/src/md_packet.cpp)에 구현되어 있습니다. 세부 통신 규약은 [하드웨어 참고 문서](aftr_hardware/docs/md_packet_protocol.md)를 참고하세요.

---

## 설치 및 실행

본 저장소는 **13개 AFTR ROS 패키지**와 별도 외부 패키지를 사용하는 ROS 2 Workspace 프로젝트입니다. 먼저 Linux Host에 Git과 Docker Engine을 준비하세요.

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

git clone https://github.com/nayana224/26_HC281.git autonomous-following-towing-robot
git clone https://github.com/roverrobotics-forks/serial-ros2.git
git clone https://github.com/Slamtec/sllidar_ros2.git
git clone --branch ros2 https://github.com/ros-perception/laser_filters.git
```

**Laptop (amd64 / CPU)** — 저장소 디렉터리에서:

```bash
cd ~/ros2_ws/src/autonomous-following-towing-robot
./scripts/docker_build_laptop.sh
./scripts/docker_run_laptop.sh
```

Container 내부에서:

```bash
./src/autonomous-following-towing-robot/scripts/build_ws.sh
source /workspace/install_laptop/setup.bash
```

**Jetson (arm64 / CUDA)** — Jetson Host에서:

```bash
cd ~/ros2_ws/src/autonomous-following-towing-robot
./scripts/docker_build_jetson.sh
./scripts/docker_run_jetson.sh
```

Container 내부에서:

```bash
./src/autonomous-following-towing-robot/scripts/build_ws.sh
source /workspace/install_jetson/setup.bash
```

빌드 완료 후 실제 로봇의 Operator GUI는 Jetson Host에서 실행합니다.

```bash
./scripts/docker_run_jetson.sh bash -lc \
  'ros2 launch aftr_gui operator_system.launch.py'
```

> [!IMPORTANT]
> 실제 로봇 운용 전 모터·센서 연결, 견인 구조, 주변 장애물, 비상 정지 수단을 확인하고 반드시 현장 운영자의 감독 아래 실행하세요.

모델 파일, 장치 별칭 및 데이터 경로 등 자세한 설치 조건은 [설치 및 실행](docs/setup.md)을 참고하세요.

## ROS 패키지

| 패키지 | 역할 |
| --- | --- |
| `aftr_audio` | 음향 및 상태 안내 |
| `aftr_bringup` | 로봇 기본 시스템과 LiDAR 실행 |
| `aftr_description` | URDF / Xacro 로봇 모델 |
| `aftr_fall_detection` | 낙상 감지 및 heartbeat |
| `aftr_gui` | 운영자 GUI |
| `aftr_hardware` | ros2_control 하드웨어 인터페이스 |
| `aftr_mode_manager` | 운용 흐름 및 안전 상태 관리 |
| `aftr_navigation` | Nav2 및 위치 추정 설정 |
| `aftr_path_manager` | 경로 기록·재생 및 위치 데이터 저장 |
| `aftr_slam` | SLAM Toolbox 설정 |
| `aftr_status_led` | 상태 표시 LED |
| `aftr_teleop` | 수동 `/cmd_vel` 제어 |
| `aftr_tracking` | 작업자 추종 |

## 참고 문서

- [설치 및 실행](docs/setup.md)
- [운용 안내](docs/operation.md)
- [시스템 구조](docs/architecture.md)
- [ROS 인터페이스](docs/interfaces.md)
- [안전 정책](docs/safety.md)
- [개발 및 유지보수](docs/maintenance.md)
- [팀 구성 및 담당 영역](docs/team.md)
- [하드웨어 및 발표 참고 자료](docs/references/README.md)
