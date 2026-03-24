# 통신 (communication) 

ROS 통신은 노드 (Node)라 불리는 프로세스들이 메시지, 서비스, 액션을 통해 데이터를 주고받는 분산 시스템이다. 총 3가지 방식을 통해 센서 데이터 처리, 제어 명령, 다중 로봇 협업을 위한 하드웨어 추상화를 제공한다. 

## ROS 핵심 통신 방식 

통로 흐름! 

````{admonition} 정리
:class: note 

- 토픽 (Topic): 데이터 흐름 
- 서비스 (Service): 짧은 요청/응답 
- 액션 (Action): 오래 걸리는 작업 + 피드백 
````
### 토픽 (Topic)

발행-구독(Publisher-Subscriber) 모델 기반의 비동기식 단방향 통신이다. 센서 데이터처럼 연속적인 스트림 데이터 전달에 적합하여, 1:N 통신이 가능하다. 토픽은 특정 데이터를 주고받는 것이 계속 되어야 하는 경우 토픽을 통해 데이터를 송수신하며, 예를 들어, 물류 센터에서는 모든 로봇들의 현재 상태 (일을 하는지 유휴 상태인지 등)를 통신할 때 사용한다. 

### 서비스 (Service)

요청-응답 (Request-Response) 모델 기반의 동기식 양방향 통신이다. 일회성 데이터 처리나 노드 간의 동기화가 필요한 경우에 사용된다. 예를 들어, 물류 센터의 통합 관제 시스템에서 로봇1에게 움직이라는 명령을 내릴때 사용한다. 

### 액션 (Action)

목표-결과 (Goal-Result) 모델 기반의 `비동기식` 양방향 통신이다. 로봇의 이동과 같이 장시간 소요되는 작업에 적합하며, 작업 도종 취소하거나 피드백을 받을 수 있다. 
예를 들어, 자율주행 `Nav2`를 통해 자율주행을 할 때 사용한다. 액션은 토픽과 서비스를 이용하여 설계되며, client-server 모델을 사용하는데, 그 로직은 다음과 같다. 

```md
Action Client Node (Goal Service Client) $rightarrow$ Action (Request, Goal Service) $rightarrow$ Action Server Node (Goal Service Server) $rightarrow$ Action Client Node (Result Service Client) $rightarrow$ Action (Request, Result Service) $rightarrow$ Action Server Node (Result Service Server) $rightarrow$ Action Server Node (feedback publisher) $rightarrow$ Action Client Node (Feedback Subscribier) $rightarrow$ repeating feedback process until the task ends ..... $rightarrow$  Action Server Node (Result Service Server) $rightarrow$  Action Client Node (Result Service Client)
```

### 동기식(Synchronous) vs. 비동기식(Asynchronous)

동기식은 요청을 보내고 응답이 올 때까지 멈춰있다가 응답이 오면 다음 코드를 실행한다. 따라서, 코드가 순서대로 실행되고 이해는 쉽지만 느릴 수 있다. ROS에선 `call()` 같은 방식이 있다. 반면, 비동기식은 요청 보내고 기다리지 않고 다음 작업을 수행한다. 특징으로는 동시에 여러 작업이 가능하고 콜백(callback)이나 future로 결과를 처리할 수 있다. ROS2에서는 기본 방식으로 `call_async()`로 채택된다. 다음 예시로 바로 이해해보자. 

```python
response = clinet.call(request)
print(response) # 응답이 도착한 후에서야 print를 한다. 
```
위의 코드에서는 응답이 도착한 후에야 print 코드가 진행되므로, 기다리는 동안 아무것도 하지 못한다. 

```python
future = client.call_async(request)

# 다른 작업 수행 
do_something()

# 나중에 결과 확인 
if future.done():
    print(future.result())

```

ROS2에서는 비동기를 권장하는데, 그 이유는 센터 처리 + 제어 + 통신을 동시에 해야하는데, 한 작업을 기다리면 전체 시스템이 멈추기 때문이다. 다만, 동기식은 순차적으로 진행되기 때문에 간단한 테스트에서 사용될 수 있고 실제 로봇 시스템에서는 비동기식을 채택하여 코드의 흐름이 병렬적으로 일어나게 한다. (병렬적 = 하나가 끝나지 않아도 다른 프로세스가 진행될 수 있다.)

| 방식 | 통신 형태 | 동기/비동기 | 특징 |
|---|---|---|---|
| Topic | Pub/Sub | 비동기 | 스트림 데이터 |
| Service | Req/Res | 동기/비동기 (주로 동기) | 빠른 요청 |
| Action | Goal | 비동기 | long task| 

## ROS 통신 구조의 특징 

1. 노드(Node): **ROS의 기본 실행 단위**로, 카메라, 센서, 모터 제어 등 특정 기능을 수행하는 프로세스 
2. 마스터 (Master/ROS1) 또는 디스커버리(Discovery/ROS2): 노드들이 서로를 찾고 연결할 수 있도록 지원하는 이름 서비스(Name Service) 기능을 제공한다. 
3. DDS (Data Distribution Service): ROS2에서 채택한 고성능 미들웨어로, 다중 로봇 시스템에서 안정적으로 실시간성 높은 통신을 지원한다. 

### Data Distribution Service (DDS)

DDS는 ROS2에서 노드들이 서로 통신할 수 있게 해주는 "미들웨어" 소프트웨어이다. 미들웨어란 </u> ROS Node들의 중간에서 통신을 대신 처리해주는 소프트웨어</u>로 구조로 본다면 다음과 같다. 

```md
ROS Node $rightaroow$ [DDS] $rightaroow$ ROS Node
```

구체적으로, 우리는 topic publish, service call을 하면 DDS가 </u>***데이터를 전달하고, 네트워크를 처리하며 연결을 관리***</u>한다. DDS가 하는 일을 정리하면 다음과 같다. 

1. 자동 discovery (노드 자동 찾기): "같은 네트워크에 있으면" 자동 연결, ROS1처럼 master가 필요없다. 
2. 데이터 전송 (pub-sub): topic 기반 통신 처리 
3. QoS (Quality of Service): 신뢰성, 속도, 버퍼 관리 
4. 네트워크 관리: UDP 기반 통신, 멀티캐스트 지원 

ROS1에서는 노드끼리 연결하는 ROS Master 중앙 서버가 필요하였으나 마스터가 죽으면 전체 다운되고, 확장성이 부족한 문제가 있었다. 따라서, DDS를 사용하여 중앙 서버가 없어지고 실시간성을 지원하며 안정성이 높은 로봇 여러대에서도 잘 동작하는 DDS를 사용하게 되었다. 즉, 사용자가 사용하는 것은 ROS API (사용자 인터페이스)이고 실제 통신은 DDS가 한다. 

예를 들어, 아래처럼 코드를 짜면 

```python
publisher.publish(msg)
```

실제 내부에서는 

```
publish() <br>
$rightarrow$ ROS layer <br>
$rightarrow$ DDS serialize<br>
$rightarrow$ 네트워크 전송<br>
$rightarrow$ 다른 노드 DDS 수신<br>
$rightarrow$ ROS layer 전달<br>
```

ROS2의 아키텍쳐(architecture)는 다음과 같다. 

![img1](../../_asset/ros/ros-architecture.jpg)

그림에서 볼 수 있듯이, 같은 DDS 구현체를 여러 노드가 사용하며, 각 '프로세스(process)' 단위마다 DDS에 participant로 등록된다. 

[DDS]<br>

 ├── Participant (Node A)<br>
 │     ├── Publisher<br>
 │     └── Subscriber<br>
 │<br>
 ├── Participant (Node B)<br>
 │     ├── Publisher<br>
 │     └── Subscriber<br>

## 핵심 통신 용어 (Core Communication Terms)

* 메시지(Message): 노드 간 주고받는 데이터의 기본 단위 
* 토픽 (Topic): 비동기식, 지속적인 데이터 스트림 방식, 게시-구독(Publish-Response) 모델
* 발행/발행자 (Publish / Publisher): 데이터를 보냄 
* 구독/구독자 (Subscribe/Subscriber): 데이터를 받음 
* 서비스(Service): 동기식, 요청-응답 (Request-Response)방식, 클라이언트-서버(Client-Server) 
* 액션(Action): 긴 작업, 중간 피드백 및 결과 반환 
* 노드(Node): 연산 처리를 하는 "최소 단위" 프로세스 
* 미들웨어(Middleware): ROS 통신을 관리하는 소프트웨어 
* DDS(Data Distribution Service): ROS2에서 사용되는 핵심 통신 프로토콜 

### 사용 예시 문장
- 노드들은 서로 통신한다.(The nodes communicate with each other) 
- 토픽에 메시지를 발행 중이다. (I'm publishing a message to a topic)
- 토픽을 구독 중이다. (I'm subscribing to a topic)
- 서비스 호출이 응답을 기다리고 있다. (The service call is waiting for a response)