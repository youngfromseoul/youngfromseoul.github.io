---
title: "리눅스 쉘 스크립트 활용"
excerpt: "쉘 스크립트 사용 방법"
categories: 
  - Linux
last_modified_at: 2021-02-18T09:29:00+09:00
tags: 
    - Linux
    - CentOS
    - Redhat
author_profile: true
read_time: true
toc_label: "Shell Script" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---
Linux Shell Script 사용 방법에 대해 정리하였습니다.
<br>
## pushd /  popd

* pushd는 현재 경로를 저장하는 명령어
* popd는 저장한 경로로 이동하는 명령어

``` bash
[root@localohst local]# pwd
/usr/local
[root@localohst local]# pushd .
/usr/local /usr/local
[root@localohst local]# cd /
[root@localohst /]# pwd/
[root@localohst /]# popd
/usr/local
[root@localohst local]# pwd
/usr/local
```
<br>
* cd - 명령 사용 시, 현재 디렉토리와 저장한 디렉토리를 번갈아가며 이동

```
[root@localohst local]# cd -
/
[root@localohst /]# cd -
/usr/local
[root@localohst local]#
```
<br>
<br>
## printf

* printf는 형식화된 출력이 가능

```
[root@localohst ~]# printf "%05d\n" 1
00001
[root@localohst ~]# printf "%03d\n" 1
001
[root@localohst ~]#
```
<br>
* -v 옵션을 통해 변수로 저장할 수도 있음

```
[root@localohst ~]# name=michael; printf -v legend "%s jackson" $name; echo $legend
michael jackson
```
<br>
<br>
## read

* 변수를 입력 받을 수 있음

```
[root@localhost ~]# read num
003
[root@localhost ~]# echo $num
003
[root@localhost ~]#
```
<br>
```
[root@localhost ~]# read -p "what is your phon number? : " v
what is your phon number? : 010-000-0000
[root@localhost ~]# echo $v
010-000-0000
```
<br>
## while loop

```
[root@localhost ~]# no=1; while (( no < 10 )); do printf "%02d\n" $no; ((no++)); done
01
02
03
04
05
06
07
08
09
```
<br>
<br>

