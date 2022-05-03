---
title: System call
date: 2022-05-03 14:00:00 +09:00
categories: [운영체제]
tags: [OS, system call, limited direct execution]
math: false
toc: true
---
무엇을 공부해할지 오래 헤맸다. 주니어인 지금 운영체제, 네트워크, 데이터베이스 이론을 제대로 공부하면 앞으로의 개발이 편해질 것이란 결론이 나왔다. 우선 [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/)를 보며 공부하는 중이다. ([과제 깃](https://github.com/younghch/operating-system-three-easy-pieces-homework))

## What is System Call?

cpu는 두가지 processor mode를 가진다. OS가 컨트롤을 가지는 kernel mode에서는 기능이 제한되지 않지만 프로세스가 실행되는 user mode에서는 보안을 위해 시스템에 영향을 줄 수 있는 기능이 제한된다. user mode에서도 제한된 기능을 사용할 수 있도록 제공되는 기능이 system call이다. 즉 프로세스가 OS에 컨트롤을 넘기고 결과를 돌려받는 것이다.

## Trap, Trap table, Trap handler
trap은 user mode에서 kernel mode로 전환시킨다. kernel mode에서 프로그램의 어떤 코드라도 실행 가능하다면 보안에 치명적일 것이다. 이를 방지하기 위해 trap table이 존재하고 프로세스가 요청할 수 있는 기능이 제한된다. system call이 요청되면 OS는 system-call number에 맞는 trap handler를 trap table내에서 찾아 실행시킨다.

## Procedure of system call
### Brief process
1. trap instruction이 실행되며 user mode에서 kernel mode로 전환된다. 실행되고 있는 프로그램이 return-from-trap 후에 다시 정상적으로 실행될 수 있도록 PC, 레지스터 정보가 kernel stack에 저장된다.
2. syscall이 실행된다.
3. return-from-trap이 실행된다. kernel stack에 넣은 값을 복구하고 user mode에서 PC의 다음 명렬을 수행한다.

### Timeline

| OS(kernel mode)                                                                                                                             | Hardware                                                               | Program(user mode)                               |
|---------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------|--------------------------------------------------|
| **boot**                                                                                                                                    |                                                                        |                                                  |
| Initialize trap table                                                                                                                       |                                                                        |                                                  |
|                                                                                                                                             | remember address of syscall handler                                    |                                                  |
| **run program**                                                                                                                                     |                                                                        |                                                  |
| Create entry for process list <br /> Allocate memory for program<br /> Load program into memory<br /> Setup user stack with argv<br /> Fill kernel stack with reg/PC |                                                                        |                                                  |
| **return from trap**                                                                                                                        |                                                                        |                                                  |
|                                                                                                                                             | restore regs from kernel stack<br /> move to user mode<br /> jump to main          |                                                  |
|                                                                                                                                             |                                                                        | Run main()<br /> execute instructions<br /> Call system call |
|                                                                                                                                             |                                                                        | **trap into OS**                                 |
|                                                                                                                                             | save regs to kernel stack<br /> move to kernel mode<br /> jump to trap handler     |                                                  |
| Handle trap<br /> do work of syscall                                                                                                             |                                                                        |                                                  |
| **return from trap**                                                                                                                        |                                                                        |                                                  |
|                                                                                                                                             | restore regs from kernel stack<br /> move to user mode<br /> jump to PC after trap |                                                  |
|                                                                                                                                             |                                                                        | execute instructions<br /> exit()                      |
|                                                                                                                                             |                                                                        | **trap**                                         |
| Free memory of processes<br /> Remove from process list                                                                                           |                                                                        |                                                  |
