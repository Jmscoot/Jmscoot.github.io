---
layout: post
title: "HAL LIB"
date: 2026-10-18 10:55:00 +0900
categories: [임베디드, 빌드]
tags: [라이브러리, STM32]
---

# HAL LIB
HAL 라이브러리는 직접적인 하드웨어 구조나 레지스터에 대한 이해/접근 없이 직관적으로 유저가 해당 기능을 사용하게 만든 라이브러리이다.<br>

## STM32 HAL 동작
cube mx에서 SPI 패리패럴을 활성화하면 SPI에 대한 HAL 라이브러리가 활성화된다.<br>
![spi hal](/assets/img/hal/hal_spi_en.png)<br>
stm32f4xx_hal_conf.h에 보면 /* #define HAL_SPI_MODULE_ENABLED */으로 주석처리 돼 있음을 확인할 수 있다.<br><br>

![spi](/assets/img/hal/spi_en.png)<br>
spi를 활성화하면 주석으로 처리된 부분이 활성화됨을 확인가능하다.. 또 왼쪽 날개에 Drivers-STM32F4xx_HAL_Driver-src디렉토리에<br>
stm32f4xx_hal_spi.c와 stm32f4xx_ll_spi.h도 추가됨을 확인가능하다.<br>
![spi](/assets/img/hal/spi2.png)<br><br>
그리고 stm32f4xx_hal_conf.h에서 #include "stm32f4xx_hal_spi.h"가 활성화된다.<br>
![include](/assets/img/hal/include.png)<br>
main.h에 stm32f4xx_hal.h를 include하며, stm32f4xx_hal.h에서는 stm32f4xx_hal_conf.h를 include한다.<br>
stm32f4xx_hal_msp.c에는 SPI로 쓸 gpio에 대한 pin세팅 작업을 하는 HAL_SPI_MspInit API가 자동 생성된다.<br>
![msp](/assets/img/hal/msp_spi.png)<br><br>
그리고 main.c에는 MX_SPI1_Init이라는 API가 정의 및 main함수 초기에 호출이된다.<br>
MX_SPI1_Init에서는 spi에 대한 configuration(SPI채널, SPI mode, 방향, 데이터크기 등등)이 이뤄진다.<br><br>

main.c에 MX_SPI1_Init이 호출, MX_SPI1_Init에서 HAL_SPI_Init이 호출, HAL_SPI_Init에서 HAL_SPI_MspInit이 호출됨을 확인 가능하다.<br><br>

정리하면 spi에 쓸 gpio clock config, spi로 쓸 gpio 핀 config, spi clock config, spi기본기능 config가 이뤄진다.<br>
따라서 register나 LL단위로 짜고 싶으면 위의 것들만 지켜서 API만들어 쓰면 된다...<br><br>

STM32 펌웨어 계층구조는 아래와 같다.<br>
![stm32 structure](/assets/img/hal/hal_structure.png)<br>
CMSIS헤더도 없는 C수준에서 low코드를 짤 때는 주소지에 대한 #define도 없으니 직접 주소지도 데이터시트 보면서<br>
*(volatile uint32_t *)0x40023844와 같이 0x40023844주소지의 값을 바꿔야된다...(RCC_APB2ENR레지스터의 주소이다.)<br>
근데 CMSIS헤더라는 편한 헤더를 사용하면 RCC->APB2ENR와 같이 쓸 수 있다.<br>

본인의 경우 PWM은 register나 LL단위로 API를 짜는게 BLDC모터 컨트롤에 더 유용한거 같은데.. UART같은 경우 굳이 register나 LL단위로 다시<br>
코드를 짜는 것이 번거로워 CUBE MX에서 hal을 주로 사용하는데.. 이런 경우 hal과 reg, LL을 섞어 쓰게 되고, 코드가 지저분해지고<br>
설정들이 헷갈리게 되어 이번 기회에 hal에 대해 정리하게 됨.

