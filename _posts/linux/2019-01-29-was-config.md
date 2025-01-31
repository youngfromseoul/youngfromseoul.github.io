---
title: "[APM] Apache, PHP, MariaDB config(설정) 및 구동"
excerpt: "Apache, PHP, MariaDB config(설정) 및 구동"
categories: 
  - Linux
last_modified_at: 2019-01-29T17:09:00+09:00
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
toc_label: "APM 설치, " 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

## 아파치(Apache) 설정 및 구동
```
[root@web /]# vi /etc/httpd/conf/httpd.conf
User nobody (apache → nobody 변경)
Group nobody (apache → nobody 변경)
```
**ROOT 권한으로 실행된 아파치의 하위 프로세스를 지정한 사용자로 실행** 
**기본값으로 apache 또는 daemon 으로 되어있지만 대부분 nobody로 변경하여 이용한다.**
{: .notice--primary}
```
ServerName xxx.xxx.xxx.xxx:80 (서버 IP 혹은 도메인 입력)
```


## 방화벽 설정 (80 Port Open)
```
[root@web /]# vi /etc/sysconfig/iptables
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT (추가)
```
지난 포스팅에서 이미 설정한 부분
{: .notice--primary}


## 방화벽 재기동 및 설정 확인
```
[root@web /]# systemctl restart iptables
[root@web /]# iptables -nL
```


## Apache 구동 확인
```
[root@web /]# ps -ef |grep httpd
root       1898      1  0 16:37 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     1899   1898  0 16:38 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     1900   1898  0 16:38 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     1901   1898  0 16:38 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     1902   1898  0 16:38 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     1903   1898  0 16:38 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
root       2699   1407  0 17:12 pts/0    00:00:00 grep --color=auto httpd
```


## PHP 환경설정 및 구동
```
[root@web /]# vi /etc/httpd/conf/httpd.conf
<IfModule dir_module>
         DirectoryIndex index.html index.htm index.php (추가)
#AddType application/x-gzip .tgz 아래 추가
AddType application/x-httpd-php .php .html .htm .inc (추가)
AddType application/x-httpd-php-source .phps (추가)
```


## PHP 적용
```
[root@web /]# vi /var/www/html/phpinfo.php
<?php phpinfo(); ?>
```

## https 재시작
```
[root@web /]# systemctl restart httpd
[root@web /]# systemctl restart httpd
```

### MariaDB 구동 및 설정

```
[root@web /]# systemctl start mariadb
[root@web /]# ps -ef | grep mysql
[root@web /]# mysql_secure_installation
mysql_secure_installation  // sql 초기설정
Enter current password for root (enter for none): Enter
Set root password? [Y/n] Y  //DB root 패스워드 설정
New password:  
Re-enter new password:  
Remove anonymous users? [Y/n] Y   
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y 
Reload privilege tables now? [Y/n] Y 
```

## MariaDB 접속
```
[root@web /]# mysql -u root -p
Enter password:
```






