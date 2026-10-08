---
layout: post
title: "TI Clock Source에 대해"
date: 2026-10-08 00:55:00 +0900
categories: [임베디드, TI]
tags: [TI, EXT, Clock Tree]
---


## TI Clock<br>
Clock은 곧 칩 세계에서의 시간이다.<br>
명령어들은 파이프라인 단계(Fetch → Decode → Execute)를 거쳐 처리되고,<br>
(아래는 TI C28x아키텍처의 파이프라인 단계로, 실제로는 이렇게 더 세분화 돼있다.)<br>
![ti pipeline](/assets/img/ti_clock/c28x_pipeline.png)<br><br>
매 클럭에 맞춰 모든 단계가 동시에 한 칸씩 진행한다.<br>
![ti pipe](/assets/img/ti_clock/pipeline_examp.png)<br><br>
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
여기서 PLL을 거쳐 CPU 클럭과 주변장치 클럭이 만들어진다. 그래서 마스터 레퍼런스라고 부른다.<br>

#### 기본사양
| 항목 | float | double |
|---|---|---|
| IEEE 754 명칭 | Single precision (binary32) | Double precision (binary64) |
| 크기 | 32비트 (4바이트) | 64비트 (8바이트) |
| 비트 구성 (부호/지수/가수) | 1 / 8 / 23 | 1 / 11 / 52 |
| 지수 바이어스 | 127 | 1023 |
| 유효 정밀도 (hidden bit 포함) | 24비트 | 53비트 |
| 십진 유효숫자 | 약 7자리 | 약 15~16자리 |
| 최소 정규수 (`FLT_MIN` / `DBL_MIN`) | ≈ 1.18 × 10⁻³⁸ | ≈ 2.23 × 10⁻³⁰⁸ |
| 최대값 (`FLT_MAX` / `DBL_MAX`) | ≈ 3.40 × 10³⁸ | ≈ 1.80 × 10³⁰⁸ |
| 머신 엡실론 (`FLT_EPSILON` / `DBL_EPSILON`) | ≈ 1.19 × 10⁻⁷ | ≈ 2.22 × 10⁻¹⁶ |
