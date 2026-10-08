---
layout: post
title: "TI Clock Source에 대해"
date: 2026-10-08 00:55:00 +0900
categories: [임베디드, TI]
tags: [TI, XTAL, Clock Tree]
---


## TI Clock<br>
Clock은 곧 칩 세계에서의 시간이다.<br>
명령어들은 파이프라인 단계(Fetch → Decode → Execute)를 거쳐 처리되고,<br>
(아래는 TI C28x아키텍처의 파이프라인 단계로, 실제로는 이렇게 더 세분화 돼있다.)<br>
![ti pipeline](/assets/img/ti_clock/c28x_pipeline.png)<br><br>
매 클럭에 맞춰 모든 단계가 동시에 한 행씩 진행한다.<br>
![ti pipe](/assets/img/ti_clock/pipeline_examp.png)<br>
다만 다중 사이클 연산, 메모리 대기, 분기 등으로 stall이나 flush가 생기면 한 단계에 여러 클럭이 걸리거나 사이클이 낭비되기도 한다.<br>

#### Clock Tree
아래는 F28388D의 클럭 트리이다.
![clock tree](/assets/img/ti_clock/clock_tree.png)<br><br>

#### Clock source
C2000 F28388D기준으로 SOC칩 내부 클럭 2개(INTOSC1, INTOSC2)가 존재한다.<br><br>
칩 외부에 클럭을 클럭 소스로 사용이 가능하다. XTAL이라고도 불리는데 그 종류로는<br>
single-ended 3.3V XTAL, Crystal XTAL, Resonator XTAL이다.<br><br>
클럭은 클럭 소스에서 분주를 거쳐서 CPU나 Peripheral들에게 공급이 된다.<br>
OSCCLK은 칩의 모든 클럭이 출발하는 "원천 클럭" 이다. 여러 clock source 중 하나를 골라 OSCCLK으로 삼고,<br>
여기서 PLL을 거쳐 CPU 클럭과 주변장치 클럭이 만들어진다. 그래서 마스터 레퍼런스라고 부른다.<br><br>

![sysclock](/assets/img/ti_clock/sysclk.png)<br>
sysclock을 보면 (SYSCTL_OSCSRC_XTAL_SE | SYSCTL_IMULT(32) | SYSCTL_REFDIV(2) | SYSCTL_ODIV(2) | SYSCTL_SYSDIV(1) | <br>SYSCTL_PLL_ENABLE | SYSCTL_DCC_BASE_1)<br>
이렇게 세팅되어 있다. 여기서 SYSCTL_OSCSRC_XTAL_SE=25Mhz이므로... 25Mhz*32/2/2/1=200Mhz가 SYSCLK으로 사용됨을 확인할 수 있다.<br>
DEVICE_SETCLOCK_CFG은 칩의 register를 set해서, 실제 200Mhz자 sysclock으로 나오게 하기 위함이다.
