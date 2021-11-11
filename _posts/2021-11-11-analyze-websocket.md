---
layout: post
title: WebSocket 기반 채팅 애플리케이션의 로직 분석
subtitle: Sangmin Lee
categories: Spring , WebSocket , Vue
tags: [Spring , Vue , WebSocket]
---

## 개요

본 글은 Spring과 Vue로 구현한 WebSocket 기반의 채팅 애플리케이션의 동작 로직을 분석하여 시퀀스 다이어그램으로 그린 것을 기록합니다.

*Project : https://github.com/LeeSM0518/spring-websocket-tutorial*

## 시퀀스 다이어그램

### 채팅방 목록 조회

```mermaid
sequenceDiagram
	participant c as Client
	participant w as Web Application
	
	c->>w: GET /chat/room
	w-->>c: room.html
	c->>w: GET /chat/rooms
	w-->>c: 채팅방 목록 (JSON)
```

### 채팅방 생성

```mermaid
sequenceDiagram
	participant c as Client
	participant w as Web Application
	
	Note over c: 채팅방 제목 입력
	c->>w: POST /chat/room <br/> { "name" : "string" }
	w-->>c: 생성된 채팅방 정보
	c->>w: GET /chat/rooms
	w-->>c: 채팅방 목록 정보
```

### 채팅방 입장

```mermaid
sequenceDiagram
	participant c as Client
	participant w as Web Application
	
	Note over c: 대화명 입력
	Note over c: LocalStorage에 wschat.sender(대화명) 저장
	Note over c: LocalStorage에 wschat.roomId(채팅방ID) 저장
	c->>w: GET /chat/room/enter/{roomId}
	w-->>c: roomdetail.html
	Note over c: http://localhost:8080/ws-stomp로 연결되는 웹 소켓 커넥션 객체 생성
	Note over c: 웹 소켓 커넥션 객체 정보로 STOMP 초기화
	c->>w: 웹 소켓 연결 (/ws-stomp)
	c->>w: SUB /sub/chat/room/{wschat.roomId}
	c->>w: PUB /pub/chat/message <br/> { type: 'ENTER', roomId: ${wschat.roomId}, sender: ${wschat.sender} }
	w->>c: PUB /sub/chat/room/{roomId} <br/> { type: 'ENTER', roomId: ${wschat.roomId}, sender: ${wschat.sender} }
```

### 채팅 전송

```mermaid
sequenceDiagram
	participant c as Client
	participant w as Web Application
	
	Note over c: 내용 입력
	c->>w: PUB /pub/chat/message <br/> { type: 'TALK', roomId: ${wschat.roomId}, sender: ${wschat.sender}, message: ${message} }
	w->>c: PUB /sub/chat/room/{roomId} <br/> { type: 'TALK', roomId: ${wschat.roomId}, sender: ${wschat.sender}, message: ${message} }
```

## Reference
https://github.com/codej99/websocket-chat-server