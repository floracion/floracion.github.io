---
layout: post
title: WebSocket 기반 채팅 애플리케이션의 로직 분석
subtitle: Sangmin Lee
categories: Spring WebSocket Vue
tags: [Spring , Vue , WebSocket]
---

## 개요

본 글은 Spring과 Vue로 구현한 WebSocket 기반의 채팅 애플리케이션의 동작 로직을 분석하여 시퀀스 다이어그램으로 그린 것을 기록합니다.

*Project : [https://github.com/LeeSM0518/spring-websocket-tutorial](https://github.com/LeeSM0518/spring-websocket-tutorial)*

## 시퀀스 다이어그램

### 채팅방 목록 조회

![image](https://user-images.githubusercontent.com/43431081/141251540-afaa7a39-b4bd-41c4-b0c9-b3871f9e7c6b.png)

### 채팅방 생성

![image](https://user-images.githubusercontent.com/43431081/141251565-abeb66ba-4229-4b58-928e-f1ffc18cf387.png)

### 채팅방 입장

![image](https://user-images.githubusercontent.com/43431081/141251601-75311c66-4178-4a22-87fe-7935bcbf12b0.png)

### 채팅 전송

![image](https://user-images.githubusercontent.com/43431081/141251644-3f428a86-bf22-4dcc-8faf-e8c9abb34e46.png)

## Reference
[https://github.com/codej99/websocket-chat-server](https://github.com/codej99/websocket-chat-server)