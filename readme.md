# URDF Rviz2 시각화


## [ROS2 Humble] 외부 URDF 파일 Rviz2 시각화 및 디버깅 가이드

## 🎯 개요

Isaac Gym이나 Gazebo 같은 무거운 시뮬레이터 환경을 전체 셋업하지 않고, **ROS2 Rviz2만을 이용하여 로봇의 URDF(외형, TF 트리, 조인트 한계치)를 빠르게 시각화하고 검증**하는 방법을 정리한다. 실제 하드웨어에 코드를 올리기 전 필수적인 디버깅 단계다.

---

## 🛠️ 1. 사전 준비 (Prerequisites)

### 1.1 필수 패키지 설치

URDF를 Rviz2에 띄우고 슬라이더로 관절을 제어하기 위한 기본 패키지들을 설치한다.

```bash
sudo apt update
sudo apt install ros-humble-urdf-tutorial ros-humble-joint-state-publisher-gui ros-humble-xacro
```

### 1.2 ROS2 환경 소싱 (Sourcing)

새로운 터미널을 열 때마다 ROS2 명령어를 인식하도록 환경을 불러온다.

```bash
source /opt/ros/humble/setup.bash
```

> **💡 Tip:** 매번 입력하기 귀찮다면 `~/.bashrc` 파일 맨 아래에 위 명렁어를 추가해두면 편리하다.
> 

---

## 🚀 2. 실행 방법 (The Happy Path)

터미널에 아래 명령어를 입력하여 런치 파일을 실행한다. `model:=` 뒤에는 실행하고자 하는 URDF 파일의 **절대 경로**를 입력한다.

```bash
ros2 launch urdf_tutorial display.launch.py model:= ${경로}
```

- **실행 결과:** Rviz2 창(3D 뷰어)과 Joint State Publisher GUI 창(관절 제어 슬라이더)이 동시에 열리며 로봇 모델을 시각적으로 확인하고 움직여볼 수 있다.

---

## 🚨 3. 트러블슈팅 및 시행착오 정리 (Lessons Learned)

이번 튜토리얼을 진행하며 겪었던 주요 에러와 해결 방법을 기록한다. **경로 문제와 명령어 오사용**이 주된 원인이었다.

### 🐛 실수 1: `ros2 run` vs `ros2 launch` 혼동

- **에러 메시지:** `No executable found`
- **발생 상황:** `ros2 run urdf_tutorial display.launch.py ...` 명령어를 입력했을 때 발생.
- **원인 및 해결:**
    - `ros2 run`은 단일 실행 파일(노드)을 실행할 때 사용한다.
    - `display.launch.py`와 같이 여러 노드(Rviz2, Joint State Publisher 등)를 한 번에 켜는 런치 파일은 **반드시 `ros2 launch` 명령어를 사용해야 한다.**

### 🐛 실수 2: 파일명 불일치로 인한 Xacro 파싱 에러

- **에러 메시지:**Plaintext
    
    `FileNotFoundError: [Errno 2] No such file or directory: '/home/jae/urdf_study/src/XBot_URDF/urdf/XBot.urdf'
    ...
    AttributeError: module 'xml' has no attribute 'parsers'`
    
- **원인 및 해결:**
    - 실제 디렉토리에는 `XBot-L.urdf` 파일이 있었으나, 명령어에 `XBot.urdf`라고 잘못된 파일명을 입력하여 발생했다.
    - 아래 발생한 `AttributeError`는 파일을 찾지 못해 발생한 첫 번째 에러를 시스템이 출력하려다 꼬인 **부차적인 버그**이므로 무시해도 된다.
    - 항상 `ls -l` 명령어로 실행하려는 경로에 **정확한 파일명이 존재하는지 확인**하는 습관을 들인다.

### 🐛 실수 3: 허공에 좌표계 축만 보이고 로봇 외형(3D)이 안 보일 때

- **발생 상황:** 명령어는 에러 없이 실행되어 Rviz2가 켜졌으나, 로봇 껍데기는 없고 뼈대(TF 축)만 보임.
- **원인:** URDF 파일 내부에 로봇 디자인 파일(.STL, .DAE)을 불러오는 `<mesh filename="...">` 경로가 `../meshes/base_link.STL` 처럼 **상대 경로**로 되어 있어서 Rviz2가 파일을 찾지 못한 것.
- **해결 방법:** 해당 패키지를 정식 빌드(`colcon build`)하지 않은 상태에서 외부 URDF를 단독으로 열 때는, 모든 mesh 경로를 **절대 경로(`file:///...`)**로 수정해야 한다.Bash
    
    **[수정 예시]**
    
    ```bash
    <mesh filename="../meshes/base_link.STL" />
    ```
    
    ```bash
    <mesh filename="file:///home/jae/urdf_study/src/XBot_URDF/meshes/base_link.STL" />
    ```
    

---

> **💡 Next Step:** Rviz2에서 로봇 관절이 의도한 대로 잘 움직이는 것을 확인했다면, 다음 단계는 이 URDF를 기반으로 `robot_state_publisher` 노드를 구성하고, 실제 하드웨어(CAN 통신 모터)와 데이터를 주고받을 하드웨어 인터페이스(ros2_control)를 작성하는 것이다.
>