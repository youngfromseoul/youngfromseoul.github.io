---
title: "클라우드에서의 SWAP Memory 사용에 대해"
excerpt: "클라우드에서의 SWAP Memory 사용에 대해"
categories: 
  - Infra
last_modified_at: 2022-05-13T12:09:00+09:00
tags: 
    - Infra
    - Devops
author_profile: true
read_time: true
toc_label: "Swap Memory" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## 클라우드 환경에서의 SWAP 메모리 사용에 대해
일반적으로 온프레미스 환경에서는 Linux Server 구성 시, SWAP Memory를 구성한다. <br>
> 다만 최근에는 메모리의 크기가 커졌기 때문에 점차 사용하지 않는 추세로 바뀌고 있다. <br>
> SWAP 사용 자체가 퍼포먼스 저하가 발생되기 때문에 다른 방법으로 해소를 하게 되는데, <br>
> 대표적인것으로 캐싱을 사용 하는 방식이다. <br>
<br>

다만, 클라우드 환경에서는 SWAP을 잘 사용하지 않는다. <br>
온프레미스의 Disk IOPS의 경우, 약 800 ~ 100000 정도이나, 클라우드의 IOPS는 일반적으로 80 ~ 4000 선이기 때문에 느리며, <br>
높은 IOPS의 Disk를 프로비저닝 하는것 보다, 메모리 사양을 늘리는 것이 비용이나 성능적으로 이점이기 때문이다. <br>
<br>
또한, 클라우드 환경에서 메모리 사용률이 높아지면, 오토 스케일링, 로드 밸런서를 통한 스케일 아웃으로 분산 구성이 쉬운 부분도 있다.
<br>
하지만 일부 환경에서는 SWAP 사용이 필요한 경우도 있다.
* OS 특정 데이터의 일시 보관을 위해 SWAP을 사용하는 경우
* 레거시한 어플리케이션 구동 시, SWAP을 쓰도록 구성된 경우
* 일부 어플리케이션의 특정 기능 사용 시

<br>

그럼에도, 최소한의 보험으로 SWAP Memory를 설정하여 운영을 하는 경우도 있음 <br>
SWAP Memory 사용 시 File SWAP을 통해 여러 디스크에 분산하여 엮어서 구성 시 로드 밸런싱하여 사용도 가능
