---
title: Context Switch
date: 2022-05-03 18:00:00 +09:00
categories: [운영체제]
tags: [OS, context switch, limited direct execution]
math: false
toc: true
---
프로세스가 cpu에서 실행되고 있을 때, OS는 실행되지 않는다. 그러면 어떻게 OS가 실행중인 프로세스를 다른 프로세스로 전환할까?

## Timer interrupt
프로세스에서 system call을 호출할때까지 기다려 OS가 컨트롤을 되찾을 수도 있지만, 프로그램이 무한 루프에 빠진다면 이를 되찾을 방법은 없다. 이에 하드웨어에서 타이머를 설정하고 특정 주기로 인터럽트를 발생시켜 OS를 호출한다.

## Context switch timeline

| OS(kernel mode)                                                                                                     | Hardware                                                                                   | Program(user mode) |
|---------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|--------------------|
| **boot**                                                                                                            |                                                                                            |                    |
| Initialize trap table                                                                                               |                                                                                            |                    |
|                                                                                                                     | remember address of syscall handler <br /> remember address of timer handler              |                    |
| start interrupt timer                                                                                               |                                                                                            |                    |
|                                                                                                                     | start timer <br /> interrupt CPU in X ms                                                   |                    |
| **run**                                                                                                             |                                                                                            |                    |
|                                                                                                                     |                                                                                            | Process A          |
|                                                                                                                     | **timer interrupt**                                                                        |                    |
|                                                                                                                     | save registers(A) to kernel stack(A)<br /> move to kernel mode <br /> jump to trap handler |                    |
| Handle the trap                                                                                                     |                                                                                            |                    |
| Call switch() routine  <br /> save regs(A) into PCB(A)<br /> restore regs(B) from PCB(B)<br /> switch to k-stack(B) |                                                                                            |                    |
| **return-from-trap (into B)**                                                                                       |                                                                                            |                    |
|                                                                                                                     | restore regs(B) from kernel stack(B)<br /> move to user mode<br /> jump to B's PC          |                    |
|                                                                                                                     |                                                                                            | Process B          |

## *Why do we store/resotre registers to both kernel stack and PCB?

모든 레지스터의 값이 PCB에 저장되는 것이 아니다. PCB의 esp에는 kernel stack의 주소만 저장된다. 이를 이용해 kernel stack에서 레지스터를 복구한다.
<img src="https://i.stack.imgur.com/zVVIC.png"/>