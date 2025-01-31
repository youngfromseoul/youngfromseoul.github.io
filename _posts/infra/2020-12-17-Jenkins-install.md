---
title: "[Jenkins] Jenkins 설치하기"
excerpt: "[Jenkins] Jenkins 설치하기"
categories: 
  - DevOps
last_modified_at: 2020-12-15T15:47:00+09:00
tags: 
    - Jenkins
    - Infra
    - Devops
author_profile: true
read_time: true
toc_label: "Jenkins Install" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins1.png?raw=true)

## 1\. Jenkins?

`Jenkins`<span style="color:#000000">는 소프트웨어 개발 시 지속적으로 통합 서비스를 제공하는 CI </span>`(Continuous Integration`<span style="color:#000000"> tool 로,</span><span style="color:#000000"></span>
<span style="color:#000000">소스 형상관리 툴 (bitbuket, git 등)과 연계하여 단위 테스트를 자동으로 수행하고 프로덕션이나 테스트 환경에 배포를 진행해주는 tool 입니다.</span>

<br>
> * 프로젝트 표준 컴파일 환경에서의 컴파일 오류 검출
> * 자동화 테스트 수행
> * 정적 코드 분석에 의한 코딩 규약 준수여부 체크
> * 프로파일링 툴을 이용한 소스 변경에 따른 성능 변화 감시
> * 결합 테스트 환경에 대한 배포작업

<br>
## 2\. Install Jenkins

* jenkins repository 다운로드

```
$ wget -O /etc/yum.repos.d/jenkins.repo [https://pkg.jenkins.io/redhat-stable/jenkins.repo](https://pkg.jenkins.io/redhat-stable/jenkins.repo)
$ rpm --import [https://pkg.jenkins.io/redhat-stable/jenkins.io.key](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)
```
<br>
* Jenkins 설치

```
$ yum install jenkins
```
<br>
* 기본 포트가 8080 포트이므로, 중복되는 경우 포트 변경
* 변경한 포트에 대해서 방화벽 Open 필요

```
$ vim /etc/sysconfig/jenkins
```

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins2.png?raw=true)

<br>
* 서비스 실행

```
$ systemctl enable jenkins 
$ systemctl start jenkins
```
<br>
* 실행 오류 발생 시, JAVA 설치

```
$ yum install java
```
<br>
* 포트 실행 확인

```
$ netstat -nltp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:20165           0.0.0.0:*               LISTEN      4899/sshd
tcp6       0      0 :::20165                :::*                    LISTEN      4899/sshd
tcp6       0      0 :::8080                 :::*                    LISTEN      9323/java
```
<br>
* [http://xxx.xxx.xxx.xxx:8080](http://xxx.xxx.xxx.xxx:8080) 웹 접속

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins3.png?raw=true)

<br>
* 암호 확인 후 붙여넣기

```
cat /var/lib/jenkins/secrets/initialAdminPassword
```
<br>
* 원하는 Plugin을 선택하여 설치하고 싶을 경우, 오른쪽 메뉴 선택

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins4.png?raw=true)

<br>
* 설치 중...

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins5.png?raw=true)

<br>
* 설치 완료 후, 어드민 계정정보 생성

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins6.png?raw=true)

<br>
* 설치 완료

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/jenkins7.png?raw=true)

<br>
다른 형상관리 툴과 연동, Ansible 연동 등에 대해서도 추 후 포스팅 예정입니다.

