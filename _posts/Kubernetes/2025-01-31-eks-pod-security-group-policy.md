---
title: "[EKS] Pod Security Group 사용하기"
excerpt: "Pod Security Group 사용하기"
categories: 
  - Kubernetes
last_modified_at: 2025-01-31T15:02:00+09:00
tags: 
    - Kubernetes
    - Pod Security Group
    - AWS
    - EKS
author_profile: true
read_time: true
toc_label: "Contents" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---



## Pod Security Group 이란?

AWS Security Group을 생성하여, 각 Pod 별로 고유한 pod security group을 가질수 있도록 구성할 수 있도록 하는 기능이다.



EKS 외 다른 쿠버네티스 환경은 사용해보지 않아서, 다른 매니지드 쿠버네티스 서비스에서 지원되는지는 잘 모르겠다.

- ref: https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/security-groups-for-pods.html#supported-instance-types



## **고려 사항**

- windows 노드는 사용 불가
- IPv6는 사용 불가
- t 패밀리 인스턴스 지원 불가 (지원 목록: https://github.com/aws/amazon-vpc-resource-controller-k8s/blob/release/pkg/aws/vpc/limits.go )
- [Nodelocal DNSCache](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)를 사용하는 클러스터에서 보안 그룹을 사용하는 Amazon EC2 노드에서 실행되는 Pods는 `POD_SECURITY_GROUP_ENFORCING_MODE`=`standard`로 설정된 버전 `1.11.0` 이상의 Amazon VPC CNI 플러그 인에서만 지원
- 연결된 보안 그룹이 있는 pods를 통해 Calico 네트워크 정책을 사용하려면 버전 `1.11.0` 이상의 Amazon VPC CNI 플러그 인을 사용하고 `POD_SECURITY_GROUP_ENFORCING_MODE`=`standard`로 설정
- 포드의 보안 그룹은 변동이 많은 pods의 경우 pod 시작 지연 시간 증가로 이어질 수 있다. 이는 리소스 컨트롤러의 속도 제한 때문





## **구성** 방법

### `ENABLE_POD_ENI` 환경 변수 활성화

`aws-node` DaemonSet에서 사용되는 환경 변수로, `ENABLE_POD_ENI`를 `true`로 설정하면, AWS CNI를 사용하여 Elastic Network Interface(ENI)를 포드에 직접 할당할 수 있음

```
kubectl set env daemonset aws-node -n kube-system ENABLE_POD_ENI=true
```



다만, 인스턴스 유형 별, 할당할 수 있는 ENI 개수가 한정적이므로, 노드 당 가용 파드 수가 줄어들게 된다. (`BranchInterface` 개수)

아래 명령어로, 사용 가능한 node 확인 가능

```
kubectl get nodes -o wide -l vpc.amazonaws.com/has-trunk-attached=true
```

트렁크 네트워크 인터페이스가 만들어지면 트렁크 또는 표준 네트워크 인터페이스에서 Pods에 보조 IP 주소를 할당한다. 

노드가 삭제되면 트렁크 인터페이스가 자동으로 삭제된다.



이후 단계에서 Pod에 대한 보안 그룹을 배포하면 VPC 리소스 컨트롤러가 `aws-k8s-branch-eni`의 설명과 함께 *브랜치 네트워크 인터페이스*라는 특수한 네트워크 인터페이스를 생성하고 거기에 보안 그룹을 연결한다.

브랜치 네트워크 인터페이스는 노드에 연결된 표준 및 트렁크 네트워크 인터페이스와 함께 생성된다. 

livendess 또는 readiness 프로브를 사용하는 경우 `kubelet`이 TCP를 사용하여 브랜치 네트워크 인터페이스의 Pods에 연결할 수 있도록 TCP 초기 demux를 사용 정지해야 한다.



 TCP 초기 Demux를 비활성화하려면 다음 명령을 실행

```
kubectl patch daemonset aws-node -n kube-system \\
  -p '{"spec": {"template": {"spec": {"initContainers": [{"env":[{"name":"DISABLE_TCP_EARLY_DEMUX","value":"true"}],"name":"aws-vpc-cni-init"}]}}}}'
```



### 리소스 적용 예시

podSelector를 통해, label이 매칭되면 연결됨.

```
apiVersion: vpcresources.k8s.aws/v1beta1
kind: SecurityGroupPolicy
metadata:
  name: nginx-security-group-policy
spec:
  podSelector:
    matchLabels:
      app: nginx-tg
  securityGroups:
    groupIds:
      - sg-xxxxxxxxxxxxxx
      - sg-xxxxxxxxxxxxxx
```



> [!TIP]
>
> Pod 에 SG를 할당하면, Node의 SG는 상속받지 않는다. 즉, 클러스터 내부 통신을 위한 규칙을 별도로 추가하거나, Node의 SG도 함께 등록해주어야 한다.



