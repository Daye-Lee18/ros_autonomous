# PinkyPro와 연결 


## Setting 

### [처음 한 번 필요] 핑키프로 패키지 다운 

PC에서도 pinky pro 패키지를 사용하기 위한 작업이다. 

```bash
# 1. 깃헙 디렉토리 다운 
mkdir -p ~/pinky_pro/src 
cd ~/pinky_pro/src 
git clone https://github.com/pinklab-art/pinky_pro.git

# 2. 의존성 설치 
sudo rosdep init # rosdep 설정 파일을 생성 및 초기화하여 기본 ROS '패키지 의존성' 소스 목록을 등록한다. (python의 pip과 동일)
rosdep update # 등록된 소스 목록에서 의존성 메타데이터를 내려받아 로컬 데이터베이스를 최신 상태로 갱신 

# 3. PinkyPro 패키지의 의존성 패키지들 설치 
cd ~/pinky_pro
rosdep install --from-paths src --ignore-src -r -y 

# 4. 빌드 
colcon build 

# 5. [빌드 에러시] 의존성 패키지 설치 command 
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install libgz-sim8-dev
```

### PinkyPro를 와이파이와 연결 [처음 한 번만]

아래 과정을 하면, PinkyPro에 와이파이관련 정보가 저장된다. 처음 설정하고 다음에 하려고 하면 안된다고 함. 

```bash
# 1. sd 카드를 로봇에 장착 후 부팅 완료 후 PC를 Pinky의 네트워크로 와이파이 연결 
# SSID: pinky_XXXX (4자리), 비밀번호: pinkypro 

ssh pinky@192.168.4.1 # [PC]에서 로봇 터미널에 접속 
nmcli device wifi list # [Pinky, ROBOT] 와이파이 리스트 확인 
./wifi_setup.sh # [Pinky, ROBOT] PinkyPro를 다른 네트워크에 연결, 로컬 PC -> Pinky -> 강의장내 네트워크 연결로 로컬 PC에서 Pinky와 강의장 내 네트워크 둘 다 사용할 수 있도록 한다. 
# ping 8.8.8.8 인터넷 연결 확인 
# Enter WiFi SSID: AIE_509_5G (강의내 와이피아)
# Enter WiFi Password:  addinedu_class1
```

### Local에서 PinkyPro와 연결 

1. PinkyPro 스위치 켜기 -> 와이파이네트워크 ssid를 보고 해당 네트워크와 접속한다. 

![pinky_lcd](../../_asset/pinky/1.png)
   
1. ssh 접속 
- ssh pinky@192.168.4.1 로 vscode에서 로봇으로 접속
- 비밀번호: 1 
- 로봇안의 폴더 및 파일 확인 가능 

1. PC와 로봇의 ROS_DOMAIN_ID 설정 (내 아이디 116)

```bash
nano ~/.bashrc 
# ROS_DOMAIN_ID = 116 <- PC터미널에서 echo로 확인 가능 
source ~/.bashrc 
echo $ROS_DOMAIN_ID # 변경 확인 
```

1. source ~/.bashrc 로 변경된 설정 모든 터미널에서 업데이트해주기 
   
```bash
battery # [Pinky] battery 상태 확인 
```

### PinkyPro 패키지 사용시 

```bash
source ~/pinky_pro/install/local_setup.bash # [PC?] local PC에서 pinky_pro package를 사용하기 위해 설정 
```
