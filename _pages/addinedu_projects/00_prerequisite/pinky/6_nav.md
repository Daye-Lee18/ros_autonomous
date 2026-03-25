# PinkyPro Navigation

````{admonition} What to learn
:class: note 

* Nav2를 사용하여, Rviz에서 직접 목표 지점(Goal spot)을 설정하지 않고 nav2 를 이용해 goal 좌표를 생성하고 자율 주행하는 것을 배운다. 
* 또한, 자율주행동안 nav2를 이용해 남은 거리 (feedback.distance_remaining) 를 출력할 수 있으며, 주행 완료 후에는 주행 결과(nav.getResult())를 성공/실패/취소 됐는지 확인할 수 있다. 
````

세팅을 하기 위해 [로컬과 PinkyPro 연결](2_connection.md) 파트를 먼저 하고 와야 한다. 
앞서 mapping (SLAM) 을 하고 저장한 메타데이터가 있는 .yaml 파일을 이용해 주행할 수 있다. 


## 핑키의 Rviz 로 주행 시작/목표 설정후 자율 주행  

```bash

ros2 launch pinky_bringup bringup_robot.launch.xml # [Robot] pinky 준비 

ros2 launch pinky_navigation bringup_launch.xml map:=저장한 맵이름.yaml # [Robot] 네비게이션 관련 준비 

ros2 launch pinky_navigation nav2_view.launch.xml # !!![PC] Rviz실행 (SLAM 용 Rviz와 상이함에 주의!)
```

Rviz UI에서, 

* 초기 위치 선택: "2d Pose Estimation" 버튼 선택 (현재 라이다와 맵 일치 시켜주기) 
* 종료 지점 선택: "Nav2 Goal" 클릭 -> 도착할 위치와 도착했을 때 바라볼 방향으로 드래그 


경로 생성을 할 때, 최단 경로 선택을 하려는데 그래서 벽을 타려고 한다. 충돌 위험 vs. 최단 경로 선택 중에서 로봇이 고민하게 되는데, waypoint를 중간 경로 지점으로 설정하는 것 같음. 

## Jupyter로 Nav2 를 이용해 자율 주행 

````{admonition} Nav2
:class: dropdown 

Nav2 (navigation stack) \
|-- planner \
|-- controller  \
|-- bt_navigator \
|-- costmap \
|-- localization \

으로 여러 노드들로 구성되어 있다. 
````

### 주피터 설치 [한 번만 실행]

```bash
pip3 install jupyter --break-system-packages
```

### 준비 단계 

핑키봇 켜고, 주행모드와 지도를 켠 후, 주피터 노트북을 실행해준다. 

```bash
ros2 launch pinky_bringup bringup_robot.launch.xml # [Robot] 핑키 bringup

ros2 launch pinky_navigation bringup_launch.xml map:=저장한 맵이름.yaml # [Robot] 내비게이션 실행 

ros2 launch pinky_navigation nav2_view.launch.xml # [PC] nav2_view.rviz 실행 (SLAM과 다름)

# Rviz에서 2D Pose Estimate 버튼을 눌러 라이다와 맵 일치 시켜주기 
# !!!! Jupyter 실행 전 ROS2 관련 환경변수 (특히, source setup.bash)가 적용된 상태에서 Jupyter 실행 
source /opt/ros/jazzy/setup.bash
source ~/pinky_pro/install/local_setup.bash   # (네 workspace)
jupyter notebook #[Robot] Initiates a local web server to run the application in your browser. 
```

### 주피터 파일에서 nav.goToPose() 를 이용한 Nav2 로 주행 

새로운 주피터 파일에서 코드 작업을 시작한다. 

```python
from nav2_simple_commander.robot_navigator import BasicNavigator
import rclpy 

rclpy.init()
nav = BasicNavigator()
```
nav2가 실행되었는지 확인한다. 

```python
nav.waitUntilNav2Active() # 'Nav2 is ready for use!' will be printed 
```

로봇의 각도 (degree)를 쿼터니언으로 변환하는 함수 선언 
```python 
import math 
import tf_transformations 

def get_quaternion_from_yaw(yaw_degrees):
    yaw_radians = math.radians(yaw_degrees)

    quaternion = tf_transformations.quaternion_from_euler(0, 0, yaw_radians)
    return quaternion 
```

/amcl_pose 토픽을 구독하는 노드 생성. '로봇의 현재 위치'를 출력하기 위한 노드이다. 

````{admonition} /amcl_pose
:class: dropdown 

Nav2의 localization을 담당하는 노드가 AMCL이고 nav2_amcl이 /amcl_pose를 생성. 
보통, nav2_bringup launch 를 하면 자동으로 같이 실행됨. 
````
```python
sub_amcl = rclpy.create_node('sub_amcl')

def amcl_callback(msg):
    print("===")
    print(msg.pose.pose.position)
    print(msg.pose.pose.orientation)

from geometry_msgs.msg import PoseWithCovarianceStamped

sub_amcl.create_subscription(PoseWithCovarianceStamped, 'amcl_pose', amcl_callback, 10)
```

Nav를 이용해 Goal을 생성해보자. 
제자리에서 회전하는 것이기 때문에, Goal의 목표 위치는 현재 로봇의 위치로 설정해준다. 

현재 로봇의 위치는 아래의 세 개 중에서 가능하나, /amcl_pose를 확인하는 게 가장 좋다. 
```bash 
ros2 topic echo /clicked_point  # Rviz에서 `Publish Point`로 클릭한 위치 
ros2 topic echo /amcl_pose # 실제 로봇 위치 
ros2 run tf2_ros tf2_echo map base_link # map에서 base_link 변환하여 현재 로봇의 pose를 알려준다. 
```

````{admonition} nav goal 좌표 PoseStamped 
:class: dropdown 

Nav goal은 "글로벌 좌표계(map)" 기준 absolute pose를 사용한다. 
즉, 현재 기준으로 180도 돌아라 (상대좌표)가 아닌, map 기준으로 이 위치 + 이 방향으로 가라 (절대좌표)의 의미이다. 

* map(global, 고정) <- odom (local drift 있음) <- base_link (로봇몸)
    * map: SLAM, localization 기준 
    * odom: 상대적 이동 
    * base_link: 로봇 자체 좌표, 앞으로 1m 이런 거 할 때 

pose.position.{x, y, z}, pose.orientation.{x, y, z, w} 즉, 해당 포즈를 취하라는 것이다. 현재 로봇좌표계에서 180도를 돌아라 이게 아니라, 절대 좌표계에서 180도 이미 회전되어 있는 상황이라면, orientation 움직이지 않을 것이다. 
````

```python
from geometry_msgs.msg import PoseStamped 

goal_yaw = 180 
q = get_quaternion_from_yaw(goal_yaw)

goal_pose = PoseStamped()
goal_pose.header.frame_id = 'map'
goal_pose.header.stamp = nav.get_clock().now().to_msg()

goal_pose.pose.position.x = ? # ? 채우기 
goal_pose.pose.position.y = ?  # ? 채우기 
goal_pose.pose.position.z = 0.0 

goal_pose.pose.orientation.x = q[0]
goal_pose.pose.orientation.y = q[1]
goal_pose.pose.orientation.z = q[2]
goal_pose.pose.orientation.w = q[3]

```

nav2에 goal 좌표를 보내면 자율주행을 시작한다. 이 경우, 목표 pose로 핑키가 움직인다. 

```python
nav.goToPose(goal_pose)
```

### 주행시, 상태 확인 

자율주행중에 실행해보면 주행 상태 확인이 가능하다. 

#### Nav2 로그 보기 

터미널에서 Nav2 `bringup` 로그를 보면 가장 디테일하다. 

1. /amcl_pose : localization 정상인지
2. /cmd_vel: 속도 명령 나가는지 
3. /plan 또는 global/local path 관련 시각화 : 경로가 생성되는지 
4. /tf : frame transform 정상인지 

#### Nav2 관련 토픽 확인하기 

* cpu 부하방지 (jupyter 멈춤방지)를 위해 timeout_sec 1초로 설정한다. 
    * while 문에서 아무 대기 없이 계속 loop을 돌면, cpu가 쉬지 않고 반복문을 실행해서 멈출 수 있다. 
    * 따라서, 최대 1초 정도 콜백/메시지를 기다리면서 쉬는 시간이 생긴다. 

* 해석 
  * 정상 주행 패턴 
    * distance_remaining 감소 
    * estimated_time_remaining 대체로 감소
    * number_of_recoveries 0또는 낮음 
  * 이상 패턴 
    * distance_remaining 정체 
    * navigation_time 만 증가 
    * number_of_recoveries 증가  
    * 경로 추종 실패, 장애물 회피 반복, localization 또는 planner/controller 문제 

```python
from rclpy.duration import Duration 
from IPython.display import clear_output # Jupyter 에서 IPyton 사용 

while not nav.isTaskComplete():
    clear_output(wait=True)
    rclpy.spin_once(sub_amcl, timeout_sec=1.0)

    feedback = nav.getFeedback()
    if feedback is not None:
        print("===")
        print(f"Distance remaining: {feedback.distance_remaining:.2f} meters")
        print(f"Navigation time: {Duration.from_msg(feedback.navigation_time).nanoseconds / 1e9:.1f} sec")

        if hasattr(feedback, 'estimated_time_remaining') and feedback.estimated_time_remaining.sec != 0:
            print(f"Estimated time remaining: {Duration.from_msg(feedback.estimated_time_remaining).nanoseconds / 1e9:.1f} sec")

        if hasattr(feedback, 'number_of_recoveries'):
            print(f"Recoveries: {feedback.number_of_recoveries}")

        if Duration.from_msg(feedback.navigation_time) > Duration(seconds=30.0):
            print("Navigation timeout. Canceling task.")
            nav.cancelTask()
```

````{admonition} nav.getFeedback()
:class: dropdown 

nav.getFeedback()은 보통 `NavigateToPose` 액션의 feedback이다. 
주로 다음과 같은 정보를 볼 수 있다. 
* distance_remaining : 목표까지 남은 거리
    * 계속 줄어들면 정상 주행 
    * 거의 안 줄어들면 정체
    * 늘어났다가 줄어들면 우회 중일 수 있음 
* navigation_time: 목표를 향해 움직이기 시작한뒤 지난 시간 
    * 너무 길어지면 실패 가능성 증가
    * 아래 코드처럼 일정 시간 넘으면 cancel 가능 
* estimated_time_remaining: 
    * 계속 감소하면 정상 
    * 계속 커지면 경로 재계획이 자주 일어나거나 장애물 회피 중일 수 있음 
* number_of_recoveries: 복구 동작 횟수, 중요! 
    * 값이 증가하면, nav2가 중간에 제자리 회전, 잠깐 후진, local costmap/global costmap clear, 재계획 등의 복구 행동을 시도했다는 뜻이므로,
    * 이 값이 늘어나면 단순히 주행이 느린 게 아니라 주행이 꼬이고 있다는 신호로 볼 수 있다. 
* current_pose: 현재 로봇의 위치 

* current_waypoint : followWaypoints를 할때만 있는 건가? , goToPose 

````

자율주행을 하다가 멈추고 싶다면, 자율주행 알고리즘 Nav2 bringup이나 pinky bringup를 꺼서 로봇 모터를 종료하면된다. 즉, nav2 ----- /cmd_vel topic으로 메세지전달 ---> pinky bringup 의 구조로 되어있기 때문이다. 


### nav.followWaypoints() 함수를 이용해 여러 좌표 보내기 

먼저 Map상에서 위치를 보기 위해, 터미널에서 Rviz상의 위치정보를 받는 /clicked_point 토픽을 구독하고  Rviz에서 위치를 찍는다. 

```bash
ros2 topic echo /clicked_point 
```

pinky의 nav2_view.rviz 맵에서 "Publish Point" 를 클릭해서 여러개 찍으면 터미널에서 해당 포인트를 볼 수 있다. 

다음으로, waypoint list를 생성하고 `nav.followWaypoints`를 사용하여 주행을 한다. 

```python
goal_pose_list = []

goal_yaw1 = 0
q1 = get_quaternion_from_yaw(goal_yaw1)

goal_pose1 = PoseStamped()
goal_pose1.header.frame_id = 'map'
goal_pose1.header.stamp = nav.get_clock().now().to_msg()
goal_pose1.pose.position.x = 2.9077
goal_pose1.pose.position.y = -1.24
goal_pose1.pose.orientation.x = q1[0]
goal_pose1.pose.orientation.y = q1[1]
goal_pose1.pose.orientation.z = q1[2]
goal_pose1.pose.orientation.w = q1[3]
goal_pose_list.append(goal_pose1)

goal_yaw2 = 180
q2 = get_quaternion_from_yaw(goal_yaw2)

goal_pose2 = PoseStamped()
goal_pose2.header.frame_id = 'map'
goal_pose2.header.stamp = nav.get_clock().now().to_msg()
goal_pose2.pose.position.x = -0.28
goal_pose2.pose.position.y = 0.248
goal_pose2.pose.orientation.x = q2[0]
goal_pose2.pose.orientation.y = q2[1]
goal_pose2.pose.orientation.z = q2[2]
goal_pose2.pose.orientation.w = q2[3]
goal_pose_list.append(goal_pose2)
```

waypoint 주행 시작 후 피드백 확인해보기

```python
nav_start = nav.get_clock().now()
nav.followWaypoints(goal_pose_list)
```

중요! 자원을 해제한다. 특히, 어제 LCD, LED사용하고 나서는 자원해제를 해야된다. 

```python
nav.lifecycleShutdown()
rclpy.shutdown()
```

### 자율주행
goal tolerence parameter: 핑키의 위치 추정과 목표 지점까지 소수점까지의 특정 오차 범위 안에 도착하면, 멈추도록 되어있음. 
- 거리와 각도 조정 가능 


