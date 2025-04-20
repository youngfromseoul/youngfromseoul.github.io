---
title: "CentOS, RHEL Network Bonding 구성하기"
excerpt: "RHEL 기반 Linux에서 Network Bonding 구성 방법"
categories: 
  - Linux
last_modified_at: 2019-01-28T21:39:00+09:00
tags: 
    - Linux
    - CentOS
    - Redhat
    - Bonding
author_profile: true
read_time: true
toc_label: "Network Bonding 구성하기" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---


## 본딩(Bonding)이란?  
한개 이상의 네트워크 포트 혹은 네트워크 카드를 묶어 하나의 디바이스로 사용하는 것으로, 
이중화의 개념으로도 사용하며, 여러개의 디바이스를 묶어서 높은 대역폭으로 사용하기도 한다.

---



## Bonding 구성
 
* 본딩 구성 할 디바이스를 vi 편집기로 열어 편집합니다.

```
 [root@Web-Server /]# vi /etc/sysconfig/network-scripts/ifcfg-eth0

 [root@WEB-Server /]# vi /etc/sysconfig/network-scripts/ifcfg-eth1
```


  아래와 같이 SLAVE Device를 지정해줍니다.
{: .notice--info}
```
  DEVICE=eth1 ( OR eth0 )

  USERCTL=no

  ONBOOT=yes

  MASTER=bond0

  SLAVE=yes

  BOOTPROTO=none

  NM_CONTROLLED=no 
```
 

* vi 편집기로 Bonding Device를 생성합니다.

```
[root@Web-Server /]# vi /etc/sysconfig/network-scripts/ifcfg-bond0
```

  



  고정IP로 설정할때
{: .notice--info}

```
  DEVICE=bond0

  TYPE=bond

  NAME=bond0

  IPADDR=            

  NETMASK=       

  GATEWAY=      

  DNS1=       

  DNS2=     

  USERCTL=no

  BOOTPROTO=none

  ONBOOT=yes

  NM_CONTROLLED=no 

  BONDING_OPTS="mode=5 miimon=100"

  BONDING_MASTER=yes
```

 

   DHCP 사용할때
  {: .notice--info}
  
```
  DEVICE=bond0

  TYPE=bond

  NAME=bond0

  DNS1=      

  DNS2=    

  USERCTL=no

  BOOTPROTO=dhcp

  ONBOOT=yes

  NM_CONTROLLED=no 

  BONDING_OPTS="mode=5 miimon=100"

  BONDING_MASTER=yes
```


## mode 참고
- mode0 = balance-rr : (Round Robin) Load Balancing, 송신할 패킷마다 사용하는 NIC을 바꾼다.
- mode1 = active-backup: Failover, bond내에서 한개의 Slave만 사용 포트문제가 생길경우 다른 Slave가 Enable
- mode2 = balance-xor : Load Balancing, 소스와 목적지의 MAC을 XOR 연산을 통해 사용할 NIC를 결정하여 분배
- mode3 = Broadcast : Fault-Tolerance, 모든 Slave으로 데이터전송(failover), 일반적으로는 잘 사용안함.
- mode4 = 802.3ad : Dynamic Link Aggregation, IEEE 802.3ad 프로토콜을 이용하여 동적 Aggregation 작성 대역폭 상승, 부하 분산, Failover 지원
- mode5 = balance-tlb(TLB) : 적응형 송신 부하 분산, 송신패킷 로드밸런싱, 송신시 부하가 낮은 NIC이용 수신은 특정 NIC이용
- mode6 = balance-alb(ALB) : 적응형 부하 분산, 송수신패킷 로드밸런싱, 송수신시 부하가 낮은 NIC를 사용 
