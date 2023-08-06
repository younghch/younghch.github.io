---
title: 어디서 토큰을 인증할 것인가?
date: 2023-08-06 18:00:00 +09:00
categories: [인증]
tags: [API Gateway, Ingresss Controller, AWS, Kubernetes]
math: false
toc: true
---
## 문제 상황

JWT을 사용하면 인증서버의 퍼블릭 키로 토큰을 검증 할 수 있다. 인증 코드 구현은 어렵지 않다. 하지만 모든 서버에 이 기능이 각각 추가되면 코드의 응집도를 떨어트리고 도메인적으로도 인증은 별개로 분리되는게 맞다고 생각했다. 인증서버와의 불필요한 통신이 없으면서 인증을 한곳에 응집시킬 방법은 어떤게 있을까?

## 환경

- AWS 내 EKS사용 없이 직접 구성한 K8s 클러스터. 
- 현재 Spring Boot 서버만 있지만 다른 종류 서버가 들어올 수 있음.
- 프런트는 Vite에 static website를 배포

## 방법

### AWS API Gateway + Lambda authorizer

API Gateway를 사용하면서 AWS Lambda를 사용해 토큰을 검증하고 인증된 트래픽만 벡앤드로 보낼 수 있다.
<img src='https://docs.aws.amazon.com/images/apigateway/latest/developerguide/images/custom-auth-workflow.png'>
aws 프리티어가 100만 requests와 400,000 GB-seconds의 compute time을 지원하고 JWT 검증은 필요메모리와 실행시간이 적으므로 비용걱정은 없다고 본다.

#### 장점

- 한곳에서 인증이 관리된다.
- BE에 트래픽이 몰리는 것을 방지할 수 있다.
    - 캐싱 지원 
    - 트래픽 throttling
    - lambda로 검증된 요청만 필터링
- MSA에 적합하다.
- 쉽게 요청을 모니터링 할 수 있다.
- aws fully managed service로 다운될 걱정이 적다.
#### 필요성에 대한 의문
- k8s Ingress를 사용하지 않는다면 서비스마다 loadbalancer 인스턴스가 필요해 추가 비용이 든다. k8s Ingress를 사용한다면 API Gateway의 필요성이 적은것 같다.
- 프런트 라우팅은 Route53에서도 할 수 있다.
### Kubenertes Ingress Controller
[Nginx_JWT_Validation_Policy](https://www.nginx.com/blog/announcing-nginx-ingress-controller-release-1-9-0/#policies)

### Multi module