# Node 


## 노드 실행 

node란, ROS에서 실행할 수 있는 최소 단위라고 생각하면 된다. 

이런 노드를 실행하기 위해선, `ros2 run` 명령어를 사용하면 된다. 

```bash
ros2 run {PKG Name} {Node Name}
```

## 노드 리스트 

```bash
ros2 node list 
```

```
/turtlesim
```

`list` command로 알게된 하나의 노드 정보를 알고 싶은 경우에는 아래 커맨드를 사용하면 된다.

```bash 
ros2 node info <Node name>
```

`info`를 통해서 알 수 있는 정보는 다음과 같다. 

* Subscribers
* Publishers
* Service Servers 
* Service Clients 
* Action Servers 