---
layout: post
title:  Docker network 구조
categories: cloud
---


## Network Namespace
​


![namespace](/assets/img/networknamespace.drawio.png)

도커는 컨테이너 생성시 네임스페이스라는 기술을 사용하여 격리된 환경을 제공한다.  
네임스페이스에는 여러 종류가 있는데, 자세한 설명은 
[여기](https://bluese05.tistory.com/11) 참고
​
네트워크 네임스페이스는 하드웨어의 네트워크 자원을 논리적으로 나누어 분할한다.  
각 네임스페이스는 다음과 같은 요소들로 구성된다
​
-   network interface
-   ip addresses
-   ip routing table
-   port numbers
-   /proc/net 디렉토리
​
## 가상 이더넷 인터페이스(veth interface)
​
가상으로 생성한 네트워크 인터페이스이다.  
네트워크 네임스페이스간의 터널 역할을 하여, 다른 네임스페이스의 네트워크 장치에 대한 브릿지를 생성할 수 있다.  
veth 는 항상 쌍으로 생성되어야 한다.  
이 쌍을 각각 다른 네트워크 네임스페이스에 연결하여 통신할 수 있도록 한다.  
자세한 설명은 [여기를 참고](https://man7.org/linux/man-pages/man4/veth.4.html)
​
---
​
## Docker network 구조
​
![docker](/assets/img/docker.drawio%20(2).png)
​
### docker0 인터페이스
​
docker0은 container가 통신하기 위한 virtual ethernet bridge 이다.
​
docker0의 ip는 도커 내부 로직에 의해 자동으로 할당받는다.
​
만약 docker0의 cidr를 변경하고 싶다면 [여기](https://bluese05.tistory.com/16) 를 참고하면 된다.
​
컨테이너를 생성하면, 컨테이너의 veth가 docker0 bridge에 하나씩 attach된다. 컨테이너 실행 시 eth0은 docker0의 cidr내에서 순차적으로 할당된다. 그림에서는 17.0.2, 17.0.3이 할당되었지만, 컨테이너를 중단시킨 후 재실행하는 등의 동작을 하면 할당받는 ip가 변경될 수도 있다.
​
컨테이너의 veth는 내부의 eth0과 쌍을 이루어 통신하며, 외부로 통신하기 위해서는 docker0 인터페이스를 거쳐야 한다.  
호스트에서 ip a 명령어를 실행하면, docker0 인터페이스는 은 확인할 수 있지만, 컨테이너에 할당된 eth0은 볼 수 없다. 컨테이너의 eht0은 호스트와 다른 네임스페이스에 존재하기 때문이다.  
컨테이너의 eth0주소를 확인하려면 다음과 같은 명령어를 사용한다
​
```
# docker exec ec2c85c542b0 ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:14:00:07  txqueuelen 0  (Ethernet)
        RX packets 1791744  bytes 2605508288 (2.6 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2054592  bytes 1745348088 (1.7 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
​
컨테이너 내부의 게이트 웨이를 확인해보면 docker0의 ip주소가 설정되어 있는것을 볼 수 있다
​
```
# docker exec ec2c85c542b0 route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         yrdy-2.local    0.0.0.0         UG    0      0        0 eth0
172.17.0.1      *               255.255.0.0     U     0      0        0 eth0
```
