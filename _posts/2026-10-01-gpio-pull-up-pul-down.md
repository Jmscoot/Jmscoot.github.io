---
layout: post
title: "GPIO PULL UP/PULL DOWM에 대해"
date: 2026-10-01 00:55:00 +0900
categories: [임베디드, STM32]
tags: [STM32, GPIO PULL UP, PULL DOWN]
---


## PULL UP/PULL DOWN<br>

#### PULL UP/PULL DOWN 개념<br>
GPIO의 default state를 HIGH state로 둘 지, LOW state로 둘 지에 대한 설정이다.<br>
PULL UP/DOWN 설정을 하지 않으면 HIGH-Z상태에 놓이고, GPIO핀이 floating해서 값이 랜덤하게 0 or 1로 바뀐다.<br>

#### STM32에서의 GPIO 회로 구성<br>
![stm32 gpio](/assets/img/gpio_pull_up_dowm/stm_output.png)<br>
사진을 보면 PULL UP일 때는 윗쪽 switch가 on되어 default로 항상 VCC에 걸리게 된다.<br>
예를 들어서 해당 GPIO핀이 입력 핀이라고 하면, 입력 신호에 0이 들어오면 VCC가 GND와 단락이 되므로<br>
여전히 GPIO 입력은 0이 유지가 된다. 반면 입력 신호에 1이 들어오면 PULL UP의 VCC가 VCC와 단락이 되므로<br>
여전히 GPIO 입력은 1이 유지가 된다. 그리고 별도의 신호 입력이 없을 경우 항상 VCC로 땡겨주기에, VCC를 유지한다.<br><br>

PULL DOWN일 때도 마찬가지로 입력 신호에 0이 들어오면 GND가 GND와 단락이 되므로 입력 신호는 0을 유지하고,<br>
입력 신호에 1이 들어오면 
