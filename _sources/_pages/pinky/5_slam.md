# 5. PinkyPro SLAM 

세팅을 하기 위해 [로컬과 PinkyPro 연결]() 파트를 먼저 하고 와야 한다. 

## SLAM 

comment 되어 있는 곳에 [Robot] / [PC]는 터미널의 종류를 의미한다. 

```bash 
ros2 launch pinky_bringup bringup_robot.launch.xml # [Robot] 로봇을 준비시킨다. 센서켜고 모터 켜고 데이터 송수신같은 것을 준비한다. 

ros2 launch pinky_navigation map_building.launch.xml # [Robot] SLAM 실행 

source ~/pinky_pro/install/local_setup.bash # [PC] local PC에서 pinky_pro package를 사용하기 위해 설정 
ros2 launch pinky_navigation map_view.launch.xml # [PC] RviZ 실행 

ros2 run teleop_twist_keyboard teleop_twist_keyboard # [Robot/PC]에서 로봇 움직임 조작, pinky봇은 max speeds로 주행되어있도록 기본 설정됨. "저속도"로 맵핑 필요 

ros2 run nav2_map_server map_saver_cli -f "<저장할 맵이름>" # [Robot] 따옴표 필수, 맵 저장, 지정 이름으로 pgm 파일 (image) 과 yaml 파 (metadata) 이 동시에 저장된다. 
```

## 맵생성시 팁

1. 지도 만들때 가능한 특징 겹치지 않도록 
- '대칭성'이 있는 간단한 네모, 원의 경우 분간하기 어려우므로, 대칭되지 않는 맵을 만들자. 

2. 맵을 생성할 때는 저속으로 지도 생성 
- 로봇을 teleop keyboard로 주행하며 맵을 생성할 때는 z키로 속도를 약간 줄이고 맵 생성 
- 회전을 할 때 k를 누르면서 천천히 회전하도록 하기 
- 빠르면 지도가 틀어질 수 있음

3. 평평한 바닥에서 지도 작성하기 
- 경사진 곳은 다른 것을 사용해서 맵핑 
- 라이다는 원래 같은 높이의 객체를 인식하는데, 경사가 기울어지면 센서가 바닥이나 천장을 인지해서 실제 맵에 없는 것을 맵으로 인식한다. 

4. 주행시, 이미지 편집 도구를 이용해 없어진 부분을 채우거나 필요없는 노이즈를 삭제해서 다듬어도 된다. 
- lazer스캔시 확률 계산할 때 사용되므로 예쁘게 색깔 의미를 고려해 다듬어도 된다.
- 검은색: 장애물 
- white: free space 
- 회색: unkown 
