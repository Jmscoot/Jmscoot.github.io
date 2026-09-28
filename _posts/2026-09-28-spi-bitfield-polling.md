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
장치 간 데이터 전송을 클럭을 사용한 동기방식으로 수행하는 통신규약 중 하나이며,<br>
물리계층 인터페이스로 single ended CMOS, LVDS가 주로 사용된다.<br>
UART와 달리 1 to 1 방식뿐만아니라 1 to N 방식도 가능하다.<br>
따라서 Chip Select 핀을 사용한다.<br><br>
또 클럭을 사용한 동기 통신이므로 비동기 통신에 비해 장거리 통신에 유리하다.<br>
하지만 clock을 마스터가 생성하기에, MISO의 경우 왕복지연이 발생한다는 문제점도 있다.<br><br>


사용하는 핀으로 MOSI(master out slave in), MISO(master in slave out), 
clock, CS(chip select)가 사용된다.<br>
![SPI diagram](/assets/img/spi_bitfield_polling/spi_diagram1.png)<br>


#### 송수신 구조<br>
먼저 CS핀이 High->Low로 내려간 동안 슬레이브가 선택되고, 이 구간에서 Clock 엣지에 맞춰,<br>
데이터가 샘플링된다.<br>
![SPI data processing](/assets/img/spi_bitfield_polling/spi_tx_rx.png)<br><br>

또 SPI는 Clock Polarity(CPOL)와 Phase(CPHA) 선택에 따라 전송 방식에 차이가 발생한다.<br>
먼저, Clock Polarity란 clock의 idle 레벨을 정의한다. 다시 말해, 전송하지 않을 때의 클락선을<br>
Low 기준으로 둘 지, High 기준으로 둘 지에 대한 정의이다.<br>
예를 들어서 CPOL=0이면 clock의 idle 레벨은 0으로 정의된다. 따라서 leading edge에서 상승하고<br>
trailing edge에서 하강한다.<br>
CPOL=1이면 clock의 idle 레벨은 1로 정의된다. 따라서 leading edge에서 하강하고<br>
trailing edge에서 상승한다.<br><br>
다음으로 CPHA는 leading edge에서 데이터 값을 샘플링 할 건지, 아니면 trailing edge에서<br>
데이터 값을 샘플링 할 건지, 샘플링 시점을 정의한다.<br>
예를 들어 CPHA=0이면 leading edge에서 데이터 값을 샘플링하고, CPHA=1이면 trailing edge에서<br>
데이터 값을 샘플링한다.<br>
![SPI CPHA, CPOL에 따른 변화](/assets/img/spi_bitfield_polling/cpha_cpol.png)<br><br>


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
