---
title: "Mac에서 Multipass로 K3s 클러스터 구축하기"
excerpt: "Multipass를 사용하여 Mac에서 간단하게 K3s 클러스터를 구축하는 방법을 설명합니다."
categories:
  - Kubernetes
last_modified_at: 2025-02-03T10:23:00+09:00
tags:
  - Kubernetes
  - K3s
  - Multipass
  - Mac
author_profile: true
read_time: true
toc_label: "Contents"
toc_icon: "cog"
toc: true
toc_sticky: true
---

# **Prerequisite**

## Homebrew 설치

Mac에서 Multipass 및 K3s를 설치하기 위해 Homebrew가 필요합니다. Homebrew가 설치되지 않았다면 아래 명령어를 실행하여 설치하세요.

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```



## **Multipass 설치**

Multipass는 Canonical에서 제공하는 경량 가상 머신 관리 도구로, Mac에서 간단하게 Ubuntu 인스턴스를 실행할 수 있습니다.

📌 **공식 문서**: [Multipass Documentation](https://multipass.run/docs/installing-on-macos)

```sh
$ brew install --cask multipass
```



## **Multipass를 이용한 K3s 노드 생성**

Multipass를 사용하여 K3s를 구성할 마스터 노드와 워커 노드를 생성합니다.

```sh
$ multipass launch --name k3s-master --cpus 4 --memory 2G --disk 10G 22.04
$ multipass launch --name k3s-worker1 --cpus 4 --memory 4G --disk 10G 22.04
$ multipass launch --name k3s-worker2 --cpus 4 --memory 4G --disk 10G 22.04
```



## **Multipass 기본 명령어**

```sh
$ multipass find        # 사용할 수 있는 이미지 목록 조회
$ multipass list        # 현재 실행 중인 인스턴스 목록 조회
$ multipass delete k3s  # 특정 인스턴스 삭제
```



## Multipass VM 리소스 변경

Multipass에서 VM의 CPU, 메모리, 디스크 크기를 변경할 수 있습니다.

```sh
# VM의 리소스 조회
$ multipass info k3s-master

# 사용 가능한 설정 키 조회
$ multipass get --keys

# 특정 설정 값 조회
$ multipass get local.k3s-master.memory

# VM을 중지 후 메모리 증가
$ multipass stop k3s-master
$ multipass set local.k3s-master.memory=4G
$ multipass start k3s-master

# 변경 사항 확인
$ multipass info k3s-master
```

------



# **Multipass 인스턴스에 K3s 설치**

📌 **공식 문서**: [K3s Quick-Start Guide](https://docs.k3s.io/quick-start)

## **마스터 노드에 K3s 설치**

```sh
# multipass 인스턴스 접속
$ multipass shell k3s-master

# 패키지 업데이트
$ sudo apt-get update && sudo apt-get upgrade -y

# K3s 마스터 노드 설치
$ curl -sfL https://get.k3s.io | sh -s - server \
  --token=SECRET \
  --node-taint CriticalAddonsOnly=true:NoExecute \
  --write-kubeconfig-mode 644
```



## **워커 노드에 K3s 설치**

```sh
# multipass 인스턴스 접속
$ multipass shell k3s-worker1
$ multipass shell k3s-worker2

# 패키지 업데이트
$ sudo apt-get update && sudo apt-get upgrade -y

# 마스터 노드의 IP를 확인하고 변경
$ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent --server https://192.168.64.3:6443 --token SECRET" sh -s -
```



## **K3s 삭제 명령어**

```sh
$ /usr/local/bin/k3s-uninstall.sh         # 마스터 노드에서 삭제
$ /usr/local/bin/k3s-agent-uninstall.sh   # 워커 노드에서 삭제
```

------



# **Mac에서 kubectl로 K3s 클러스터 연결**

📌 **공식 문서**: [Install kubectl on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos)

```sh
$ brew install kubectl
```

클러스터의 kubeconfig를 Mac으로 가져와야 합니다. 마스터 노드에서 `~/.kube/config`를 로컬로 복사한 후, `server` 주소를 수정해야 합니다.

```sh
# 마스터 노드에서 config 파일을 Mac으로 복사
$ multipass transfer k3s-master:/etc/rancher/k3s/k3s.yaml ~/.kube/config

# config 파일 수정
# server: https://127.0.0.1:6443 -> server: https://192.168.64.3:6443
```

------



## **kubectl autocomplete 설정**

📌 **공식 문서**:

- [Bash 자동완성](https://kubernetes.io/ko/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/)
- [Zsh 자동완성](https://kubernetes.io/ko/docs/tasks/tools/included/optional-kubectl-configs-zsh/)

```sh
# Zsh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc

# Bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
```

------

이제 Mac에서 Multipass를 사용하여 K3s 클러스터를 쉽게 구축하고 관리할 수 있습니다. Kubernetes 실습 환경이 필요할 때 활용해 보세요! 🚀