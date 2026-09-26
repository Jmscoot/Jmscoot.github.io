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
![오실로스코프 TX 파형](assets/img/_posts/2026-09-23-bitfield-uart-polling/스크린샷 2026-09-27 012844.png)
