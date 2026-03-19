# PinkyPro LED

[Robot] 터미널 
cd ~/pinky_pro/src/pinky_pro

git pull origin rpi4 # 강사가 코드 바꿈 (LED부분만 바꿈)

cd ~/pinky_pro # colcon build는 workspace에서 해야함 

colcon build --packages-select pinky_led # 바뀐 LED 부분만 colcon build 가능 







