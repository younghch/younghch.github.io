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

## 방법

### API Gateway + Lambda authorizer

#### 왜 API Gateway가 필요한가?


#### Lambda Authroizer

https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html
<img src= >

#### 장단점 정리
**Pros**

**Cons**

### Kubenertes Ingress Controller

### Multi module