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
장치간 데이터 전송을 비동기방식으로 수행하는 통신규약 중 하나이며,
물리계층 인터페이스로 RS-232,422,485,TTL/CMOS가 주로 사용된다.<br>
비동기 방식의 특징으로 장치간 클럭 동기화가 돼있지 않고, 대신에 
Baud rate를 동일하게 설정한다.<br><br>
선로가 길어질수록 케이블의 RC값(시정수)이 커져서, 엣지(과도구간)에서의 상승/하강 시간이
늘어나고, 그 결과 start 엣지를 인식하는 시점이 밀리게 되고 그 뒤의 비트들도 전부 밀리게 된다.
클럭이 아닌, Baud rate로 동기화하는 UART특성상 장거리 통신에 있어서 부정확하다는 단점이 있다.<br>
![UART tx/rx diagram](/assets/img/ti_uart_bitfield_polling/uart_diagram3.png)
<br>UART통신은 Half Duplex방식과 Full Duplex 방식 둘 중 선택이 가능하다.<br><br><br>

#### Frame 구조<br>
1 frame은 다음과 같이 구성되어 있다.<br>
![UART 1 frame](/assets/img/ti_uart_bitfield_polling/frame.png)<br>
1 start bit+DATA BITS+1 stop bit로 구성되어 있다.<br>
중간에 DATA BITS의 BIT갯수는 경우에 따라 달라진다.<br>
![UART 1 frame](/assets/img/ti_uart_bitfield_polling/frame_type.png)<br><br><br>
Baud rate는 이 frame에서 비트의 전송속력을 의미한다.<br>
그리고 frame을 수신할 때 1bit를 그대로 받는게 아니라, 1bit를 8-oversampling 혹은 16-oversampling을 해서 수신한다.<br>
![UART sampling rate](/assets/img/ti_uart_bitfield_polling/bclk.png)<br><br><br>

#### UART Receiver/Transmitter의 구조<br>
![Diagram](/assets/img/ti_uart_bitfield_polling/block_diagram2.png)<br><br>

좀 더 간략화한 구조는 아래와 같다...<br>
![Diagram2](/assets/img/ti_uart_bitfield_polling/parallel.png)<br>
serial data송신(1bit씩)->송신 FIFO에 1bit씩 누적->송신 FIFO에 1Byte 데이터가 shift register에 1Byte 단위로 송신<br>
->shift register의 값 1Byte를 Parallel to Serial(1bit 단위 송신)<br>
->shift register로 1bit값이 누적되어 1Byte 저장->Serial to Parallel로 shift register의 1Byte 데이터가 Receiver FIFO로 송신<br>
->Receiver FIFO에서 1bit 단위로 값을 꺼내옴<br><br>

CLK이 Baud Generator로 input되고, output으로 BCLK을 만든다.
#### example 1<br>
16-oversampling, Baud rate 115200[bit/sec]라고 한다면 BCLK=16[cycle/bit]*115200[bit/sec]=대략 1.84[Mega cycle/sec]=1.84[Mhz]<br>

## ti UART bitfield flow
1. clock configuration<br>
1.1 XTAL ON<br>
1.2 XTAL을 PLL SRC로 set<br>
1.3 PLL을 sysclk으로 set<br>
   
2. GPIO configuration<br>
2.1 GPIO 소유권 선택<br>
2.2 GPIO mux(mode selection)<br>
2.3 GPIO IN/OUT<br>
2.4 GPIO pull-up/pull down selection<br>

3. SCI configuration<br>
3.1 SCI clock config<br>
3.2 SCI BAUD config<br>


## source code<br>
https://github.com/Jmscoot/ti_bitfield_uart_polling.git
