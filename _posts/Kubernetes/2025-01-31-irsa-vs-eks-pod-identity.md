---
title: "[EKS] IRSA vs EKS Pod Identity"
excerpt: "IRSA와 EKS Pod Identity 기능 비교"
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



## **EKS Pod Identity와 IRSA 비교**

### 기능 측면

| 기능                   | EKS Pod Identity                                             | IRSA                                                         | **요약 설명**                                                |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **역할 확장성**        | Amazon EKS 서비스 보안 주체 `pods.eks.amazonaws.com`과 신뢰를 구축하면, 이후 역할 신뢰 정책을 업데이트할 필요 없음 | 새 클러스터에서 역할을 사용할 때마다 OIDC 공급자 엔드포인트로 IAM 역할의 신뢰 정책을 업데이트해야 함 | - Pod Identity: `Node 단위` 인증<br>- IRSA: `Pod 단위` 인증  |
| **클러스터 확장성**    | IAM OIDC 공급자 설정이 필요 없음                             | EKS 클러스터별로 OIDC 공급자 생성 필요 (AWS 계정당 최대 100개 제한) | - Pod Identity: OIDC `미사용`<br>- IRSA: OIDC `사용` (EKS 클러스터 100개 이상 불가) |
| **역할 확장성**        | 신뢰 정책에서 IAM 역할과 서비스 계정 간의 신뢰 관계 정의 필요 없음 | 신뢰 정책에서 IAM 역할과 서비스 계정 간 신뢰 관계를 정의해야 함 (기본 신뢰 정책 크기 2048, 보통 4~8개 신뢰 관계 가능) | Pod Identity는 고정된 값 사용으로 설정이 단순                |
| **역할 재사용 가능성** | IAM STS 세션 태그(클러스터, 네임스페이스, 서비스 계정 등) 포함 가능 → 단일 IAM 역할을 여러 서비스 계정에서 공유 가능 (ABAC 지원) | STS 세션 태그 지원 안됨 → 포드 단위로 IAM 역할을 세밀하게 제어 불가 | - Pod Identity: `Role 세션 태그`로 `IAM role 공유` 가능      |
| **지원 환경**          | Amazon EKS 전용                                              | Amazon EKS, EKS Anywhere, Red Hat OpenShift, EC2 기반 자체 관리형 Kubernetes 등 | 큰 차이 없음                                                 |
| **지원 EKS 버전**      | Kubernetes `1.24` 이상                                       | 모든 EKS 클러스터 버전                                       | 큰 차이 없음                                                 |



### Implementation 측면

| **단계**                               | **IRSA**                           | **EKS Pod Identity** | **비고**                                                    |
| -------------------------------------- | ---------------------------------- | -------------------- | ----------------------------------------------------------- |
| **IAM Role 생성**                      | ✅                                  | ✅                    | 동일                                                        |
| **OIDC Provider 생성**                 | 필요                               | 불필요 (OIDC 미사용) | - `OIDC Provider 생성 (1회)` vs `Agent 설치`                |
| **Agent 설치**                         | 불필요                             | 필요                 |                                                             |
| **Trust relationship 추가**            | ✅ (OIDC Provider별 다름)           | ✅ (고정값)           | - Pod Identity가 더 편리 (클러스터별 신뢰 정책 변경 불필요) |
| **Service Account 생성 및 Pod에 할당** | N (default 사용)                   | N (default 사용)     | 동일                                                        |
| **IAM role association**               | 서비스 계정에 annotation 추가 방식 | AWS API 호출 방식    | - `Apply k8s yaml` vs `Call AWS API` (큰 차이 없음)         |



## **References**

- **Pod Identity:**
  - [EKS Pod Identity 공식 문서](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/pod-identities.html)
  - [Amazon EKS Pod Identity 블로그](https://aws.amazon.com/ko/blogs/aws/amazon-eks-pod-identity-simplifies-iam-permissions-for-applications-on-amazon-eks-clusters/)

- **IRSA:**
  - [서비스 계정에 대한 IAM 역할 - 공식 문서](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/iam-roles-for-service-accounts.html)
