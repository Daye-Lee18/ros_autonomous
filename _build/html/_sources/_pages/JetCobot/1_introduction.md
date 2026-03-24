# 1. JetCobot Introduction and Setting 

## JetCobot 모델 정보 

Elephant Robotics 
제품명: myCobot 280 for Arduino
(보통 로봇 이름 뒤에 숫자는 payload)

[제품 링크](https://www.elephantrobotics.com/en/mycobot-en/?utm_term=ur5e%20robot&utm_campaign=mycobotpi-%E7%AB%9E%E4%BA%89%E5%AF%B9%E6%89%8B%E7%BB%84-%E5%93%81%E7%89%8C%E7%9B%AE%E7%9A%84%EF%BC%8C200%E4%BB%A5%E4%B8%8A&utm_source=adwords&utm_medium=ppc&dm_acc=3657328933&dm_cam=16587103178&dm_grp=137310206329&dm_ad=587890743054&dm_src=g&dm_tgt=kwd-3500001&dm_kw=ur5e%20robot&dm_mt=b&dm_net=adwords&dm_ver=3&gad_source=1&gad_campaignid=16587103178&gbraid=0AAAAACw-MsVWa0x6l1Tn8FpgRaP9SouiQ&gclid=Cj0KCQjwmunNBhDbARIsAOndKpkYqo1pCAmmomr9ioV39IgdYTdVkP_VYm0YEEcM5LMGTQoJm4sIVFkaAjG9EALw_wcB)

먼저 하드웨어 스펙을 확인해서, 할 수 있는 것을 찾기. 
- Joint Rotation Range: 가동범위가 정해져있음. 
- Working Radius 
- Payload 
- Joint Maximum Speed 

## JetCobot
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

칼만필터, 파티클 필터 

구글 block-nerf 