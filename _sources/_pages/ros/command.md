# Command for ROS 

## Linux 
* alias 
* sudo poweroff 
* echo "" >> <file name>
* echo $환경변수 

## ROS 

###  패키지 및 환경 변수 인식 
```bash
source <setup.bash / local_setup.bash>
```
* 현재 '터미널' 세션에 ROS 패키지, 노드, 라이브러리, 그리고 환경 변수의 위치를 인식시킨다. (`ROS_PACKAGE_PATH`, `LD_LIBRARY_PATH`, `PATH` 등의 환경 변수)
* 'source'로 설정된 환경 변수는 해당 터미널(세션) 내에서만 유효하다. 
* '내가 만든(혹은 설치한) 패키지가 여기 있으니, 터미널아, 이제부터 이 명령어를 알아들어!' 

### 노드 실행 
```bash
ros2 run <package name> <node name>
```

### 로봇 시스템 한 번에 실행 및 관리 
```bash
ros2 launch <package name> <launch file>
```
  * 로봇 시스템을 한 번에 실행 및 관리한다. 개별 노드를 일일이 실행할 필요가 없어 시스템 구동을 간소화한다. 
  * 주로 Python을 기반으로 한 `.launch.py` 파일을 사용한다. 

### 의존성 패키지 분석 및 자동 설치 
```bash
rosdep install --from-paths src --ignore-src -r -y
```
* ROS 패키지를 빌드하거나 실행하는데 필요한 시스템 의존성을 자동으로 식별하고 설치 
* package.xml 파일에 명시된 의존성 패키지를 분석해 apt-get 등으로 필요 라이브러리를 자동으로 설치 
* 현재 워크스페이스(src폴더) 내의 패키지들을 스캔하여 필요한 모든 의존성을 설치
* sudo rosdep init: rosdep을 처음 사용할 때 초기화하는 명령어 (한번만 실행)
* rosdep update: 의존성 패키지 목록을 최신으로 업데이트 

