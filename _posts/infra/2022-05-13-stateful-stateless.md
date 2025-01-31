---
title: "[Infra] Stateful, Stateless 차이"
excerpt: "스테이트풀과 스테이트리스의 차이 알기"
categories: 
  - Infra
last_modified_at: 2022-05-13T15:24:00+09:00
tags: 
    - Infra
    - AWS
author_profile: true
read_time: true
toc_label: "State" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## Stateful / Stateless

* Stateful은 Client에서 요청한 정보가 세션이 종료되기 전까지 Server에 저장
* Stateless는 Client에서 요청한 정보가 Server에 저장되지 않음

---

<br>
## AWS SG, NACL로 비교

---

### AWS SG

* AWS Security Group은 인바운드되는 네트워크 정보를 저장하고 있기 떄문에 아웃바운드 정책을 적용하지 않아도 접근 제어가 가능한 Stateful 방식
* 들어올 때 허용되었으니, 나갈때도 허용한다는 의미

### AWS NACL

* AWS NACL은 인바운드되는 네트워크 정보를 저장하지 않기 때문에 아웃바운드 정책을 적용해야 네트워크 접근 제어가 가능한 Stateless 방식


### Stateful

* Client에서 요청한 정보가 세션 종료까지 서버에 저장된다
* Server에 저장되기 때문에 Stateless 보다 CPU, Memory 사용량이 높음
* FTP, SSH, Telnet

### Stateless

* Client에서 요청한 정보가 서버에 저장되지 않는다.
* Server에 저장되지 않기 때문에 Stateful보다 CPU, Memory 사용량이 낮음
* UDP, DNS, HTTP
