# Autonomous Following and Towing Robot (AFTR)

> **작업자 추종, 경로 기록 및 자율주행을 통합한 ROS 2 기반 견인 로봇**

![ROS 2](https://img.shields.io/badge/ROS_2-Humble-22314E?logo=ros&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Jetson_Orin_Nano-76B900)
![Docker](https://img.shields.io/badge/Environment-Docker-2496ED?logo=docker&logoColor=white)
![Status](https://img.shields.io/badge/Branch-devel-orange)

**AFTR(Autonomous Following and Towing Robot)**은 LiDAR 기반으로 작업자를 추종하며 이동 경로를 기록하고, SLAM으로 생성한 지도와 저장 경로를 활용해 자율주행 및 복귀 주행을 수행하는 ROS 2 Humble 기반 이동형 견인 로봇 프로젝트입니다.

**이 문서는 `devel` 브랜치를 기준으로 작성되었습니다.** 실제 로봇 운용에는 지정된 하드웨어와 환경별 설정이 필요합니다.

<p align="center">
  <img src="images/robot_prototype.png" alt="Autonomous Following and Towing Robot" width="900">
</p>

<p align="center">
  <img src="images/outline.png" alt="작업자 추종, 경로 저장, 반복 자율주행으로 구성된 프로젝트 핵심 개념" width="950">
</p>

## 주요 기능

| 기능 | 설명 |
| --- | --- |
| **작업자 추종** | LiDAR 기반 작업자 인식 및 추종 주행 |
| **경로 기록·재생** | 추종 중 경로·지도 기록 및 정방향·복귀 주행 |
| **지도 작성 및 위치 추정** | SLAM Toolbox를 이용한 지도 작성과 AMCL 기반 위치 추정 |
| **자율주행** | Nav2 기반 저장 경로 주행 |
| **안전 상태 모니터링** | 낙상 감지, 연결 상태 감시 및 안전 정지 |
| **운영자 인터페이스** | GUI, 안내음 및 상태 LED를 통한 제어·상태 확인 |
| **배포 환경** | Laptop(CPU)과 Jetson(CUDA) Docker 환경 분리 |

## 프로젝트 정보

| 항목 | 내용 |
| --- | --- |
| 프로젝트 | Autonomous Following and Towing Robot |
| 프로그램 | 한이음 드림업 |
| 저장소 브랜치 | `devel` (본 문서 기준) |
| 플랫폼 | ROS 2 Humble |
| 배포 대상 | NVIDIA Jetson Orin Nano |
| 개발 환경 | Linux amd64 Laptop / Jetson arm64 + Docker |

## 시스템 구성

<p align="center">
  <img src="images/system_architecture.png" alt="시스템 구성도" width="950">
</p>

- **LiDAR**: RPLIDAR S2
- **Depth Camera**: Intel RealSense D435i
- **Motor Driver**: MD400T 2채널 BLDC Driver
- **Drive System**: BLDC In-Wheel Motor × 2
- **Software**: ROS 2 Humble, ros2_control, SLAM Toolbox, Nav2
- **Interface**: Operator GUI, Audio, Status LED

## 운용 흐름

<p align="center">
  <img src="images/operation_workflow.png" alt="운용 흐름" width="1200">
</p>

1. **초기 설정**: 통신 상태와 센서·구동계 연결을 확인하고 운용 모드 선택
2. **작업자 추종 및 경로 기록**: LiDAR 기반으로 작업자를 추종하며 이동 경로와 지도를 기록
3. **경로 재현 및 자율주행**: 저장된 지도·경로로 위치를 추정하고 Nav2 기반으로 정방향·복귀 주행

자세한 운용 순서는 [운용 안내](docs/operation.md)를 참고하세요.

## 운영자 GUI (Operator GUI)

<p align="center">
  <img src="images/operator_gui.png" alt="Operator GUI" width="1200">
</p>

- 추종 및 자율주행 모드 선택
- 경로 저장 설정
- 로봇 운용 상태와 속도 표시
- 저장 경로 기반 정방향·복귀 주행
- 지도 및 현재 위치 모니터링
- 운행 중단 및 안전 상태 확인

## 기구 설계

<p align="center">
  <img src="images/robot_components.png" alt="로봇 구성 요소" width="850">
</p>

<p align="center"><sub>조절식 디스플레이 거치대, 상태 표시 LED 및 견인 연결부를 포함한 로봇 기구 설계</sub></p>

<p align="center">
  <img src="images/steering_latch_mechanism.png" alt="견인 연결 구조" width="850">
</p>

<p align="center"><sub>작업 카트와 로봇의 기계적 결합을 위한 연결부 구조</sub></p>

## 빠른 시작

### 사전 준비

Ubuntu/Linux 호스트에 **Git과 Docker Engine**을 설치하고 Docker 실행 권한을 준비합니다. ROS 2 Humble 개발 환경은 프로젝트의 Docker 이미지에서 제공합니다. Laptop과 Jetson의 CPU 아키텍처 및 GPU 구성이 다르므로 이미지와 빌드 결과물을 혼용하지 않습니다.

> [!IMPORTANT]
> 실제 로봇을 움직이기 전에는 센서·모터 연결 상태, 비상 정지 수단, 주변 장애물을 확인하고 현장 운영자가 즉시 정지할 수 있도록 준비해야 합니다. 자세한 내용은 [안전 정책](docs/safety.md)을 참고하세요.

### Workspace 구성

일반적인 ROS 2 Workspace를 준비하고 **`devel` 브랜치**를 clone합니다.

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

git clone --branch devel https://github.com/nayana224/autonomous-following-towing-robot.git
git clone https://github.com/roverrobotics-forks/serial-ros2.git
git clone https://github.com/Slamtec/sllidar_ros2.git

cd autonomous-following-towing-robot
```

`laser_filters`도 같은 `~/ros2_ws/src` 아래에 별도로 준비해야 합니다. 프로젝트용 fork와 검증된 revision은 추후 문서에 반영할 예정입니다.

### Laptop 개발 환경 (amd64 / CPU)

```bash
cd ~/ros2_ws/src/autonomous-following-towing-robot
./scripts/docker_build_laptop.sh
./scripts/docker_run_laptop.sh
```

Container 안에서:

```bash
./src/autonomous-following-towing-robot/scripts/build_ws.sh
source /workspace/install_laptop/setup.bash
```

### Jetson 배포 환경 (arm64 / CUDA)

```bash
cd ~/ros2_ws/src/autonomous-following-towing-robot
./scripts/docker_build_jetson.sh
./scripts/docker_run_jetson.sh
```

Container 안에서:

```bash
./src/autonomous-following-towing-robot/scripts/build_ws.sh
source /workspace/install_jetson/setup.bash
```

빌드가 완료된 Jetson에서 GUI를 시작하려면 호스트에서 다음과 같이 실행합니다.

```bash
./scripts/docker_run_jetson.sh bash -lc \
  'ros2 launch aftr_gui operator_system.launch.py'
```

전체 설치 조건, 모델·데이터 경로, 장치 연결 및 실제 운용 절차는 [설치 및 실행](docs/setup.md)과 [운용 안내](docs/operation.md)를 참고하세요.

## Workspace 구성

```text
~/ros2_ws/
├── src/
│   ├── autonomous-following-towing-robot/
│   ├── laser_filters/
│   ├── serial-ros2/
│   └── sllidar_ros2/
├── models/
│   └── fall_detection/
└── data/
    ├── paths/
    ├── maps/
    └── poses/
```

주요 런타임 파일은 호스트의 `models/`, `data/`에서 관리하며 컨테이너 내부에서는 `/models`, `/data`로 접근합니다. 실행 데이터와 모델 파일을 Git에 포함하지 않는 것이 권장됩니다.

## ROS 패키지

| Package | 역할 |
| --- | --- |
| `aftr_audio` | 안내음 및 이벤트 음성 |
| `aftr_bringup` | Robot base, controller, LiDAR bringup |
| `aftr_description` | URDF / Xacro 및 robot model |
| `aftr_fall_detection` | 낙상 감지 및 heartbeat |
| `aftr_gui` | Operator GUI |
| `aftr_hardware` | ros2_control hardware interface |
| `aftr_mode_manager` | 작업 흐름 및 안전 상태 관리 |
| `aftr_navigation` | Nav2 / localization 설정 |
| `aftr_path_manager` | 경로 기록·재생 및 pose 저장 |
| `aftr_slam` | SLAM Toolbox 설정 |
| `aftr_status_led` | 상태 LED |
| `aftr_teleop` | 수동 `/cmd_vel` 입력 |
| `aftr_tracking` | 작업자 추종 |

## Team

| 이름 | 담당 분야 | 담당 영역 |
| --- | --- | --- |
| 박재범 | Path Planning 및 Autonomous Navigation Support | `aftr_status_led`, `aftr_path_manager` |
| 이인표 | Firmware 및 System Integration | `aftr_audio`, `aftr_bringup`, `aftr_description`, `aftr_gui`, `aftr_hardware`, `aftr_mode_manager`, `aftr_navigation`, `aftr_slam`, `aftr_teleop` |
| 이재빈 | Mechanical Design | 기구부 설계 및 CAD |
| 이하나 (팀장) | Leg Tracking 및 Fall Detection | `aftr_tracking`, `aftr_fall_detection` |

## 문서

- [설치 및 실행](docs/setup.md)
- [운용 안내](docs/operation.md)
- [시스템 구조](docs/architecture.md)
- [ROS 인터페이스](docs/interfaces.md)
- [안전 정책](docs/safety.md)
- [개발 및 유지보수](docs/maintenance.md)
- [하드웨어 및 발표 참고 자료](docs/references/README.md)
