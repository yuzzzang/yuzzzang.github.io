---
layout: post
title:  Openstack Manila 
categories: cloud
---


## 1\. Manila란?
​
manila는 인스턴스가 사용할 수 있는 공유 파일 시스템 서비스를 제공합니다. manila를 통해 공유 파일 시스템을 만들고 가시성, 접근성 및 사용 할당량과 같은 속성을 관리할 수 있습니다.
​
### 1.1. 서비스 구성

![manila](/assets/img/manila_process_layout.png)
​
![manila2](/assets/img/manila.png)

manila는 다음과 같은 네 가지의 서비스로 구성됩니다.
​
-   **manila-api**
    -   공유 파일 시스템 서비스 전체에서 요청을 인증하고 라우팅합니다. OpenStack API를 지원합니다.
-   **manila-data**
    -   요청 수신, 복사, share 마이그레이션, 백업과 같이 실행 시간이 오래 걸리는 데이터 작업을 처리하는 독립 실행형 서비스입니다.
-   **manila-scheduler**
    -   적절한 share 서비스로 요청을 스케줄링 및 라우팅합니다.
-   **manila-share**
    -   공유 파일 시스템을 제공하는 back-end 장치를 관리합니다. manila-share 서비스는 share server 유무에 따라 두 가지 모드로 실행할 수 있습니다.
        -   share server는 share network를 통해 파일 공유를 내보냅니다. share server를 사용하지 않는 경우 네트워크 요구 사항은 manila 외부에서 처리됩니다.
​
### 1.2. Key Concepts
​
-   **share**
    -   스토리지의 기본 단위로, 프로토콜, 크기 및 액세스 목록을 가지고 있습니다.
    -   모든 공유는 backend에 존재하며 일부 share는 share network 및 share server와 연결되어 있습니다.
    -   NFS , CIFS, GlusterFS, HDFS, CephFS 등의 프로토콜이 지원됩니다.
        
        | 프로토콜 | 설명 |
        | --- | --- |
        | NFS | Network File System (NFS) |
        | CIFS | Common Internet File System (CIFS) |
        | GLUSTERFS | Gluster file system (GlusterFS) |
        | HDFS | Hadoop Distributed File System (HDFS) |
        | CEPHFS | Ceph File System (CephFS) |
        | MAPRFS | MapR File System (MAPRFS) |
        
​
-   **backend**
    -   공유 파일 시스템 공급자입니다. backend는 드라이버를 통해 스토리지 시스템과 통신합니다.
    -   generic backend
        -   블록 스토리지 서비스(Cinder), Nova VM,Neutron Network를 사용하여 manila를 제공합니다.
-   **snapshot**
    -   스냅샷은 share의 특정 시점을 복사본으로 만든 것입니다.
    -   스냅샷은 새 share를 생성하는 데만 사용할 수 있습니다. 또한 관련된 모든 스냅샷이 삭제될 때까지 share를 삭제할 수 없습니다.
-   **share type**
    -   share type은 share를 특정짓기 위한 추상적인 기준입니다.
        -   예) 성능을 기준으로 share type을 명시 : premium share, basic share..
-   **share network**
    -   인스턴스가 share에 접근하는 데 사용하는 Neutron 네트워크과 subnet을 정의합니다.
    -   share는 하나의 share network와 연결될 수 있습니다.
​
### 참고문서
​
[https://docs.openstack.org/manila/queens/admin/shared-file-systems-key-concepts.html](https://docs.openstack.org/manila/queens/admin/shared-file-systems-key-concepts.html "key concepts")
​
[https://docs.openstack.org/manila/queens/configuration/shared-file-systems/overview.html](https://docs.openstack.org/manila/queens/configuration/shared-file-systems/overview.html "manila 개요")
