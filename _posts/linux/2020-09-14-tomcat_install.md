---
title: "[Centos7] Tomcat, JDK 설치방법"
excerpt: "[Centos7] Tomcat, JDK 설치방법"
categories: 
  - Linux
last_modified_at: 2020-09-14T14:55:00+09:00
tags: 
    - Linux
    - CentOS
    - Tomcat
    - JDK
    - Java
author_profile: true
read_time: true
toc_label: "Tomcat" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---
![](https://blog.kakaocdn.net/dn/4Ka59/btqC49T2Bzl/EegI0P2KW1K5FQTXBlekYk/img.png)

- - -

<span style="color:  #333333;;">서버환경 : CentOS7</span>

<span style="color:  #333333;;">톰캣 버전 : apache tomcat 9</span>

<span style="color:  #333333;;">Java 버전: JDK 1.8</span>

- - -

### **1.** **Java 다운로드 및 설치**

<span style="color:  #333333;;">java의 경우, 오라클 로그인이 되어야 가입 가능하기 때문에 아래 주소에서 다운로드</span>

https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html﻿

<br>
<span style="color:  #333333;;">**Linux x64 Compressed Archive 다운로드**</span>

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/tomcat1.png?raw=true)

<span style="color:  #333333;;">**다운로드 완료 후, FTP 등을 통하여 서버에 업로드**</span>
<br>
```
[root@test ~]# tar -xvf jdk-8u241-linux-x64.tar.gz 
[root@test ~]# mv jdk1.8.0_241/ /usr/local/jdk
```

<span style="color:  #333333;;"></span>

<span style="color:  #333333;;">**자바 환경 변수 설정**</span>
<br>
```
[root@test ~]# vi /etc/profile 

export JAVA_HOME=/usr/local/jdk 
export PATH=$PATH:$JAVA_HOME/bin 
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar 

[root@test ~]# source /etc/profile 

[root@test ~]# java -version 
java version "1.8.0_241" 
Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
```

<span style="color:  #333333;;"></span>

### **2.** **Apache Tomcat 다운로드 및 설치**

```
[root@test ~]# wget http://archive.apache.org/dist/tomcat/tomcat-9/v9.0.8/bin/apache-tomcat-9.0.8.tar.gz 
[root@test ~]# tar -xvf apache-tomcat-9.0.8.tar.gz 
[root@test ~]# mv apache-tomcat-9.0.8 /usr/local/tomcat
```

<span style="color:  #333333;;"></span>

### **3\. Tomcat 실행 및 웹 페이지 확인**

```
[root@test ~]# /usr/local/tomcat/bin/startup.sh 
Using CATALINA_BASE: /usr/local/tomcat 
Using CATALINA_HOME: /usr/local/tomcat 
Using CATALINA_TMPDIR: /usr/local/tomcat/temp 
Using JRE_HOME: /usr/local/jdk 
Using CLASSPATH: /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar 
Tomcat started.
```

```
[root@test bin]# netstat -nltp 

Active Internet connections (only servers) 
Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name 

tcp6 0 0 127.0.0.1:8005 :::* LISTEN 1574/java 
tcp6 0 0 :::8009 :::* LISTEN 1574/java 
tcp6 0 0 :::8080 :::* LISTEN 1574/java
```

<span style="color:  #333333;;">8080, 8005, 8009 포트 실행된 모습 확인</span>

<span style="color:  #333333;;"></span>

<span style="color:  #333333;;"></span>

<span style="color:  #333333;;">**주소 창에 IP:8080으로 접속하여 톰캣 기본페이지 확인**</span>

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/tomcat2.png?raw=true)
