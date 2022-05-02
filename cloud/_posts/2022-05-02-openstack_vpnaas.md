---
layout: post
title:  Openstack VPNaas
categories: cloud
---


## Virtual Private Network as a Service (VPNaaS)란?

클라우드 기반 네트워크 인프라를 활용하여 VPN서비스를 제공하는 새로운 VPN기술로, Cloud VPN 또는 hosted VPN이라고도 합니다.  
VPNaas를 사용하여 사용자는 public 인터넷에 연결된 클라우드 플랫폼을 통해서 vpn에 접근할 수 있습니다.

## Openstack VPNaas

Openstack Queens버전부터 vpnass는 L3 agent에서 지원합니다.  
neutron-vpn-agent는 별도의 바이너리를 제공하지 않으며, L3 agent 에서 config 설정으로 vpnass를 사용할 수 있도록 변경되었습니다.  
[Queens Series Release Notes 참고](https://docs.openstack.org/releasenotes/neutron-vpnaas/queens.html)

#### Openstack network 구조


![네트워크구조](/assets/img/1aa-network-domains-diagram.png)

![vpnaas](/assets/img/vpnaas.drawio.png)

Openstack Neutron은 network node에 Agent를 포함하는 구조로 구성되어있습니다.  
L3 Agent는 VM의 외부 네트워크 액세스를 위한 L3/NAT 전달을 제공합니다.  
L3 Agent에서 VPNaas 플러그인을 설정하는 방법으로 Site to Site IPSec VPN을 사용할 수 있습니다. .

## 주요 용어 설명

### IPsec (Internet Protocol Security)

End Point 또는 보안/터널 게이트웨이(라우터, 방화벽, VPN 등) 구간에서 IP패킷을 인증하고 암호화하는 프로토콜의 집합입니다.

### IKE (Internet Key Exchange)

IPsec 프로토콜 집합의 일부인 키 교환 프로토콜입니다. IKE는 보안 연결을 설정 하는 동안 사용 되며, 사용자 개입 없이 비밀 키와 기타 보호 관련 매개 변수의 안전한 교환을 수행 합니다.

### IKE Policy

IKE 정책을 생성하여 프로세스에서 사용할 peer 인증 알고리즘, 암호화 알고리즘 등과 같은 보안 매개변수를 정의할 수 있습니다.

---

## 설치 및 사용

-   /etc/neutron/neutron.conf 에서 플러그인 설정

```
[DEFAULT]
service_plugins = vpnaas
```

-   /etc/neutron/neutron\_vpnaas.conf 에서 서비스 provider 설정
    -   서비스 드라이버는 리눅스 배포판에 따라 다른 종류를 사용할 수 있습니다.

```
[service_providers]
service_provider = VPN:openswan:neutron_vpnaas.services.vpn.service_drivers.ipsec.IPsecVPNDriver:default
```

-   L3 agent에 vpnaas 플러그인 구성
    -   StrongSwanDriver: ubuntu 배포에 사용되는 드라이버
-   etc/neutron/l3\_agent.ini

```
    [agent]
    extensions = vpnaas,port_forwarding
    [AGENT]
    extensions = vpnaas
    [vpnagent]
    vpn_device_driver = neutron_vpnaas.services.vpn.device_drivers.strongswan_ipsec.StrongSwanDriver
```

### 준비사항

-   neutron client 설치

```
apt install python3-neutron
```

### 테스트 순서

IKE 정책, IPsec 정책, VPN 서비스, 로컬 엔드포인트 그룹(subnet) 및 peer 엔드포인트 그룹(CIDR)을 만듭니다.

위의 정책 및 서비스를 적용하는 IPsec site connection을 만듭니다.

### IKE Policy 생성

```
$ openstack vpn ike policy create ikepolicy
  +-------------------------------+----------------------------------------+
  | Field                         | Value                                  |
  +-------------------------------+----------------------------------------+
  | Authentication Algorithm      | sha1                                   |
  | Description                   |                                        |
  | Encryption Algorithm          | aes-128                                |
  | ID                            | 735f4691-3670-43b2-b389-f4d81a60ed56   |
  | IKE Version                   | v1                                     |
  | Lifetime                      | {u'units': u'seconds', u'value': 3600} |
  | Name                          | ikepolicy                              |
  | Perfect Forward Secrecy (PFS) | group5                                 |
  | Phase1 Negotiation Mode       | main                                   |
  | Project                       | 095247cb2e22455b9850c6efff407584       |
  | project_id                    | 095247cb2e22455b9850c6efff407584       |
  +-------------------------------+----------------------------------------+
```

### IPSec Policy 생성

```
$ openstack vpn ipsec policy create ipsecpolicy
  +-------------------------------+----------------------------------------+
  | Field                         | Value                                  |
  +-------------------------------+----------------------------------------+
  | Authentication Algorithm      | sha1                                   |
  | Description                   |                                        |
  | Encapsulation Mode            | tunnel                                 |
  | Encryption Algorithm          | aes-128                                |
  | ID                            | 4f3f46fc-f2dc-4811-a642-9601ebae310f   |
  | Lifetime                      | {u'units': u'seconds', u'value': 3600} |
  | Name                          | ipsecpolicy                            |
  | Perfect Forward Secrecy (PFS) | group5                                 |
  | Project                       | 095247cb2e22455b9850c6efff407584       |
  | Transform Protocol            | esp                                    |
  | project_id                    | 095247cb2e22455b9850c6efff407584       |
  +-------------------------------+----------------------------------------+
```

### VPN Service 생성

```
$ openstack vpn service create vpn \
  --router 9ff3f20c-314f-4dac-9392-defdbbb36a66
  +----------------+--------------------------------------+
  | Field          | Value                                |
  +----------------+--------------------------------------+
  | Description    |                                      |
  | Flavor         | None                                 |
  | ID             | 9f499f9f-f672-4ceb-be3c-d5ff3858c680 |
  | Name           | vpn                                  |
  | Project        | 095247cb2e22455b9850c6efff407584     |
  | Router         | 9ff3f20c-314f-4dac-9392-defdbbb36a66 |
  | State          | True                                 |
  | Status         | PENDING_CREATE                       |
  | Subnet         | None                                 |
  | external_v4_ip | 192.168.20.7                         |
  | external_v6_ip | 2001:db8::7                          |
  | project_id     | 095247cb2e22455b9850c6efff407584     |
  +----------------+--------------------------------------+
```

### Endpoint group 생성

-   local endpoint group

```
$ openstack vpn endpoint group create ep_subnet \
  --type subnet \
  --value 1f888dd0-2066-42a1-83d7-56518895e47d
  +-------------+-------------------------------------------+
  | Field       | Value                                     |
  +-------------+-------------------------------------------+
  | Description |                                           |
  | Endpoints   | [u'1f888dd0-2066-42a1-83d7-56518895e47d'] |
  | ID          | 667296d0-67ca-4d0f-b676-7650cf96e7b1      |
  | Name        | ep_subnet                                 |
  | Project     | 095247cb2e22455b9850c6efff407584          |
  | Type        | subnet                                    |
  | project_id  | 095247cb2e22455b9850c6efff407584          |
  +-------------+-------------------------------------------+
```

-   peer endpoint group

```
$ openstack vpn endpoint group create ep_cidr \
  --type cidr \
  --value 192.168.1.0/24
  +-------------+--------------------------------------+
  | Field       | Value                                |
  +-------------+--------------------------------------+
  | Description |                                      |
  | Endpoints   | [u'192.168.1.0/24']                  |
  | ID          | 5c3d7f2a-4a2a-446b-9fcf-9a2557cfc641 |
  | Name        | ep_cidr                              |
  | Project     | 095247cb2e22455b9850c6efff407584     |
  | Type        | cidr                                 |
  | project_id  | 095247cb2e22455b9850c6efff407584     |
  +-------------+--------------------------------------+
```

### ipsec site connection 생성

```
$ openstack vpn ipsec site connection create conn \
  --vpnservice vpn \
  --ikepolicy ikepolicy \
  --ipsecpolicy ipsecpolicy \
  --peer-address 192.168.20.9 \
  --peer-id 192.168.20.9 \
  --psk secret \
  --local-endpoint-group ep_subnet \
  --peer-endpoint-group ep_cidr
  +--------------------------+--------------------------------------------------------+
  | Field                    | Value                                                  |
  +--------------------------+--------------------------------------------------------+
  | Authentication Algorithm | psk                                                    |
  | Description              |                                                        |
  | ID                       | 07e400b7-9de3-4ea3-a9d0-90a185e5b00d                   |
  | IKE Policy               | 735f4691-3670-43b2-b389-f4d81a60ed56                   |
  | IPSec Policy             | 4f3f46fc-f2dc-4811-a642-9601ebae310f                   |
  | Initiator                | bi-directional                                         |
  | Local Endpoint Group ID  | 667296d0-67ca-4d0f-b676-7650cf96e7b1                   |
  | Local ID                 |                                                        |
  | MTU                      | 1500                                                   |
  | Name                     | conn                                                   |
  | Peer Address             | 192.168.20.9                                           |
  | Peer CIDRs               |                                                        |
  | Peer Endpoint Group ID   | 5c3d7f2a-4a2a-446b-9fcf-9a2557cfc641                   |
  | Peer ID                  | 192.168.20.9                                           |
  | Pre-shared Key           | secret                                                 |
  | Project                  | 095247cb2e22455b9850c6efff407584                       |
  | Route Mode               | static                                                 |
  | State                    | True                                                   |
  | Status                   | PENDING_CREATE                                         |
  | VPN Service              | 9f499f9f-f672-4ceb-be3c-d5ff3858c680                   |
  | dpd                      | {u'action': u'hold', u'interval': 30, u'timeout': 120} |
  | project_id               | 095247cb2e22455b9850c6efff407584                       |
  +--------------------------+--------------------------------------------------------+
```

참고자료

[https://docs.openstack.org/neutron/latest/admin/vpnaas-scenario.html#enabling-vpnaas](https://docs.openstack.org/neutron/latest/admin/vpnaas-scenario.html#enabling-vpnaas)
