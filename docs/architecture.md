# 시스템 구조

자율주행 기반 물류 카트 견인 로봇(AFTR)은 ROS 2 Humble 기반으로 작업자 추종, 경로 기록, Mapping, Localization 및 저장 경로 재생을 수행합니다.

## 전체 구조

```text
Operator GUI
     │
     ▼
Mode Manager
     │
     ├── Tracking
     ├── Path Manager
     ├── SLAM
     ├── Navigation / Nav2
     ├── Fall Detection
     └── Audio / Status LED
     │
     ▼
Bringup / ros2_control
     │
     ▼
MD400T Motor Driver
     │
     ▼
BLDC In-Wheel Motors
```

## 패키지 역할

| Package | 역할 |
| --- | --- |
| `aftr_gui` | 운영자 화면 및 사용자 입력 |
| `aftr_mode_manager` | 전체 작업 흐름, 상태 전이, 프로세스 시작·종료 및 안전 제어 |
| `aftr_tracking` | 작업자 탐지 및 추종 |
| `aftr_fall_detection` | 낙상 상태와 detector heartbeat 제공 |
| `aftr_path_manager` | 경로 기록·저장·재생 및 pose 관리 |
| `aftr_slam` | SLAM Toolbox 기반 Mapping |
| `aftr_navigation` | Nav2 및 Localization |
| `aftr_bringup` | Controller, LiDAR 및 base bringup |
| `aftr_hardware` | MD400T ros2_control hardware interface |
| `aftr_audio` | Operator audio feedback |
| `aftr_status_led` | 로봇 상태 LED |
| `aftr_teleop` | 수동 속도 명령 |
| `aftr_description` | URDF/Xacro 및 robot model |

## 핵심 주행 기술

### LiDAR 기반 작업자 추종

2D LiDAR scan을 거리 기준으로 군집화하여 하체 후보를 검출하고, 두 다리 간 거리 조건을 이용해 작업자 후보를 선정합니다. 초기 탐색과 추종 단계에서 시야각을 다르게 운용하며, 작업자와의 거리 및 각도 오차를 속도 제어에 반영합니다. 장애물 감지 시 추종 주행을 정지하고 상태에 따라 재개합니다.

### 경로 기록 및 안전 경로 생성

로봇 위치를 map 좌표계에서 TF로 수집하고 거리 기반 필터링을 적용해 `x`, `y`, `yaw_rad` 경로점을 CSV로 저장합니다. 기록 경로를 재샘플링하고 점유 격자 지도에서 장애물 잔상과 자유 영역을 처리한 뒤 Distance Transform을 통해 장애물 이격거리를 평가합니다. 원본 경로 이탈량과 경로 연속성을 고려하여 재주행용 경로를 보정합니다.

### Wide-Turn 및 경로 재주행

지도 코너와 회전각을 분석해 Wide-Turn 구간을 검출하고, **3차 Bézier 곡선** 기반 외측 선회 궤적과 충돌 검사를 적용합니다. 실제 저장 경로 재주행은 안전 경로 생성 후 `map_corner_turn_path_csv_generator_node.py`에서 방향별 Wide-Turn 경로를 생성하는 순서로 수행합니다. 보정 waypoint를 ROS 2 Path로 변환하여 정방향 및 복귀 주행에 사용합니다.

### 모터 구동 및 견인 구조

`ros2_control` 인터페이스를 통해 MD400T BLDC 모터 드라이버를 제어하며 엔코더 정보를 기반으로 로봇 상태를 갱신합니다. 카트 연결부의 물리적 조향각 제한 래치는 급선회 시 과조향과 슬립을 완화하도록 설계했습니다.

## 기본 Workflow

```text
HOME
  ↓
FOLLOW / RECORDING_FOLLOW
  ↓
ALIGNMENT_DECISION
  ↓
ALIGNMENT 또는 LOCALIZING
  ↓
AUTONOMOUS_READY
  ↓
AUTONOMOUS_DRIVING
```

### 경로 저장 추종

```text
Base 준비
→ SLAM 시작
→ 새 /map 확인
→ Path Manager 준비
→ 경로 기록 시작
→ 작업자 추종 시작
```

### 위치 추정 준비

```text
기록 종료
→ 경로 / 지도 / pose 저장
→ SLAM 종료 확인
→ Nav2 시작
→ Initial Pose 전달
→ AMCL Pose 확인
```

## GUI와 Mode Manager 역할

GUI는 현재 상태를 표시하고 허용된 command를 전송합니다. Workflow 정책과 안전 판단은 `aftr_mode_manager`가 담당합니다.

GUI는 `/mode_manager/status`의 `allowed_commands`를 기준으로 버튼을 활성화합니다.

## 구동 명령 소유권

`/cmd_vel`은 한 시점에 하나의 제어 주체만 사용하도록 운용합니다.

- 작업자 추종: `aftr_tracking`
- 수동 정렬: GUI / teleop 계열
- 자율주행: Nav2

## 외부 패키지

다음 패키지는 Workspace의 별도 Git repository로 관리합니다.

- `laser_filters`
- `serial-ros2` (`serial`)
- `sllidar_ros2`
