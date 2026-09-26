---
layout: post
title: "UART Polling 방식 BITFIELD 구현"
date: 2026-09-21 18:55:00 +0900
categories: [임베디드, TI]
tags: [TI, UART, BITFIELD]
---

## 환경
칩: F28379D
IDE: CCS 21.0.1


## UART 프로토콜이란
비동기 통신규약 중 하나이며 물리계층 인터페이스로 RS-232,422,485,TTL/CMOS가 주로 사용된다.
1 frame은 다음과 같이 구성되어 있다.
![UART 1 frame](assets/img/post/frame.png)
Baud rate는 이 frame에서 비트의 전송속력을 의미한다.

그리고 frame을 수신할 때 1bit를 그대로 받는게 아니라, 1bit를 8-oversampling 혹은 16-oversampling을 해서 수신한다.
![UART sampling rate](assets/img/post/bclk.png)

## UART Receiver/Transmitter의 구조
![Diagram](assets/img/post/block_diagram2.png)
좀 더 간략화한 구조는 아래와 같다...
![Diagram2](assets/img/post/block_diagram1.png)
UART HW로 CLK이 Baud Generator로 input되고, output으로 BCLK을 만든다.

## example 1)
16-oversampling, Baud rate 115200[bit/sec]라고 한다면 BCLK=16[cycle/bit]*115200[bit/sec]=대략 1.84[Mega cycle/sec]=1.84[Mhz]
