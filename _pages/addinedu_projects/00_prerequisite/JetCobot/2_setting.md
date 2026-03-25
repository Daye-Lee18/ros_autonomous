
# 2. JetCobot 연결  

## 전원 연결 
1. sd 카드 금속 단자 위로 연결 
2. c-type charger 연결 
3. 로봇팔 (동그라미 전원) 연결: 전원 들어가면 관절이 고정되기 때문에, 원하는 자세에서 전원 연결이 되도록 한다. (얼굴 조심)

로봇팔의 컨트롤보드 (c-type)과 라즈베리파이 (주황색 박스 안 A-type)에 연결. 

## PC에서 JetCobot의 네트워크로 연결 

JetCobot 위의 붙어있는, SSID와 비밀번호 확인하여 PC에서 같은 이름의 와이파이 연결 

* ssid는 jetcobot_xxxx (4자리 랜덤)
* 비밀번호는 jetcobot 

해당 네트워크의 IPv4 탭에서 DNS를 8.8.8.8로 설정 

## PC에서 ssh로 로봇 접속 

```bash
ssh jetcobot@192.168.6.1 # 비밀번호 1 
```

## [첫 한번만] JetCobot 에서 공용 와이파이 연결 

이미 되어 있는 경우, 다시 하면 연결이 안되므로 주의 
```bash
nmcli device wifi list # [Jetcobot]
./wifi_setup.sh # [Jetcobot] 와이파이 정보 세팅 
# Enter WiFi SSID: AIE_509_5G (강의내 와이피이)
# Enter WiFi Password:  addinedu_class1
```

연결하려는 ssid가 없다면 재부팅후 다시 확인 

## PC에서 vscode로 로봇 접속 

`jetcobot@192.168.6.1`로 접속 
비밀번호: 1 

## VNC Viewer: GUI 를 위한 원격 접속 도구 [처음 한번만]

원격제어 프로그램. 
NOTE!! 필요한 경우만 사용하면 되고, 터미널과 VSCode로 ssh접속해서 개발! 

[링크](https://www.realvnc.com/en/connect/download/viewer/)로 들어가서 RealNVC Viewer 를 설치한다. deb-64? 고르면 됨. 

```bash
cd Downloads/ # [PC]
sudo apt install ./VNC-Viewer-7.13.1-Linux-x64 # [PC] 파일경로를 apt install에 전달 
```

PC에서 NVC Viewer 설정 
- [PC] 설치 후 NVC Viewer 앱 실행 
- 로그인없이 무료 기능만 사용 ('Use RealVNC Viewer without signing in)
- RVNC Connect 칸에 접속하려는 PC의 주소 입력
  - 192.168.6.1
- Continue 
- 접속하려는 PC의 유저명과 암호 입력 
  - Username: jetcobot 
  - Password: 1 
- 접속 성공 후 

## PC에서 Jetcobot jupyter notebook 접속하기 

제코봇 터미널에서 jupyter notebook을 먼저 실행해놔야 PC에서 원격으로 jupyter notebook으로 연결할 수 있다. 
- [Robot] jupyter notebook 으로 먼저 실행 후 
- PC 웹브라우저에서 위의 로봇터미널에 뜬 IP주소를 입력하면된다.  
  - 로봇 PC에 jupyter notebook으로 접근 가능함. 
  - 예를 들어, 192.168.6.1:8888

