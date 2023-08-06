---
title: 어디서 토큰을 인증할 것인가?
date: 2023-08-06 17:30:00 +09:00
categories: [인증]
tags: [API Gateway, Ingresss Controller, AWS, Kubernetes]
math: false
toc: true
---
## 문제 상황

JWT을 사용하면 인증서버의 퍼블릭 키로 토큰을 검증 할 수 있다. 인증 코드 구현은 어렵지 않다. 하지만 모든 서버에 이 기능이 각각 추가되면 코드의 응집도를 떨어트린다. 인증서버와의 불필요한 통신이 없으면서 인증을 한곳에 응집시킬 방법은 어떤게 있을까?

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

- BE에 트래픽이 몰리는 것을 방지할 수 있다.
    - 캐싱 지원 
    - 트래픽 throttling
    - lambda로 검증된 요청만 필터링
- MSA에 적합하다.
- 쉽게 요청을 모니터링 할 수 있다.
- aws fully managed service로 다운될 걱정이 적다.
#### 필요성에 대한 의문
- K8s Ingress를 사용하지 않는다면 서비스마다 loadbalancer 인스턴스가 필요해 추가 비용이 든다. K8s Ingress를 사용한다면 Gateway를 두개 사용하는 것과 같아 API Gateway의 필요성이 적은것 같다.
- 프런트 서비스로의 라우팅은 Route53에서도 할 수 있다.
### Kubernetes Ingress Controller

Nginx Kubernetes Ingress Controller는 JWT validation을 지원한다.([Nginx_JWT_Validation_Policy](https://www.nginx.com/blog/announcing-nginx-ingress-controller-release-1-9-0/#policies))

#### 장점

- BE에 트래픽이 몰리는 것을 방지할 수 있다.
    - [캐싱_지원](https://docs.nginx.com/nginx-controller/app-delivery/about-caching/)
    - [트래픽_rate_limit](https://docs.nginx.com/nginx-ingress-controller/configuration/policy-resource/#ratelimit)
    - 검증된 요청만 필터링
- MSA에 적합하다.

#### 단점
- 모든 기능을 직접 설정해야한다.
- single point of failure => controller가 다운되면 전체 서비스가 다운된다.


### Multi module

인증관련 코드를 모듈화하고 공통으로 사용할 수 있다. AOP로 만들어 놓고 인증이 필요한 경우 간단히 어노테이션을 추가하면 된다.

#### 장점

- 구현이 간단하다.
- 인증이 필요한 곳에 어노테이션만 추가하면 되서 직관적이다.

#### 단점
- 스프링 이외의 서버가 들어왔을 때 인증이 여러 곳에서 관리된다.