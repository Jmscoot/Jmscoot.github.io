---
layout: post
title: "SPI Polling 방식 BITFIELD 구현"
date: 2026-09-28 00:55:00 +0900
categories: [임베디드, TI]
tags: [TI, STM32, SPI, BITFIELD]
---

## 환경
칩: F28379D, F446RE
IDE: CCS 21.0.1


## SPI 프로토콜이란
장치 간 데이터 전송을 클럭을 사용한 동기방식으로 수행하는 통신규약 중 하나이며,
물리계층 인터페이스로 single ended CMOS, LVDS가 주로 사용된다.
UART와 달리 1 to 1 방식뿐만아니라 1 to N 방식도 가능하다.
따라서 Chip Select 핀을 사용한다.
또 클럭을 사용한 동기 통신이므로 비동기 통신에 비해 장거리 통신에 유리하다.
하지만 clock을 마스터가 생성하기에, MISO의 경우 왕복지연이 발생한다는 문제점도
있다.<br><br>

사용하는 핀으로 MOSI(master out slave in), MISO(master in slave out), 
clock, CS(chip select)가 사용된다.<br>



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
3.3 SCI data bit config<br>
3.4 SCI TX, RX 활성화<br>
3.5 SCI SWRESET<br>


## source code<br>
https://github.com/Jmscoot/ti_bitfield_uart_polling.git
