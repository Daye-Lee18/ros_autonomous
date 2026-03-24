# 4. Jetcobot 카메라 사용하기 

VNC viwer를 사용할때 GUI로 네트워크 설정을 끄거나 키면 무선으로 Jetcobot에 연결할 수 없다. 유선으로만 해결 가능하기 때문에, VNC로 연결하지 마라 절대! 무의식적으로 습관이 나온다. 

## Setting 
라즈베리파이에 카메라와 제코봇을 연결한다. 

## VNC에서 웹캠 확인 

/dev 는 device 의 short term 으로 하드웨어를 연결할때마다 폴더? 파일이 생긴다. 
----> jetcocam0 이름으로 udev 규칙에 따라 생성된 디바이스 노드를 확인할 수 있다. 
udev는 한번 연결한 장치를 내가 원하는 이름으로 설정할 수 있다. 꽂을때마다 device0, 1, 2,... 으로 같은 device여도 다르게 잡혀서 코드에서 불편함이 있다. 
* device를 같은 이름으로 설정하는 command ? 
  
cf) udev (userspace /dev) is the linux kernel's device manager 
    * Dynamic device management: Unlike static /dev directories, udev creates device nodes instantly when a device is plugged in, and removes them when unplugged.
    * rule-based system : udev uses rules (stored in /etc/udev/rules.d) to detect specific hardware based on vendor ID, serial number, or port, allowing users to define custome device names, symlinkes, or actions 
    * persistent naming: Udev provides consisten naming for network interfaces and block devices across reboots, overcoming arbitrary naming based on detection order 
    * Common Uses of Udev Rules:
      * Automaticaaly renaming a network card to `eth0` or `my_usb_device`
      * Setting specific permissions (e.g. allowing a user to access a USB Arduino without `sudo`)
      * Auto-mounting external hard drives
* sudo systemctl restart NetworkManager
 * 이거한 후에도 ./wifi 어쩌고 안해도 됨. 
* ls /dev : to get device path in /dev folder 
* cv2 library, cv2.VideoCapture("device dev path")
* 항상 자원 낭비 방지로 자원 해체 및 윈도우 닫기 cap.release(), cv2.destroyAllWindows()
* 가상환경 열고, 위의 python 파일 터미널에서 실행 (python3 )
  * ubuntu에서는 python말고 python3로 해야 python3를 사용할 수 있는 건가? 
  
## Flask 를 통해 다른 PC에서 웹캠 확인 

같은 웹캠의 이미지를 여러 PC에서 볼 수 있게 한다. 
여러 대의 로봇과 PC를 운용할 경우 웬만한 네트워크 장비가 감당할 수 없다.

* flask 설치 
서버와 클라이언트에 필요 패키지를 가상환경에 설치한다. 
* 서버 코드: Flask, threading
  * 서버 코드에서도 IP를 잘 특정해야한다. 
* 클라이언트 코드: requests, threading 

````{admonition} Flask application and threading library 
:class: dropdown 

<server code>
```python
import threading 
from flask import Flask, Response 

frame_lock = treading.Lock()
app = Flask(__name__)
res = Response() # class object 

capture_thread = threading.Thread(target=capture_frames)
capture_thread.daemon = True 
capture_thread.start()

app.run(host='', port=, threaded=True)
```

<client code>
- client class로 동작 
```python

```

In a Flask app, you use a `threading.Lock()` to prevent "race conditions" when multiple threads (handling different web requests) acess and modify the same shared resouce, such as a global variable, a file, or an image. 
````


````{admonition} IP address 
:class: dropdown 

핑키 네트워크에서의 대왕은 핑키이다. 보통 마지막 번호가 1이다. 192.168.4.1 이고 이 네트워크에 접속한 PC는 할당을 받는데, 보통 번호가 2, 3, ...등으로 커진다. IP주소를 할당받아야 그 나라에 살 수 있다. 

핑키도 강의장 공유기로부터 네트워크를 받으면 또다른 IP를 할당받는다. 

그러면 핑키는 2개의 IP 주소를 가지고 있다. 이를 확인하는 방법은, pinky terminal에서 `ifconfig` 명령어를 치면된다. 그중 `AP`를 가진 것: 핑키가 나라, `IS`? 로 시작하는 것이 다른 곳에서 받은 IP주소. 

(두번째 사진)
강의장 공유기에 두 대의 로봇팔이 연결되어있고, 들의 각 네트워크가 `192.168.5.1`로 동일해도 상관없다. 
IP주소는 같지만, 다른 왕국이라서 괜찮다. 

- IP 주소는 “네트워크 안에서만 유효”
  - 서울에도 “101번 집” 있음
  - 부산에도 “101번 집” 있음
  - 주소(=IP주소) 가 같아서 도시 (=네트워크)가 다르니까 상관없음. 

또한 동적할당과 고정할당으로 고칠 수 있다. 

질문: pc에서 연결할때 다수의 pc에서 각각 접속해야하나? 
- 모두가 연결된 네트워크 (즉, 강의장공유기)안에 접속하면 된다. 
- 즉, robot은 강의장 네트워크에 접속하고, 
- 각 PC는 강의장 네트워크에서 접속하면된다? 
````

[flask link](https://testdriven.io/courses/learn-flask/routing/) 

![img1](../../_asset/jetcobot/flask1.png)
![img2](../../_asset/jetcobot/flask2.png)