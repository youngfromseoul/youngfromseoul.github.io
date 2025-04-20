---
title: "Karpenter를 활용한 Kubernetes Node 관리"
excerpt: "Karpenter의 주요 Concept에 대한 개념을 설명합니다."
categories:
  - Kubernetes
last_modified_at: 2025-02-01T15:02:00+09:00
tags:
  - Kubernetes
  - Karpenter
  - AWS
  - EKS
author_profile: true
read_time: true
toc_label: "Contents"
toc_icon: "cog"
toc: true
toc_sticky: true
---

# Karpenter?

Karpenter는 Kubernetes 클러스터의 노드 수명 주기를 효율적으로 관리하기 위해 설계된 오픈 소스 프로젝트로, 애플리케이션의 요구 사항에 따라 노드를 자동으로 프로비저닝하고, 불필요한 노드를 제거하여 클러스터의 효율성과 비용 최적화가 가능하다.

- 📖 [공식 문서](https://karpenter.sh/docs/)



## Install 

Karpenter의 설치는 비교적 간단하며, 공식 가이드를 따라 클러스터에 배포할 수 있다. 

설치의 경우, 사용하는 환경에 따라 다르기 때문에 해당 문서에서는 아래 가이드로 대체

- 📖 [설치 가이드](https://karpenter.sh/docs/getting-started/getting-started-with-karpenter/)



# Concept

Karpenter의 주요 기능은 다음과 같다.



## NodeClass

nodeClass를 정의하여, 실행되는 Kubernetes Node (EC2)가 어떤 구성을 가지고 실행되어야 하는지 설정할 수 있다.

구성 할 수 있는 스펙들은 EC2에 설정되어야 하는 설정들과 비슷하다. 주요 설정은 다음과 같음.

- Subnet
- Security Group
- Instance Profile
- AMI
- Block Device
- User-data



### NodeClass 예제

```yaml
apiVersion: karpenter.k8s.aws/v1
kind: EC2NodeClass
metadata:
  name: default
spec:
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: "${CLUSTER_NAME}"
        environment: test
   securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: "${CLUSTER_NAME}"
        environment: test
  amiFamily: AL2
  instanceProfile: KarpenterNodeInstanceProfile-${CLUSTER_NAME}
  blockDeviceMappings:
    - deviceName: /dev/xvda
      ebs:
        volumeSize: 50Gi
        volumeType: gp3
        encrypted: true
  tags:
    Name: ${CLUSTER_NAME}-karpenter-node
    Environment: Production
```



### spec.kubelet 구성

node에서 실행되는 kubelet 구성을 사용자 정의할 수 있다. (이전 버전에서는 nodePool에서 정의할 수 있었던 것이 최신 버전에서 nodeClass로 변경되었다.)

```yaml
spec:
  ...
  kubelet:
    clusterDNS: ["10.0.1.100"]
    systemReserved:
      cpu: 100m
      memory: 100Mi
      ephemeral-storage: 1Gi
    kubeReserved:
      cpu: 200m
      memory: 100Mi
      ephemeral-storage: 3Gi
    evictionHard:
      memory.available: 5%
      nodefs.available: 10%
      nodefs.inodesFree: 10%
    evictionSoft:
      memory.available: 500Mi
      nodefs.available: 15%
      nodefs.inodesFree: 15%
    evictionSoftGracePeriod:
      memory.available: 1m
      nodefs.available: 1m30s
      nodefs.inodesFree: 2m
    evictionMaxPodGracePeriod: 60
    imageGCHighThresholdPercent: 85
    imageGCLowThresholdPercent: 80
    cpuCFSQuota: true
    podsPerCore: 2
    maxPods: 20
```

- `systemReserved`
  - 파드가 아닌 os 시스템에 사용되는 데몬 (sshd, udev 등) 을 위해 리소스를 예약해두는 옵션
- `kubeReserved`
  - `kubeReserved`, `container runtime`, `node problem detector` 을 위해 자원을 예약해두는 것
- `evictionHard` / `evictionSoft`
  - 가용 용량이 해당 설정 아래로 떨어지면 커널 OOM 등을 방지하기 위해 파드를 제거 함
  - 하드는 즉시 제거, 소프트는 GracePeriod 시간동안 기다려 줌.
- `evictionSoftGracePeriod`
  - 소프트 이빅션에 대한 유예 시간 설정
- `podsPerCore`
  - 파드의 밀도를 vcpu로 제한
- `maxPods`
  - 파드의 밀도를 개수로 제한



### AMI

AMI는 특정 AMI를 선택하거나, 혹은 AMI Family를 지정할 수 있다. 

AMI Family는 아래 항목 지원

- `AL2`
  - AWS에서 관리하는 amazon linux 2로, ami 관리에 대한 부담이 적음
- `Bottlerocket`
  - 컨테이너 환경에 최적화된 리눅스 기반 오픈소스 운영체제로, 최소 소프트웨어로 구성되어있어, 보안이 강화되고 가볍기 때문에 빠르게 프로비저닝이 가능
- `Ubuntu`
- `Custom`



### AMI Update

managed node group을 사용하면 cluster upgrade 시, node도 rolling update가 되지만, karpenter node는 그렇지 않다. karpenter node의 ami를 최신 버전으로 변경하는 방법은 아래 2가지가 있다.

- nodepool의`expireAfter` 사용
  - 예를 들어 `expireAfter` 를 720h로 설정했을 때, 720h에 한번씩 노드가 만료되고, 재 생성되기 때문에 생성 시점에 최신 버전으로 구성된다.
- drift
  - 현재 클러스터에서 ami 업데이트를 감지하고 자동으로 해당 노드를 drift 한다.



## NodePool

NodePool을 통해 아래와 같은 작업 설정이 가능

- taint를 통해 카펜터가 생성한 노드에서 실행 될 수 있는 파드/네임스페이스를 제한 (gpu node 등)
- 특정 가용영역으로 노드 생성
- 온디맨드, 스팟으로 생성
- 특정 아키텍처로 생성
- 레이블을 통해 분리하여 노드 관리
- 특정 시간 동안 비어있는 노드가 있다면 삭제
- 더 효율적이고 저렴한 노드로 통합



### NodePool 예제

```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
    	nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
      taints:
        - key: key: example.com/special-taint
          effect: NoSchedule
      expireAfter: 720h
      requirements:
        - key: "karpenter.k8s.aws/instance-category"
          operator: In
          values: ["m"]
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["on-demand", "spot"]
        - key: "kubernetes.io/os"
          operator: In
          values: ["linux"]
        - key: "kubernetes.io/arch"
          operator: In
          values: ["amd64"]
       disruption:
         consolidationPolicy: WhenEmptyOrUnderutilized
         consolidateAfter: 5m
         budgets:
        - nodes: 10%
        - schedule: "0 9 * * mon-fri"
          duration: 8h
      		nodes: "0"
  limits:
    cpu: "1000"
    memory: "1000Gi"
  weight: 10

```



Nodepool은  `nodeClassRef` 지정을 통해 NodeClass를 선택할 수 있다.

```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
```



### requirements

requirements를 정의하여, Node의 요구사항을 정의할 수 있다. 

- e.g) 특정 인스턴스 유형 (t3, m5 등), 특정 아키텍처, OS

```yaml
apiVersion: karpenter.sh/v1alpha5
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["t", "m"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["on-demand"]
        - key: karpenter.k8s.aws/instance-generation
          operator: In
          values: ["3", "5"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]

```

지원되는 requirement key는 크게 아래가 있음 (참고 : [Scheduling](https://karpenter.sh/preview/concepts/scheduling/#well-known-labels) )

- az
- instance-type
- os
- arch
- capacity-type (spot, on-demand)
- karpenter.k8s.aws/instance-category (t, m, c 등)
- karpenter.k8s.aws/instance-generation (t3, t2 등)
- karpenter.k8s.aws/instance-family (g4dn 등의 좀더 구체적인)
- karpenter.k8s.aws/instance-size (8xlarge 등의 리소스는 비슷하지만 카테고리가 다른)
- karpenter.k8s.aws/instance-cpu (cpu 수에 맞는 인스턴스 타입)
- etc



### limits

아래와 같이 resource limit 을 설정하는 경우, limit에 다 다르면 더 이상 노드를 생성하지 않는다.

 이때, 파드는 다른 요구 사항에 맞는 노드 풀을 통해 스케쥴링 되거나, 맞는게 없다면 계속 pending 상태로 남게 된다. 

> [!NOTE]
>
> 이 Limit 설정은 생성된 노드(Instance) 단위가 아닌, NodePool 단위이다. 즉 해당 Nodepool을 통해 생성된 모든 노드의 리소스가 Limit을 넘을 수 없다. RI 구매한 것을 활용할 때 유용하다.)



```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: default
spec: 
  limits:
    cpu: 1000
    memory: 1000Gi
```



## Disruption

Karpenter는 불필요한 노드를 자동으로 제거하여 리소스 효율성을 높인다. 주요 전략은 다음과 같다.

- **비어 있는 노드(Empty)**: 일정 시간 동안 파드가 없는 노드를 자동으로 삭제한다.
- **만료된 노드(Expire)**: 설정된 시간이 지난 노드를 교체하여 최신 상태를 유지한다.
- **통합(consolidation)**: 클러스터의 리소스 사용률을 분석하여 노드를 통합하거나 교체한다.
  - resource request, node-selector, node/pod affinity, pod anti-affinity, topology, aws cni 등을 고려해서 가장 효율적인 type 중 low price type을 provisioning 한다.




> [!TIP]
>
> Karpenter Node에 Consolidation이 발생할 때, 기존 활용도가 떨어지는 노드의 Pod들은 모두 eviction 된다. 당연하지만 이때 Rolling Update 형태로 동작하지 않는다. 실제 동작하는 프로세스는 다음과 같다.
>
> 1. 새로운 Node 실행
> 2. 기존 Node 삭제  (이때 Pod Eviction 되면서 각 Pod는 kubescheduler에 의해 적절한 Node로 배치된다.)
>
> 즉, PDB와 Replicas를 적절하게 설정해두어야 서비스 중단이 발생되지 않는다.


