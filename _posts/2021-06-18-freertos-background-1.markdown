---
layout: post
title: FreeRTOS Background Part-1
date: 08-06-2021 18:00:00 +0200
description: FreeRTOS Background . # Add post description (optional)
img: FreeRTOS_Logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [RTOS]
---

FreeRTOS Background Part-1
## 1. RTOS and GPOS comparision:  

|                                                       | RTOS                                     | PGOS                                                          |
|-------------------------------------------------------|------------------------------------------|---------------------------------------------------------------|
| Goal                                                  | Time Deterministic                       | Maximum Throughput                                            |
| Handling of interrupts and internal system exceptions | Kept minimal                             | Regular                                                       |
| Handling of Critical Section                          | Usually turn on                          | May be turn off                                               |
| Scheduling Mechanism                                  | Simple & Favorites Hight Priority Tasks  | Can be complicated & favorites small task (fairness policy)   |
| Interrupt Latency Scheduling Latency                  | Bounded                                  | Unbounded                                                     |
| Priority invertion                                    | Avoided                                  | Exists                                                        |


## 2. Task:
* It is a C function.
* It should be run within infinite loop.
* It has its own part of stack and priority.
* It can be in one of 4 states: RUNNING. BLOCKED, SUSPENDED, READY.
* It is created and deleted by calling API functions  

## 3. Resources used:
* Core resources:  
  - System Timer (SysTick) - generate system time (time slice)
  - Two stack pointers: MSP (Master Stack Pointer), PSP (Process Stack Pointer)
* Interrupt vectors:
  - SVC     - system service call
  - PendSV  - pended system call (switching context)
  - SysTick - System Timer
* Flash memory: 6-10kB
* RAM memory: ~0.5kB + task stacks

### SVC: System Service Call:
Instruction and an exception. Once the svc instruction is executed, SVC IRQ is triggered immediately (unless there is higher priority IRQ active).
SVC contains an 8bit immediate value that could help to determine which OS service is requested.
Do not use SVC inside NMI (Non Maskable Interrupt) or Hard Fault handler.

### PendSV: Pended System Call:
Progrmmable exception triggered by SW (write to the ICSR register @0xE000ED04: SCB->ICSR |= (1<<28))
It is not precise (in contrary to SVC). After set a pending bit, CPU can execute a number of instructions before the exception will start. Usually it is used like a subroutine called i.e. by the system timer in OS.

### SysTick - System Timer:
It is necessary to trigger a context switching in regular time slots.
In CortexM architecrute 24Bit downcounting SysTick is used for this purpose (it can be changed).
System Timer is triggering PendSV SW interrupt to perform context switch.
In case we are using HAL library it is strongly recommended to change its TimeBase from SysTick to other timer available.