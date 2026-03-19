# 통신 (communication) 

ROS 통신은 노드 (Node)라 불리는 프로세스들이 메시지, 서비스, 액션을 통해 데이터를 주고받는 분산 시스템이다. 총 3가지 방식을 통해 센서 데이터 처리, 제어 명령, 다중 로봇 협업을 위한 하드웨어 추상화를 제공한다. 

## ROS 핵심 통신 방식 

### 토픽 (Topic)

발행-구독(Publisher-Subscriber) 모델 기반의 비동기식 단방향 통신이다. 센서 데이터처럼 연속적인 스트림 데이터 전달에 적합하여, 1:N 통신이 가능하다. 

### 서비스 (Service)

요청-응답 (Request-Response) 모델 기반의 동기식 양방향 통신이다. 일회성 데이터 처리나 노드 간의 동기화가 필요한 경우에 사용된다. 

### 액션 (Action)

목표-결과 (Goal-Result) 모델 기반의 비동기식 양방향 통신이다. 로봇의 이동과 같이 장시간 소요되는 작업에 적합하며, 작업 도종 취소하거나 피드백을 받을 수 있다. 

## ROS 통신 구조의 특징 

1. 노드(Node): ROS의 기본 실행 단위로, 카메라, 센서, 모터 제어 등 특정 기능을 수행하는 프로세스 
2. 마스터 (Master/ROS1) 또는 디스커버리(Discovery/ROS2): 노드들이 서로를 찾고 연결할 수 있도록 지원하는 이름 서비스(Name Service) 기능을 제공한다. 
3. DDS (Data Distribution Service): ROS2에서 채택한 고성능 미들웨어로, 다중 로봇 시스템에서 안정적으로 실시간성 높은 통신을 지원한다. 

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

예시 문장)
- 노드들은 서로 통신한다.(The nodes communicate with each other) 
- 토픽에 메시지를 발행 중이다. (I'm publishing a message to a topic)
- 토픽을 구독 중이다. (I'm subscribing to a topic)
- 서비스 호출이 응답을 기다리고 있다. (The service call is waiting for a response)