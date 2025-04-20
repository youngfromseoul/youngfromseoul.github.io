---
title: "CentOS7 Web Server 구축하기"
excerpt: "CentOS7에서 확인 가능한 명령어 및 WAS 구축 가이드"
categories: 
  - Linux
last_modified_at: 2019-01-29T14:39:00+09:00
tags: 
    - Linux
    - CentOS
    - Redhat
    - WAS
    - PHP
    - Apache
    - MariaDB
author_profile: true
read_time: true
toc_label: "WAS 설치하기" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## Kernel 정보 확인
```
[root@gpu1 ~]# uname -r
3.10.0-693.el7.x86_64
```

## OS Version 확인
```
[root@gpu1 ~]# cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```

## FileSystem 확인
```
[root@gpu1 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G  5.3G   12G  31% /
devtmpfs                 478M     0  478M   0% /dev
tmpfs                    489M     0  489M   0% /dev/shm
tmpfs                    489M  6.7M  482M   2% /run
tmpfs                    489M     0  489M   0% /sys/fs/cgroup
/dev/sda1               1014M  125M  890M  13% /boot
tmpfs                     98M     0   98M   0% /run/user/0
```

## Memory 및 Swap 확인
```
[root@gpu1 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            976         134         699           6         142         679
Swap:          2047           0        2047
```

## CPU core 확인
```
cat /proc/cpuinfo | grep processor | wc -l
1
```


## KST Time Zone 설정
```
[root@gpu1 ~]# ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
[root@gpu1 ~]# date
Tue Jan 29 14:23:06 KST 2019
```

## Hostname 변경
```
[root@gpu1 ~]# hostnamectl set-hostname WAS-TEST
[root@gpu1 ~]# logout
[root@was-test ~]#
```


```
[root@was-test ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
success
[root@was-test ~]# firewall-cmd --reload
success
```

## FTP 설치
```
[root@was-test ~]# yum -y install vsftpd
Complete!
[root@was-test ~]# systemctl enable vsftpd.service
vsftpd.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig vsftpd on
[root@was-test ~]# systemctl start vsftpd
```

## 방화벽 설치
```
[root@was-test ~]# yum -y install system-config-firewall-tui
```

## Port허용
```
[root@was-test ~]# #vi /etc/sysconfig/iptables
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 21 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```
  ▶ 포트 80 : 아파치
  ▶ 포트 22 : SSH
  ▶ 포트 21 : FTP
  ▶ 포트 3306 : MySql 


## Firewalld 중지
```
[root@was-test ~]# systemctl mask firewalld
Created symlink from /etc/systemd/system/firewalld.service to /dev/null.

[root@was-test ~]# systemctl stop firewalld
[root@was-test ~]# systemctl status firewalld
● firewalld.service
   Loaded: masked (/dev/null; bad)
   Active: inactive (dead) since Tue 2019-01-29 14:41:46 KST; 2s ago
 Main PID: 782 (code=exited, status=0/SUCCESS)

Jan 29 14:06:11 gpu1 firewalld[782]: WARNING: ICMP type 'reject-route' is not supported by t...pv6.
Jan 29 14:06:11 gpu1 firewalld[782]: WARNING: reject-route: INVALID_ICMPTYPE: No supported I...ime.
Jan 29 14:26:57 was-test firewalld[782]: WARNING: ICMP type 'beyond-scope' is not supported b...v6.
Jan 29 14:26:57 was-test firewalld[782]: WARNING: beyond-scope: INVALID_ICMPTYPE: No supporte...me.
Jan 29 14:26:57 was-test firewalld[782]: WARNING: ICMP type 'failed-policy' is not supported ...v6.
Jan 29 14:26:57 was-test firewalld[782]: WARNING: failed-policy: INVALID_ICMPTYPE: No support...me.
Jan 29 14:26:57 was-test firewalld[782]: WARNING: ICMP type 'reject-route' is not supported b...v6.
Jan 29 14:26:57 was-test firewalld[782]: WARNING: reject-route: INVALID_ICMPTYPE: No supporte...me.
Jan 29 14:41:46 was-test systemd[1]: Stopping firewalld.service...
Jan 29 14:41:46 was-test systemd[1]: Stopped firewalld.service.
Hint: Some lines were ellipsized, use -l to show in full.
```

## iptables 시작 및 Enable
```
[root@was-test ~]# systemctl start iptables
[root@was-test ~]# systemctl enable iptables.service
[root@was-test ~]# systemctl status iptables.service
```

## 의존성 라이브러리 설치
```
[root@was-test /]# rpm -qa libjpeg* libpng* freetype* gd-* gcc gcc-c++ gdbm-devel libtermcap-devel
[root@was-test /]# yum install libjpeg* libpng* freetype* gd-* gcc gcc-c++ gdbm-devel libtermcap-devel
```

## Apache 설치
```
[root@was-test ~]# yum install httpd
[root@was-test ~]# systemctl enable httpd
[root@was-test /]# systemctl start httpd
[root@was-test ~]# systemctl status httpd
```

## Maria DB Repo 추가

-배포사이트 : http://mariadb.org/
-버전별 셋팅방법 : http://downloads.mariadb.org/mariadb/repositories
{: .notice--info}

```
[root@was-test /]# vi /etc/yum.repos.d/MariaDB.repo
========================================
# MariaDB 10.1 CentOS repository list
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
========================================
 ```
 
 
 ## Maria DB 10.1 설치
 ```
[root@was-test /]# yum install MariaDB-server MariaDB-client
```


## PHP 7 설치

버전별 참고사이트  : https://webtatic.com/projects/yum-repository/
{: .notice--info}
```
[root@was-test /]# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
[root@was-test /]# rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
[root@was-test /]# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
[root@was-test /]# yum install php70w
```

## 자주쓰는 관련 PHP 설치
```
[root@was-test /]# yum install php70w-mysql php70w-pdo php70w-pgsql php70w-odbc php70w-mbstring php70w-mcrypt php70w-gd
[root@was-test /]# yum install php70w-pear php70w-pdo_dblib php70w-pecl-imagick php70w-pecl-imagick-devel php70w-xml php70w-xmlrpc
[root@was-test /]# yum search php70w
```

## 설치 확인
```
[root@was-test /]# httpd -v  
[root@was-test /]# php -v    
[root@was-test /]# mysql -v 
```
 
 
 
 
다음 포스팅 자료에서는 실제 설정 및 구동에 대한 내용을 포스팅 하도록 하겠습니다.


