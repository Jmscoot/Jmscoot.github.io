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
spi를 활성화하면 주석으로 처리된 부분이 활성화됨을 확인가능하다.. 또 왼쪽 날개에 Drivers-STM32F4xx_HAL_Driver-src디렉토리에 stm32f4xx_hal_spi.c도 추가됨을 확인가능하다.<br>
![spi](/assets/img/hal/spi2.png)<br><br>

