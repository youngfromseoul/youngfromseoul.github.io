# Kapenter 구성 테스트

## **Install**

인스톨은 간단하기 때문에 아래의 가이드에 따라 구성

- [https://karpenter.sh/v0.27.3/getting-started/getting-started-with-karpenter/](https://karpenter.sh/v0.27.3/getting-started/getting-started-with-karpenter/)

## **Provisioner**

프로비저너는 카펜터가 생성할 수 있는 노드의 정보를 설정

Ref: [https://karpenter.sh/v0.27.3/concepts/provisioners/](https://karpenter.sh/v0.27.3/concepts/provisioners/)

프로비저너를 통해 아래와 같은 작업 설정이 가능

- taint를 통해 카펜터가 생성한 노드에서 실행 될 수 있는 파드/네임스페이스를 제한 (gpu node 등)
- 특정 가용영역으로 노드 생성
- 온디맨드, 스팟으로 생성
- 특정 아키텍처로 생성
- 프로비저너는 여러개를 생성할 수 있으며, 레이블을 통해 서비스 기반으로 분리하여 노드 생성
- 특정 시간 동안 스케쥴되지 않은 노드가 있다면 삭제

### requirements

예시) 특정 인스턴스 유형 (t3, m5 등), 특정 아키텍처, OS

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
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
  providerRef:
    name: default
  ttlSecondsAfterEmpty: 60

```

지원되는 requirement key는 크게 아래가 있음 (참고 : [Scheduling](https://karpenter.sh/preview/concepts/scheduling/#well-known-labels) )

[https://karpenter.sh/favicon.ico](https://karpenter.sh/favicon.ico)

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
- 등등..

### limit

아래와 같이 리소스 리밋을 설정하는 경우, 더 이상 노드를 생성하지 않음, 다만 퍼센테이지로는 설정이 안됨

- 해당 파드는 다른 요구 사항에 맞는 프로비저너를 통해 스케쥴링 되거나, 맞는게 없다면 계속 pending 상태로 남게 됨

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot"]
  limits:
    resources:
      cpu: 1000
      memory: 1000Gi
      nvidia.com/gpu: 2
```

### spec.kubelet 구성

node에서 실행되는 kubelet 구성을 사용자 정의할 수 있다.

```
spec:
  ...
  kubeletConfiguration:
    clusterDNS: ["10.0.1.100"]
    containerRuntime: containerd
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

- `containerRuntime`
    - `dockerd` 컨테이너 런타임 사용 가능 (디폴트는 `containerd`)
- `systemReserved`
    - 파드가 아닌 os 시스템에 사용되는 데몬 (sshd, udev 등) 을 위해 리소스를 예약해두는 옵션
- `kubeReserved`
    - `kubeReserved`, `container runtime`, `node problem detector` 을 위해 자원을 예약해두는 것
- `evictionHard` / `evictionSoft`
    - 가용 용량이 해당 설정 아래로 떨어지면 OOM 등을 방지하기 위해 파드를 제거 함
    - 하드는 즉시 제거, 소프트는 정상 종료 후 재 스케쥴링될 수 있도록 유예 함
- `evictionSoftGracePeriod`
    - 소프트 이빅션에 대한 유예 시간 설정
- `podsPerCore`
    - 파드의 밀도를 vcpu로 제한
- `maxPods`
    - 파드의 밀도를 개수로 제한

## **Node Templates**

node template에는 아래의 설정을 지정할 수 있다.

- subnet
- security group
- instance profile
- ami
- block device
- user-data

provisioner는 node `spec.providerRef` 지정을 통해 nodeTemplate을 선택할 수 있다.

- default는 default

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  providerRef:
    name: default
```

아래는 tag를 통해 discovery해서 서브넷과, SG를 적용

```
    apiVersion: karpenter.k8s.aws/v1alpha1
    kind: AWSNodeTemplate
    metadata:
      name: default
    spec:
      subnetSelector:
        karpenter.sh/discovery: ${module.eks.cluster_name}
      securityGroupSelector:
        karpenter.sh/discovery: ${module.eks.cluster_name}
      tags:
        karpenter.sh/discovery: ${module.eks.cluster_name}
```

이 외에도 다양한 옵션들이 있으며, 운영하면서 필요한 부분을 문서 보면서 찾아서 하면 될것 같음

### Launch Template

Karpenter는 기본적으로 node가 생성될 때, node template에 정의한 spec 대로 launch template을 만든다.

예전 버전에서는 launch template id를 지정할 수 있었던 것으로 보이나, 현재 문서에서는 찾아볼 수 없음

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  providerRef:
    name: default
---
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
metadata:
  name: default
spec:
  subnetSelector: { ... }        # required, discovers tagged subnets to attach to instances
  securityGroupSelector: { ... } # required, discovers tagged security groups to attach to instances
  instanceProfile: "..."         # optional, overrides the node's identity from global settings
  amiFamily: "..."               # optional, resolves a default ami and userdata
  amiSelector: { ... }           # optional, discovers tagged amis to override the amiFamily's default
  userData: "..."                # optional, overrides autogenerated userdata with a merge semantic
  tags: { ... }                  # optional, propagates tags to underlying EC2 resources
  metadataOptions: { ... }       # optional, configures IMDS for the instance
  blockDeviceMappings: [ ... ]   # optional, configures storage devices for the instance
  detailedMonitoring: "..."      # optional, configures detailed monitoring for the instance
```

### AMI

AMI는 특정 AMI를 선택하거나, 혹은 AMI Family를 지정할 수 있다. 다만, AMI를 직접 관리하는 것은 업데이트 관리가 너무 어렵다. (kubelet 을 수동으로 업데이트 해주어야 한다.)

AMI Family는 아래 항목 지원

- `AL2`
    - AWS에서 관리하는 amazon linux 2로, ami 관리에 대한 부담이 적음
- `Bottlerocket`
    - 컨테이너 환경에 최적화된 리눅스 기반 오픈소스 운영체제로, 최소 소프트웨어로 구성되어있어, 보안이 강화되고 가볍기 때문에 빠르게 프로비저닝이 가능
- `Ubuntu`
- `Custom`
    - custom 선택 시, ami select가 가능

### AMI Update

managed node group을 사용하면 cluster upgrade 시, node도 rolling update가 되지만, karpenter node는 그렇지 않음. karpenter node의 ami를 최신 버전으로 변경하는 방법은 아래 2가지가 있다.

- `ttlSecondsUntilExpired` 사용
    - 예를 들어 `ttlSecondsUntilExpired` 를 1달로 설정했을 때, 1달에 한번씩 노드가 만료되고, 재 생성되기 때문에 재 생성 시점에 최신 버전으로 구성된다.
- drift 사용
    - 현재 클러스터에서 ami 업데이트를 감지하고 자동으로 해당 노드를 drift 한다.
    - featureGates.driftEnabled 을 true로 설정해야 됨 (default는 false)
        - 해당 옵션을 켜둔 상태

## **Scheduling**

node 혹은 pod를 배치하는 전략에 대한 설명

### weight

여러개의 프로비저너를 두고, 가중치를 줄 수 있음

예시) SP 구매한 m5 타입을 사용하도록 가중치 50을 주고, limit 100을 주었음.

이 경우, 절반은 m5.large로 우선 순위로 실행. 다만 cpu limit 100을 주었기 때문에, limit 100이 넘어서면 다른 프로비저너로 스케쥴링이 된다. 또한, 가중치가 높은 것이 우선순위를 가짐

(공식 문서에는 기재되어있지 않아 테스트해봐야하지만, gpt에 따르면 모든 프로비저너의 가중치가 100을 넘어서는 경우, 스케쥴링이 되지 않는다고 한다. 또한 분산은 가중치가 높은 순대로 라운드 로빈으로 생성이된다고 한다.)

- savings-plan 프로비저너

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: savings-plan-instance
spec:
  weight: 50
  requirements:
  - key: "node.kubernetes.io/instance-type"
    operator: In
    values: ["m5.large"]
  limits:
    cpu: 100
```

- 기본 프로비저너

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  requirements:
  - key: karpenter.sh/capacity-type
    operator: In
    values: ["spot", "on-demand"]
  - key: kubernetes.io/arch
    operator: In
    values: ["amd64"]
```

### taint

gpu 등 일반적인 파드가 실행되면 안되는 특수의 경우는 taint를 통해 스케쥴링 되지 않도록 함

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: gpu
spec:
  ttlSecondsAfterEmpty: 60
  requirements:
  - key: node.kubernetes.io/instance-type
    operator: In
    values: ["p3.8xlarge", "p3.16xlarge"]
  taints:
  - key: nvidia.com/gpu
    value: "true"
    effect: NoSchedule
```

톨로테이션을 통해, 허용되어야만 스케쥴링 가능

### node selector

아래에 명시된 알려진 label은 바로 사용 가능

[https://karpenter.sh/v0.27.3/concepts/scheduling/#well-known-labels](https://karpenter.sh/v0.27.3/concepts/scheduling/#well-known-labels)

### 사용자 정의 labels

사용자 정의 label을 사용하는 경우, 해당 label을 requiredments에 추가해주어야 인식

- 프로비저너

```
requirements:
  - key: node-type: workernode
    operator: Exists
```

- 디플로이먼트

```
  nodeSelector:
    node-type: workernode
```

### affinity / antiaffinity

- 어피니티, 안티 어피니티를 통해 좀 더 세분화하여 선호도에 따라 노드를 선택할 수 있음

```
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: system
            operator: In
            values:
            - backend
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: inflate
        topologyKey: kubernetes.io/hostname
```

## **deprovisioning**

node 삭제를 결정하는 방법

- **노드가 비어있을 경우** - `ttlSecondsAfterEmpty` 시간 동안 기다린 후 삭제
- **노드 만료** - `ttlSecondsUntilExpired` 시간 이후 삭제 (ami 업데이트 등 최신 버전이 아닌 오래된 노드를 삭제하는 경우)
- **통합** - 카펜터가 알아서 사용량에 따라 더 저렴한 노드로 변경하거나, 더 큰 리소스를 가진 노드로 합쳐는 등 작업을 하면서 기존 노드가 삭제됨
- **Drift** - ami 변경 등 노드에 변경사항이 있는 경우, drift 노드로 판단하고 디프로비저닝 함
- **중단** - 인스턴스 상태가 fail이거나, 스팟 중단 등의 이벤트를 감지하고 노드를 cordon, drain, terminate 처리를 함

### consolidation

Karpenter 0.16.0 버전 이상에서는 기본적으로 활성화 되어있음

클러스터 통합 매커니즘은 아래 2가지가 있음

- 삭제 - 특정 노드에서 실행되는 모든 파드가 다른 노드에서 실행될 수 있는 경우, 통합하고 삭제함
- 교체 - 특정 노드에서 실행되는 모든 파드가 더 저렴한 노드로 교체가 가능한 경우 교체

통합 작업을 식별하기 위한 매커니즘은 아래 3가지

- 비어있는 노드 - 비어있는 노드를 병렬로 동시에 삭제
- 다중 노드 통합 - 2개 이상의 노드는 2개 노드 가격보다 더 저렴한 1개의 노드로 통합
- 단일 노드 통합 - 단일 노드는 더 저렴한 단일 노드로 통합

또한 삭제, 교체 작업이 가능한 노드가 많은 경우, 아래 사항을 우선순위로 고려하여 통합

- 더 작은 파드를 실행하는 노드
- 곧 만료되는 노드
- 우선 순위가 낮은 파드가 있는 노드

### interruption

카펜터는 aws sqs를 통해 아래의 aws 이벤트를 수신해서 사전에 cordon, drain 작업을 진행함

- spot 중단 경고
- 예약된 상태 변경 이벤트 (유지 관리 이벤트)
- 인스턴스 종료
- 인스턴스 중지

### drift

카펜터는 노드에서 사용되는 AMI가 nodetemplate에서 설정한 ami와 일치하지 않는 경우, drift로 판단하고, 자동으로 디프로비저닝 함

혹은 사용자가 수동으로 노드에 `karpenter.sh/voluntary-disruption: "drifted"` 어노테이션을 달아서 드리프트 시킬수도 있다.

## **deprovisioning disable**

### pod evict

`karpenter.sh/do-not-evict: "true"`Pod에 주석을 설정하여 Pod를 제거하지 않도록 선택할 수 있다.

예를 들어, 실시간이 필요한 파드 혹은 중단되면 다시 시작해야하는 긴 배치 성 파드는 아래와 같이 어노테이션을 달아주면 해당 파드가 실행되고 있는 노드는 자발적으로 제거하지 않는다. (제거하면 파드가 이동되면서 세션 달라지게 되는 등)

```
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        karpenter.sh/do-not-evict: "true"
```

해당 주석을 달게되면 노드 통합, 노드 만료, 드리프트, 비어있는 노드 등 디프로비저닝이 먹히지 않음

### node

프로비저너에서도 `arpenter.sh/do-not-consolidate: "true"` 어노테이션을 달게되면, 해당 프로비저너에서 시작한 노드는 통합 작업을 하지 않음

## **파드 밀도 제어**

노드 당 파드의 수를 제한할 수 있음 (kubernetes의 기본 제한은 110개, aws eks ami은 인스턴스 유형에 따라 다름)

### aws 제한 사항

aws의 경우, 노드에서 실행될 수 있는 파드의 수는 eni 수와, eni에 할당할 수 있는 ip 주소 개수로 제한됨

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI)

### Static Pod Density

프로비저너에서 `.spec.kubeletConfiguration.maxPods`로 지정 가능, 지정 시 스케쥴링 할때 maxpod를 고려하여 스케쥴링

### Dynamic Pod Density

`.spec.kubeletConfiguration.podPerCore` 를 통해 vCPU 개수를 기반으로 파드 밀도를 고려해서 스케쥴링

## **Metric**

아래의 메트릭들을 컨트롤러가 수집해서 작업들을 판단한다.

[https://karpenter.sh/v0.27.3/concepts/metrics/](https://karpenter.sh/v0.27.3/concepts/metrics/)

## **Test**

테스트 디플로이를 통해 노드 스케일 아웃 테스트

```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inflate
spec:
  replicas: 0
  selector:
    matchLabels:
      app: inflate
  template:
    metadata:
      labels:
        app: inflate
    spec:
      terminationGracePeriodSeconds: 0
      containers:
        - name: inflate
          image: public.ecr.aws/eks-distro/kubernetes/pause:3.7
          resources:
            requests:
              cpu: 1
EOF
```

현재 노드 상태

![image.png](Kapenter%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20169fc20319f08041b9a8d1cb107b57c1/image.png)

`replicas` 를 5로 늘려서 테스트

```
kubectl scale deployment inflate --replicas 5
```

`replicas`를 늘리자마자, pod 스케쥴링을 시작하고, 스케쥴링할 노드가 없자 바로 노드 하나를 추가한다. (약 5초 소요)

![image.png](Kapenter%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20169fc20319f08041b9a8d1cb107b57c1/image%201.png)

약 30초만에 Ready 상태로 들어가면서 파드가 스케쥴링되면서 컨테이너를 생성하고 있음

![image.png](Kapenter%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20169fc20319f08041b9a8d1cb107b57c1/image%202.png)

로그 상으로는 언스케쥴 파드를 감지하고, 노드 생성을 요청하는데 약 2초정도 소요

`replicas` 를 다시 0으로 변경

```
kubectl scale deployment inflate --replicas 0
```

변경 즉시, 카펜터 컨트롤러에서 감지하고, TTL로 설정한 60초 이후 바로 삭제

![image.png](Kapenter%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20169fc20319f08041b9a8d1cb107b57c1/image%203.png)

## **현재 구성 정보**

### Provisioner

- 동일한 셋으로 node-type infra, workernode로 분리

```
resource "kubectl_manifest" "karpenter_provisioner" {
  yaml_body = <<-YAML
    apiVersion: karpenter.sh/v1alpha5
    kind: Provisioner
    metadata:
      name: default
    spec:
      providerRef:
        name: al2-workernode
      label:
        node-type: workernode
      requirements:
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["m"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["on-demand", "spot"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
      ttlSecondsAfterEmpty: 300
  YAML
  depends_on = [
    helm_release.karpenter
  ]
}
```

### node template

- 동일한 set으로, ec2 tag name workernode, infra로 분리

```
resource "kubectl_manifest" "karpenter_node_template" {
  yaml_body = <<-YAML
    apiVersion: karpenter.k8s.aws/v1alpha1
    kind: AWSNodeTemplate
    metadata:
      name: al2-workernode
    spec:
      subnetSelector:
        karpenter.sh/discovery: ${module.eks.cluster_name}
      securityGroupSelector:
        karpenter.sh/discovery: ${module.eks.cluster_name}
      amiFamily: AL2
      instanceProfile: KarpenterInstanceNodeRole
      blockDeviceMappings:
        - deviceName: /dev/xvda
          ebs:
            volumeSize: 50Gi
            volumeType: gp3
            encrypted: true
      userData: |
        #!/bin/bash
        mkdir -p ~ec2-user/.ssh/
        aws s3 cp s3://mrt-devops-dev/keypair/authorized_keys ~ec2-user/.ssh/authorized_keys
        chmod -R go-w ~ec2-user/.ssh/authorized_keys
        chown -R ec2-user ~ec2-user/.ssh
      tags:
        karpenter.sh/discovery: ${module.eks.cluster_name}
        Name: ${module.eks.cluster_name}-workernode
        Service: EKS
        Department: SRE
        Revision: 3.0
  YAML
  depends_on = [
    helm_release.karpenter
  ]
}
```

### Environment

```
  set {
    name  = "settings.aws.enablePodENI"
    value = true
  }
  set {
    name  = "settings.featureGates.driftEnabled"
    value = true
  }
}
```

`enablePodENI`: pod security를 사용하기 위해, pod에 ENI 할당하는 것을 허용. 이 경우, karpenter에서 branch ENI 사용이 가능한 타입으로 프로비저닝 하게 된다.

`featureGates.driftEnabled`: aws managed ami에서 신규 버전이 릴리즈 되는 경우, 이전 node들을 drift 상태로 변경하고, 새로운 노드를 프로비저닝 하게 됨. * ( 이부분은 노드가 동시에 내려가는 상황이 발생되는지, 최소 노드 유지 수 혹은 롤링 업데이트 등의 여부를 추가 체크 예정)