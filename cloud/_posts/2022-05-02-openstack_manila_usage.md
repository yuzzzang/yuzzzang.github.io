---
layout: post
title:  Openstack Manila 사용법
categories: cloud
---

## 1\. Manila란?
​
[\[Openstack\] Manila 란?](2022-05-03-openstack_manila.md) 참고
​
## 2\. 배포(kolla)
​
현재 kolla로 배포할 수 있는 manila 서비스는 다음과 같다
​
-   manila-api
-   manila-schduler
-   manila-share
​
### 2.1. 설정
​
cinder, ceph, manila, generic back end 사용가능하도록 설정
​
**`/etc/kolla/globals.yml`**
​
```
enable_cinder: "yes"
enable_ceph: "yes"
enable_manila: "yes"
enable_manila_backend_generic: "yes"
```
​
참고: [https://docs.openstack.org/kolla-ansible/queens/reference/manila-guide.html](https://docs.openstack.org/kolla-ansible/queens/reference/manila-guide.html)
​
### 2.2. 이미지 빌드
​
`/etc/kolla/kolla-build.conf` 수정 후 빌드 수행
​
[kolla image 빌드(source)](https://yujeong-dev.tistory.com/31) 참고
​
## 3\. 사용법(cli)
​
### 3.1. 준비
​
-   manila client 설치
​
`apt install python3-manilaclient`
​
### 3.2 manila 서비스 확인
​
```
# manila service-list
+------------------+----------------+------+---------+-------+----------------------------+-----------------+
|      Binary      |    Host        | Zone |  Status | State |         Updated_at         | Disabled Reason |
+------------------+----------------+------+---------+-------+----------------------------+-----------------+
| manila-scheduler | controller     | nova | enabled |   up  | 2014-10-18T01:30:54.000000 |       None      |
| manila-share     | share1@generic | nova | enabled |   up  | 2014-10-18T01:30:57.000000 |       None      |
+------------------+----------------+------+---------+-------+----------------------------+-----------------+
```
​
### 3.3. share 생성 순서
​
**default share type 생성**
​
```
# manila type-create default_share_type True
+--------------------------------------+--------------------+------------+------------+-------------------------------------+-------------------------+
| ID                                   | Name               | Visibility | is_default | required_extra_specs                | optional_extra_specs    |
+--------------------------------------+--------------------+------------+------------+-------------------------------------+-------------------------+
| 8a35da28-0f74-490d-afff-23664ecd4f01 | default_share_type | public     | -          | driver_handles_share_servers : True | snapshot_support : True |
+--------------------------------------+--------------------+------------+------------+-------------------------------------+-------------------------+
```
​
**manila에서 제공하는 share server 이미지 다운로드**
​
```
# wget http://tarballs.openstack.org/manila-image-elements/images/manila-service-image-master.qcow2
# glance image-create --name "manila-service-image" \
  --file manila-service-image-master.qcow2 \
  --disk-format qcow2 --container-format bare \
  --visibility public --progress
[=============================>] 100%
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | 48a08e746cf0986e2bc32040a9183445     |
| container_format | bare                                 |
| created_at       | 2016-01-26T19:52:24Z                 |
| disk_format      | qcow2                                |
| id               | 1fc7f29e-8fe6-44ef-9c3c-15217e83997c |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | manila-service-image                 |
| owner            | e2c965830ecc4162a002bf16ddc91ab7     |
| protected        | False                                |
| size             | 306577408                            |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2016-01-26T19:52:28Z                 |
| virtual_size     | None                                 |
| visibility       | public                               |
+------------------+--------------------------------------+
```
​
**network확인 및 manila share-network생성**
​
-   manila network가 아닌 subnet이 존재하는 private 네트워크 이용
​
```
# openstack network list
​
+--------------------------------------+---------+----------------------------------------------------+
| id                                   | name    | subnets                                            |
+--------------------------------------+---------+----------------------------------------------------+
| 0e62efcd-8cee-46c7-b163-d8df05c3c5ad | public  | 5cc70da8-4ee7-4565-be53-b9c011fca011 10.3.31.0/24  |
| 7c6f9b37-76b4-463e-98d8-27e5686ed083 | private | 3482f524-8bff-4871-80d4-5774c2730728 172.16.1.0/24 |
+--------------------------------------+---------+----------------------------------------------------+
​
# manila share-network-create --name demo-share-network1 \
--neutron-net-id PRIVATE_NETWORK_ID \
--neutron-subnet-id PRIVATE_NETWORK_SUBNET_ID
+-------------------+--------------------------------------+
| Property          | Value                                |
+-------------------+--------------------------------------+
| name              | demo-share-network1                  |
| segmentation_id   | None                                 |
| created_at        | 2016-01-26T20:03:41.877838           |
| neutron_subnet_id | 3482f524-8bff-4871-80d4-5774c2730728 |
| updated_at        | None                                 |
| network_type      | None                                 |
| neutron_net_id    | 7c6f9b37-76b4-463e-98d8-27e5686ed083 |
| ip_version        | None                                 |
| nova_net_id       | None                                 |
| cidr              | None                                 |
| project_id        | e2c965830ecc4162a002bf16ddc91ab7     |
| id                | 58b2f0e6-5509-4830-af9c-97f525a31b14 |
| description       | None                                 |
+-------------------+--------------------------------------+
```
​
**flavor생성**
​
-   _kolla 이미지 빌드 시_ `*/etc/kolla/config/manila-share.conf` 파일의 `manila_instance_flavor_id` 값을 설정하지 않은 경우만\*
​
```
# nova flavor-create manila-service-flavor 100 128 0 1
```
​
**share server 생성**
​
-   NFS생성
​
```
# manila create NFS 1 --name demo-share1 --share-network demo-share-network1
+-----------------------------+--------------------------------------+
| Property                    | Value                                |
+-----------------------------+--------------------------------------+
| status                      | None                                 |
| share_type_name             | None                                 |
| description                 | None                                 |
| availability_zone           | None                                 |
| share_network_id            | None                                 |
| export_locations            | []                                   |
| host                        | None                                 |
| snapshot_id                 | None                                 |
| is_public                   | False                                |
| task_state                  | None                                 |
| snapshot_support            | True                                 |
| id                          | 016ca18f-bdd5-48e1-88c0-782e4c1aa28c |
| size                        | 1                                    |
| name                        | demo-share1                          |
| share_type                  | None                                 |
| created_at                  | 2016-01-26T20:08:50.502877           |
| export_location             | None                                 |
| share_proto                 | NFS                                  |
| consistency_group_id        | None                                 |
| source_cgsnapshot_member_id | None                                 |
| project_id                  | 48e8c35b2ac6495d86d4be61658975e7     |
| metadata                    | {}                                   |
+-----------------------------+--------------------------------------+
```
​
**접근 가능한 네트워크 설정**
​
```
# manila access-allow demo-share1 ip INSTANCE_PRIVATE_NETWORK_IP
```
​
**마운트 설정**
​
-   마운트할 폴더 생성
​
```
# mkdir ~/test_folder
```
​
-   share 정보 확인
​
```
# manila show demo-share1
+-----------------------------+----------------------------------------------------------------------+
| Property                    | Value                                                                |
+-----------------------------+----------------------------------------------------------------------+
| status                      | available                                                            |
| share_type_name             | default_share_type                                                   |
| description                 | None                                                                 |
| availability_zone           | nova                                                                 |
| share_network_id            | fa07a8c3-598d-47b5-8ae2-120248ec837f                                 |
| export_locations            |                                                                      |
|                             | path = 10.254.0.3:/shares/share-422dc546-8f37-472b-ac3c-d23fe410d1b6 |
|                             | preferred = False                                                    |
|                             | is_admin_only = False                                                |
|                             | id = 5894734d-8d9a-49e4-b53e-7154c9ce0882                            |
|                             | share_instance_id = 422dc546-8f37-472b-ac3c-d23fe410d1b6             |
| share_server_id             | 4782feef-61c8-4ffb-8d95-69fbcc380a52                                 |
| host                        | share1@generic#GENERIC                                               |
| access_rules_status         | active                                                               |
| snapshot_id                 | None                                                                 |
| is_public                   | False                                                                |
| task_state                  | None                                                                 |
| snapshot_support            | True                                                                 |
| id                          | e1e06b14-ba17-48d4-9e0b-ca4d59823166                                 |
| size                        | 1                                                                    |
| name                        | demo-share1                                                          |
| share_type                  | 6e1e803f-1c37-4660-a65a-c1f2b54b6e17                                 |
| has_replicas                | False                                                                |
| replication_type            | None                                                                 |
| created_at                  | 2016-03-15T18:59:12.000000                                           |
| share_proto                 | NFS                                                                  |
| consistency_group_id        | None                                                                 |
| source_cgsnapshot_member_id | None                                                                 |
| project_id                  | 9dc02df0f2494286ba0252b3c81c01d0                                     |
| metadata                    | {}                                                                   |
+-----------------------------+----------------------------------------------------------------------+
```
​
위의 명령어를 통해 얻은 export location path를 사용해 만들어둔 디렉토리에 마운트한다
​
```
# mount -v 10.254.0.3:/shares/share-422dc546-8f37-472b-ac3c-d23fe410d1b6 ~/test_folder
```
​
참고
​
[https://docs.openstack.org/kolla-ansible/queens/reference/manila-guide.html](https://docs.openstack.org/kolla-ansible/queens/reference/manila-guide.html)
