---
title: "portainer (docker 웹 관리 콘솔) 사용하기"
excerpt: "portainer (docker 웹 관리 콘솔) 사용하기"
categories: 
  - Linux
last_modified_at: 2020-07-05T23:00:00+09:00
tags: 
    - Linux
    - CentOS
    - Redhat
    - Ubuntu
    - docker
author_profile: true
read_time: true
toc_label: "docker" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

portainer ( docker 웹 관리 콘솔) 기능이 도커 사용 시에 관리나 모니터링 면에서 유용한 툴

<br>
서버환경 : centos 7
<br>
### 1\. docker 설치

```
yum install yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo [https://download.docker.com/linux/centos/docker-ce.repo](https://download.docker.com/linux/centos/docker-ce.repo)
yum install docker-ce docker-ce-cli containerd.io
systemctl enable docker && systemctl start docker
```
<br>
### 2. Portainer 에서 사용할 Volume 을 생성

```
docker volume create portainer_data
```
<br>
<br>
### 3. Portainer 컨테이너 이미지 다운로드 및 실행

```
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --restart=always portainer/portainer
```
<br>
### 4\. 9000번 포트로 웹 접속 및 관리 패스워드 설정

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image1.png?raw=true)
<br>
### 5\. potainer가 설치된 로컬 서버가 아닌\, 리모트 도커 서버도 관리 가능

(현재는 로컬에만 설치되어있기 때문에 로컬 선택 후 connect)
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image2.png?raw=true)
<br>
### 6, 대시보드 통해, 도커 정보 확인 가능

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image3.png?raw=true)

<br>
### 7\. 컨테이너 항목 통해 현재 실행 중인 컨테이너 확인 및 관리 가능

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image4.png?raw=true)

**-quick actions**
<br>
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image5.png?raw=true)

* 왼쪽부터 컨테이너 로그 - 컨테이니 정보 - 리소스 통계 - bash 콘솔

<br>
1\. 컨테이너 로그
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image6.png?raw=true)

2\. 컨테이너 정보
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image7.png?raw=true)

3.  리소스 통계
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image8.png?raw=true)

4\. 콘솔
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image9.png?raw=true)

<br>
### 8\. 이벤트 메뉴 통해 도커 로그 확인 가능

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image10.png?raw=true)
 
### 9\. registries 메뉴에서 로컬 레지스트리 추가 가능

* <span style="color:#e11d21">웹 상으로 레지스트리 내 이미지들을 관리하려면 익스텐션 (유료 라이선스) 구매 필요 ( 95$ / 연)</span>

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image11.png?raw=true)

<br>
**\- 로컬 레지스트리 생성하기**

1\. Registry 이미지 다운로드 & 실행

```
docker pull registry
docker run -d -p 5000:5000 -v /home/registry:/var/lib/registry/docker/registry/v2 --restart=always --name godo_registry registry
```
<br>
2\. 레지스트리 포트 지정

```
vim /etc/docker/daemon.json

{
    "insecure-registries": ["server iP:5000"]
}
```
<br>
3\. 이미지 업로드 & 배포
<br>
```
# 이미지 태깅
docker image tag mysql 192.168.0.120:5000/mysql_5.6

# 이미지 업로드
docker push 192.168.0.120:5000/mysql_5.6

# 이미지 배포
docker pull 192.168.0.120:5000/mysql_5.6
```
<br>
### 10, image 메뉴 통해 새로운 이미지 빌드, import, export, 로컬 레지스트리에서 이미지 가져오기 기능
<br>
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image12.png?raw=true)
<br>
### 11\. endpoints 메뉴에서  리모트 도커 서버 추가

```
#docker.service 복사
cp /lib/systemd/system/docker.service /etc/systemd/system/docker.service
```
<br>
```
#복사한 docker.service를 편집
vim /etc/systemd/system/docker.service
```
<br>
```
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (기존 설정 주석처리)
ExecStart=/usr/bin/dockerd -H unix:// -H tcp://0.0.0.0:4000 (임의로 4000포트로 설정함)
```
<br>
```
# Docker 재시작
systemctl daemon-reload
systemctl restart docker
```
<br>
* 엔드포인트 생성

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image13.png?raw=true)
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image14.png?raw=true)
<br>
* 리모트 도커 서버의 정보 확인 가능 <span style="color:#e11d21">(로컬서버와 동일하게 모든 권한 부여됨)</span>

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image15.png?raw=true)
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image16.png?raw=true)

<br>
### 12\. user 메뉴 통해 권한 관리 가능

<span style="color:#e11d21">(세부적인 권한은 익스텐션 95$ / 연 구매해야 사용가능)</span>

<br>
* 특정 팀 생성 및 유저 생성하여 팀에 포함 설정

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image17.png?raw=true)
<br>
* 엔드포인트 (도커 서버) 그룹 생성

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image18.png?raw=true)
<br>
* 엔드포인트에 접근할 수 있는 팀 or 유저 추가 가능 (하단의 Access) 
* 어드민 계정은 전체 접근 가능

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/image19.png?raw=true)
