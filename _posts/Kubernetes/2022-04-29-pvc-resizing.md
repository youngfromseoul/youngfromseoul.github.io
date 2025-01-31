---
title: "[Kubernetes] EKS PVC Resizing"
excerpt: "EKS PVC 확장하기"
categories: 
  - Kubernetes
last_modified_at: 2022-04-29T17:37:00+09:00
tags: 
    - Kubernetes
    - Infra
    - DevOps
author_profile: true
read_time: true
toc_label: "pvc" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

# PVC (Persistent Volumes) 정보 확인

```
kubectl get pvc
```

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/eks-volume-1.PNG?raw=true)

* 증설 대상인 data-zabbix-postgresql-0 의 storageclass 확인 (gp2)

# SC (Storage Class) 정보 확인

```
kubectl get sc
```

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/eks-volume-2.PNG?raw=true)

* 확장이 필요한 storage class의 ALLOWVOLUMEEXPANSION 확인 시, true로 확인 / false일 경우 true로 변경
> ALLOWVOLUMEEXPANSION 옵션은, 볼륨 확장을 허용할건지에 대한 설정으로, true로 설정해주어야 확장이 가능하다.

# PVC (Persistent Volumes) 확장

```
kubectl edit pvc data-zabbix-postgresql-0 -n zabbix
```

```
storage: 8Gi -> 20Gi
```

* 8Gi → 20Gi 로 변경

# 변경 확인
![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/eks-volume-3.PNG?raw=true)

* 콘솔 확인 시, 옵티마이징 중으로 확인 됨

<br>

```
kubectl get pvc
```

![image.png](https://github.com/youngfromseoul/youngfromseoul.github.io/blob/master/assets/images/eks-volume-4.PNG?raw=true)

* 20Gi 변경 확인