---
title: "[KVM] qcow2 zero area 삭제하기"
excerpt: "KVM qcow2 zero area 삭제하기"
categories: 
  - Linux
last_modified_at: 2020-09-06T10:51:00+09:00
tags: 
    - Linux
    - CentOS
    - KVM
    - VM
    - Virtualization
    - Hypervisor
author_profile: true
read_time: true
toc_label: "KVM" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## KVM Zero Area란?
qcow2 이미지는 동적이미지로 할당된 사이즈에서 실제 사용한 만큼만 증가하게 된다. <br>
일정수치만큼 증가 후 실제 데이터를 삭제하게 되었을 경우 삭제한 데이터를 감지하지 못하게 된다. <br>
이럴경우 비어있는(zero area or zero-fill) 데이터를 삭제하여 이미지 크기를 실제 사용하는 데이터로 변환해주는 방법이다.
<br>

## Zero Area 삭제

zero area를 삭제하는 명령은 다음과 같다.
```
qemu-img convert -c -O qcow2 source.qcow2 target.qcow2
```
해당 명령 사용 시, 원본의 데이터는 그대로 있고 새로운 파일이 생성되게 된다.
용량이 없을 경우, 다른 경로로 생성 후, VM의 경로를 변경해주거나, 원본을 삭제 후 이동해야 한다.
