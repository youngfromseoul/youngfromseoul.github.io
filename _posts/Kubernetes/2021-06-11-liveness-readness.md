---
title: "[Kubernetes] liveness, readness, startup probe 이해"
excerpt: "쿠버네티스 핵심 개념 liveness, readness, startup probe"
categories: 
  - Kubernetes
last_modified_at: 2021-06-11T08:10:00+09:00
tags: 
    - Kubernetes
    - Infra
    - DevOps
author_profile: true
read_time: true
toc_label: "probe" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---
## Probe types

<span style="color:#000">Liveness Probe와 Readiness Probe는 컨테이너가 정상유뮤를 확인하는 방법으로 3가지 방식을 제공한다.</span>
<br>
* Command probe
    * 특정 명령의 결과를 통해 정상 유뮤를 체크 ( 값이 0이면 성공, 0이 아니면 실패 )

<br>
* HTTP probe
    * 특정 URI에 HTTP GET 요청을 통해 체크( 응답코드가 200\~300일 경우, 정상 그 외 오류코드는 비 정상으로 판단 )

<br>
* TCP probe
    * 특정 TCP 포트에 연결을 시도하여 체크 ( 연결 성공하면 정상으로 판단 )

<br>
<br>
## Liveness Probe

> 컨테이너 살았는지 판단하고 다시 시작하는 기능 <br>
> 컨테이너의 상태를 스스로 판단하여 교착 상태에 빠진 컨테이너를 재시작 <br>
> 버그가 생겨도 높은 가용성을 보임 <br>

<br>
<br>
## Readness Probe

> 포드가 준비된 상태에 있는지 확인하고 정상 서비스를 시작하는 기능 <br>
> 포드가 적절하게 준비되지 않은 경우 로드밸런싱을 하지 않음 <br>

<br>
<span style="color:#000">Liveness probe와 차이점은 Liveness probe는 컨테이너 비 정상으로 판단 시, 해당 Pod를 재시작하지만,</span><br>
<span style="color:#000"> Readiness probe는 컨테이너가 비 정상일 경우, 해당 Pod을 서비스에서 제외</span>

<br>
<br>
## Startup Probe

> 애플리케이션의 시작 시기 확인하여 가용성을 높이는 기능 <br>
> Liveness와 Readiness의 기능을 비활성화 <br>

<br>
컨테이너 내의 애플리케이션이 시작되었는지를 나타낸다. <br>
스타트업 프로브(startup probe)가 주어진 경우, 성공할 때까지 다른 나머지 프로브는 활성화되지 않는다. <br>
만약 스타트업 프로브가 실패하면, kubelet이 컨테이너를 죽이고, 컨테이너는 재시작 정책에 따라 처리된다. <br>
컨테이너에 스타트업 프로브가 없는 경우, 기본 상태는 `Success`이다.
