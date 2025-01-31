---
title: "CentOS8 지원 종료 및 대안"
excerpt: "CentOs8 지원 종료 및 대안"
categories: 
  - Linux
tags: 
    - CentOS
    - Infra
    - Linux
author_profile: true
read_time: true
toc_label: "CentOS8" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---


## CentOS
그동안 CentOS 는 RHEL(Red Hat Enterprise Linux)와 호환이 되는 안정적인 운영체제였습니다. <br>
RHEL 이 정식 발표된 후에 소스를 가져다가 다시 빌드해서 만드므로 100% 바이너리 호환성을 갖고 있으며 제품 지원 기간(보통 10년)이 길고 별도의 비용 지불이 필요없으므로 Web Server, 개발 시스템,내부 용도 서비스에 사용하기에 적합했습니다. <br>

하지만 2020년 12월 9일에 발표된 RedHat 발표에 따르면 CentOS8 은 2020년 12월 31일에 지원이 끝나고 22년부터는 CentOS Stream 이라는 운영체제로 변경됩니다. <br>

**다만, CentOS7은 2024년 6월 30일까지 기존 EOL과 동일한 정책으로 서비스 됩니다.**

<br>

정리하면 CentOS 는 버전별로 다음과 같은 lifecycle 을 갖게 됩니다.
> * CentOS 8 : 2021년 12월 31일까지만 지원되고 종료, 이후는 CentOS Stream 으로 전환
> * CentOS 7 : 2024년 6월 30일까지 지원
> * CentOS 6 : 2020년 11월 30일 종료됨

<br>

**CentOS Stream 과 차이** <br>
RHEL 발표후 소스를 다시 빌드해서 제공하는 CentOS 와는 달리 CentOS Stream 은  CentOS Stream 은 RHEL 의 Upstream(development) 운영체제로 Fedora 에서 추가된 신기능이나 app 등을 선별해서 검증하는 베타 채널의 개념으로 운영될 예정입니다.

<br>

## 대안 
**2021년 12월 31일로 CentOS8이 종료되므로 CentOS8 사용자는 여러 대안중에 고민해야 하며 CentOS7 사용자도 장기적으로 이전 준비를 해야합니다.**

<br>

- **RHEL 로 전환** <br>
Red Hat 이 제시하는 방법중 하나로 비용 문제를 제외하면 전환도 쉽고 좋은 방법이지만 여러 이유로 CentOS 를 사용했던 곳에서 RHEL 로 전환하지는 않을 듯 합니다.

<br>

- **Rocky LinuxLink to Rocky Linux** <br>
Rocky Linux 는 CentOS 프로젝트의 창시자인 Gregory Kurtzer 가 시작한 프로젝트로 현재의 CentOS 처럼 RHEL 의 공개 소스를 가져와서 다시 빌드하고 패키징하는 것을 목표로 하고 있으며 심지어 RHEL 의 버그까지 똑같이 재연하는 것을 목료포 하고 있습니다. <br>
커뮤니티의 전폭적인 지지를 받고 있으며 github 저장소에는 프로젝트의 방향성과 FAQ 를 정리한 문서밖에 없지만 좋아요를 5,000 개 이상 받았으며 아직 구체적인 릴리스 날자가 발표되지는 않았습니다.

<br>

- **Cloud LinuxLink to Cloud Linux** <br>
Cloud Linux 라는 배포판을 만드는 cloudlinux.com 이라는 회사에서 RHEL 8 과 호환되는 CentOS 와 유사한 배포판을 발표할 계획이며 RHEL 8 과 동일하게 29년까지 제품 지원을 제공할 예정입니다.
이 회사는 구체적으로 21년 1분기에 제품을 출시하겠다고 발표했습니다. <br>
(참고 - Announcing Open-sourced & Community-Driven RHEL Fork by CloudLinux)

<br>

- **Oracle LinuxLink to Oracle Linux** <br>
RHEL 기반의 배포판인 Oracle Linux 를 만드는 오라클은 자사 제품으로 전환을 가장 적극적으로 지원하고 있습니다. <br>
오라클 리눅스는 커스텀 커널인 UEK(Oracle Unbreakable Enterprise Kernel) 기반인데 원할 경우 RHEL 에 포함된 커널을 사용할 수도 있으며 자사 제품으로 쉽게 전환하기 위해서 centos to oracle linux 라는 쉘 스크립트까지 제공하고 있습니다. <br>
하지만 Sun Micro systems 를 인수한후에 썬이 진행하던 오픈 소스(Hudson, Open Solaris, MySQL 등)를 대하던 태도나 Java 정책 변경등을 보면 오라클 리눅스로 전환이 과연 CentOS 정책 변경의 대안일까 하는 의문이 듭니다. <br>
특히 이번 정책 변경이 아쉽긴 하지만 Red Hat 은 꾸준히 다양한 Open Source 를 지원하는 회사이고 Linux Kernel 에 가장 많은 기여를 하는 회사중 하나지만 오라클은 별다른 기여가 없는 것으로 알고 있으며 기존 사례로 봤을 때 오라클 리눅스가 어떻게 정책을 변경할지 알수 없으므로 저라면 오라클 리눅스로 전환하기 않을 것입니다.

