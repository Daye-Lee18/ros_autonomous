# JetCobot Introduction and Setting 

## JetCobot 모델 정보 

## 로봇 환경설정 

robot 팔에는 라즈베리 파이 5가 탑재됨. 

Planning 
- global planner: global cost map에서 충돌하지 않도록 최적 경로 생성 
- local planner: local cost map 기반, 자율주행 중 장애물 피하도록 함. 


- `inflation_radius`와 `scaling_factor`를 같이 조절 
사람의 눈에는 갈 수 있을 것 같은데, cost map이 두꺼워서 경로가 안 생김. 
관련해서, Inflation radius 파라미터 설정. 
* 회피를 잘 못하겠지만, 갈 수 있는 경로는 많아지겠지 

Localization 과정 
motion model: 예측
sensor model: 보정 
