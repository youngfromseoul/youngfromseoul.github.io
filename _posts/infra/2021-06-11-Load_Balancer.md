---
title: "[Infra] Load Balancer란?"
excerpt: "Load Balancer Type 및 동작 방식"
categories: 
  - Infra
last_modified_at: 2021-06-11T09:02:00+09:00
tags: 
    - Infra
    - Devops
author_profile: true
read_time: true
toc_label: "Load-Balancer" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## Load Balancer?

여러 대의 서버를 통해, 부하를 분산처리하고자 할 경우, 로드 밸런서를 통해 각 서버로 트래픽을 분산해줄 수 있는 서비스

<br>

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/load-balancing.png?raw=true) 

<br>

* **Round Robin** <br>
라운드 로빈 방식은 트래픽을 모든 서버에 연속적으로 전송하여 요청을 순차적으로 전달 <br>
트래픽을 균등하게 분배하지만 각 서버의 부하량 고려할 수 없음

* **Least Connections** <br>
서버와 소프트웨어간에 활성 연결이 가장 적은 서버로 트래픽을 전달 <br>
이 방법은 상대적으로 효율적이지만 노드의 응답 성을 고려하지 않음

<br>

## Load Balancer Type
* **DSR(Direct Server Return)**

> 서버로 요청 온 패킷을 Load Balancer를 거치지않고 클라이언트에게 전달 <br>
> Load Balancer를 거치지 않기때문에 Load Balancer의 부하를 감소 <br>
> Direct로 전달하기 때문에, 서버에서 Client IP로 접근 됨 <br>
> Session 기반 서버에서는 사용이 어려움 <br>


* **Proxy**

> DSR과 반대로 Load Balancer를 거치기 때문에, 서버 단에서 Load Balancer IP로 접근됨 <br>
> Client IP 식별을 위해 Load Balancer에서 X-FORWARD-FOR에 Client IP를 찍어서 서버로 전달 <br>
> Session을 관리할 중간관리 역할도 수행하므로, Session기반 서버에서 사용 <br>
