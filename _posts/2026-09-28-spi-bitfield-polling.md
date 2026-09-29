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
STM32에서 MOROROLA 표준을 선택하면 CPOL=0이면 clock의 idle 레벨은 0으로 정의된다. 따라서 leading edge에서 상승하고<br>
trailing edge에서 하강한다.<br>
CPOL=1이면 clock의 idle 레벨은 1로 정의된다. 따라서 leading edge에서 하강하고<br>
trailing edge에서 상승한다.<br><br>
다음으로 CPHA는 leading edge에서 데이터 값을 샘플링 할 건지, 아니면 trailing edge에서<br>
데이터 값을 샘플링 할 건지, 샘플링 시점을 정의한다.<br>
예를 들어 MOTOROLA 표준으로 CPHA=0이면 leading edge에서 데이터 값을 샘플링하고, CPHA=1이면 trailing edge에서<br>
데이터 값을 샘플링한다.<br>
**주의**TI C2000칩의 경우 CPOL은 MOTOROLA 표준과 동일하지만 CPHA는 MOTOROLA 표준과 반대로 CPHA=0이면 trailing edge에서 샘플링하고<br>
CPHA=1이면 leading edge에서 샘플링한다.<br><br><br>

### 간단한 예시<br>
CPOL=1 이므로 idle은 레벨 1, 즉 초기 clock의 위치는 1의 위치. CPHA=0 이므로, leading edge에서 데이터 값을<br>
샘플링한다.<br>
leading edge는 1에서 0으로 하강하는 하강엣지이다.
![SPI CPHA, CPOL에 따른 변화](/assets/img/spi_bitfield_polling/cpha_cpol.png)<br><br>


#### SPI Receiver/Transmitter의 구조<br>
shifter, data register, status register ...etc로 구성되어 있다.<br>
![Diagram](/assets/img/spi_bitfield_polling/structure.png)<br><br>

Master-Slave간 데이터 전송은 동시에 발생한다. 예를 들어서 Master에서 Slave로 데이터를 수신할 때 Master가<br>
보낼만한 유의미한 데이터가 없더라도, Master가 데이터 수신을 하면, 자동으로 shift reg의 data가 Slave로 보내지게 된다..<br> 
TI F28379D TRM의 SPI chapter에서도 해당 내용이 명시돼있다.<br>
![ti의 SPI Master-Slave relation](/assets/img/spi_bitfield_polling/ti_tx_rx_spi_op.png)
따라서 SPI에서는 데이터 송수신이 동시에 발생한다.<br><br>


좀 더 간략화한 구조는 아래와 같다...<br>
![SPI data 전송은 tx/rx가 동시에 발생해야된다.](/assets/img/spi_bitfield_polling/ti_master_slave_operation.png)<br><br><br><br>

#### 오버샘플링
SPI는 동기식(synchronous) 통신이라 1비트당 1번, 클럭 엣지에서 한 번만 샘플링한다.<br>
UART가 oversampling을 하는 이유는 비동기식이라 수신 측이 송신 측의 클럭을 모르기 때문에<br>
수신 측이 스스로 해결해야 하는 문제가 두 가지 있습니다.<br><br>

첫번째, 비트 중앙 찾기: start bit의 하강 엣지를 감지한 뒤, 자기 클럭으로 시간을 세어 각 비트의 가운데 지점을 추정해야 한다.<br>
16배 oversampling이면 한 비트를 16조각으로 나눠 보면서 8번째 조각 근처를 샘플링한다.<br><br>

두번째, 클럭 오차 흡수와 노이즈 판정: 양쪽 baud rate가 조금씩 다르므로, 한 프레임 동안 누적되는 오차를 견딜 여유가 필요하다.<br>
반면에 SPI는 클럭으로 동기화하기에 오버샘플링이 불필요하다.<br><br><br>

## ti SPI 요약
![ti SPI 요약](/assets/img/spi_bitfield_polling/ti_spi.png)<br><br><br>
ti spi master의 코드를 보면 다음과 같다. 먼저 master의 SPIDAT shift reg의 데이터가 SPISIMO를 통해서<br>
MSB부터 slave로 shifted 송신되면, slave측에서는 자동으로 SPISOMI를 통해서 LSB로 값이 shifted 수신된다.<br>
그러면 slave의 SPIDAT shift reg에 저장된 값들이 SPIRXBUF로 이동되고, INT_FLAG가 1로 set된다.<br><br><br>

그리고 CPOL=0, CPHA=0이면 clock idle state = 0, 샘플링 지점은 leading edge인 MOTOROLA 표준과 다르게<br>
CPOL=0, CPHA=0이면 clock idle state = 0, 샘플링 지점은 trailing edge이다.<br>
![ti SPI CPOL, CPHA example](/assets/img/spi_bitfield_polling/cpha_cpol_order.png)<br><br>
![ti SPI master에서 데이터 송수신 절차](/assets/img/spi_bitfield_polling/ti_spi_code.png)<br><br>

아래는 saleae 로직 애널라이저 SPI세팅에서 CPOL=0, CPHA=trailing edge로 선택했을 때 정상적으로 출력됨을 확인 가능하다.<br>
![saleae logic](/assets/img/spi_bitfield_polling/saleae.png)<br><br><br>
## ti SPI bitfield flow
1. clock configuration<br>
1.1 XTAL ON<br>
1.2 XTAL을 PLL SRC로 set<br>
1.3 PLL을 sysclk으로 set<br>
   
2. GPIO configuration<br>
2.1 GPIO 소유권 선택<br>
2.2 GPIO mux(mode selection)<br>
2.3 GPIO IN/OUT<br>
2.4 GPIO pull-up/pull down selection<br>

3. SPI configuration<br>
3.1 데이터 비트 8비트 설정<br>
3.2 MODE 0 (CPOL:0, CPHA:0)<br>
3.3 master_slave=1 (master mode로)<br>
3.4 BRR 레지스터 값 설정<br>
3.5 FIFO 사용 유무 설정<br>
3.6 SWRESET=1로 SPI start<br>


## source code<br>
https://github.com/Jmscoot/ti_spi_master_bitfield.git
