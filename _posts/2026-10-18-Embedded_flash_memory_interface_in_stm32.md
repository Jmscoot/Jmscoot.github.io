---
layout: post
title: "stm32에서 flash mem read time과 CPU clock freq간 관계"
date: 2026-10-18 00:55:00 +0900
categories: [임베디드, STM32]
tags: [Flash memory, STM32]
---

## Flash memory 읽기 속력의 한계
플래시 메모리의 R/W작업은 overload가 상당하다.<br>
플래시 메모리 READ 작업은 실행되는 프로그램에 따라 32비트 명령어 4개 또는 16비트 명령어 8개에 해당하는 "128비트" 고정으로 데이터를 읽는다.<br>
앞서 읽어 들인 명령어 라인을 실행하는 데 최소 4개의 CPU 사이클이(4CLK) 필요하다.<br>
![none fetch](/assets/img/flashmem/nonprefetch.png)<br><br>


#### Wait State 정의
기본적으로 메모리에서 데이타 R/W는 느리기에... flash작업에서 CPU가 다른 작업을 못하고 flash를 기다리며 허비되는 클럭을 말한다.<br>
![wait state](/assets/img/flashmem/waitstate.png)<br>
그리고 전압이 낮을수록 flash를 read하는 속력이 느려지기 때문에 WS 1개당 시간이 더 길어지게(늘어지게)된다.<br>
더 빨리 flash를 read하려면 더 높은 전압을 사용 CPU의 주파수를 높여야된다.<br>
![overdrive](/assets/img/flashmem/overdrive.png)<br><br>


#### Prefetch 기능
앞선 느린 Flash의 R/W 단점을 극복하기 위해.. I-Code 버스에서의 프리페치(prefetch) 기능을 사용하면, 플래시 인터페이스가 이전에 읽은 128비트를 CPU에 주는 동안,<br>
다음 128비트를 플래시에서 미리 읽는다. <br>

그러면 Flash R/W 속력은 여전히 느리지만... 미리 flash에서 데이터를 읽어 바로바로 CPU에서 사용이 가능하다.<br>
이 프리페치 기능은 FLASH_ACR 레지스터의 PRFTEN 비트를 설정하여 활성화할 수 있다..<br>
![pipeline](/assets/img/flashmem/withprefetch.png)<br>
> 위의 사진을 보면 플래시에서 기계어로된 펌웨어에서 128비트 단위로 명령(instruction) 1,2,3,4를 읽어서(read)<br>
이를 cpu에 넘기기(gives instruction 1,2,3,4)까지 시간이 걸린다. 하지만 inst 1,2,3,4를 gives하면서<br>
prefetch를 통해, read ins 5,6,7,8을 함으로써, 다음 inst gives까지 시간 단축이 가능하다.<br>

