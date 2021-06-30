---
layout: post
title: FreeRTOS Background Part-3
date: 19-06-2021 08:00:00 +0200
description: FreeRTOS Background. # Add post description (optional)
img: FreeRTOS_Logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [RTOS]
---

FreeRTOS Background Part-3
## 1. Memory allocation hints:
* FreeRTOS uses a region of memory called Heap (into the RAM) to allocate memory for tasks, queues, timers, semaphoresm mutexes and when dynamically creating variables. FreeRTOS heap is different than the system heap defined at the compiler level.
* When FreeRTOS requires RAM instead of calling the standard malloc it calls PvPortMalloc(). When it needs to free memory it calls PvPortFree() instead of the standard free().
* FreeRTOS offers several heap management schemes that range in complexity and features. it includes five sample memory allocation implementations, each of which are described in the following link: http://www.freertos.org/a00111.html
* The total amount of available heap space is set by configTOTAL_HEAP_SIZE which is defined in FreeRTOSConfig.h.
* The xPortGetFreeHeapSize() API function return the total amount of the heap space that remains unallocated (allowing the configTOTAL_HEAP_SIZE setting to be optimized). The total amount of heap space that remains unallocated is also available with xFreeBytesRemaining variable for heap management schemes 2 to 5.

### Tasks
* Each created task (including the idle task) requires a Task Control Block (TCB) and a stack that are allocated in the heap.
* The TCB size in bytes depends of the options enabled in the FreeRTOSConfig.h.
    - With minimum configuration the TCB size is 24 words i.e 96 bytes.
    - If configUSE_TASK_NOTIFICATIONS enabled add 8 bytes (2 words).
    - If configUSE_TRACE_FACILITY enabled add 8 bytes (2 words).
    - if configUSE_MUTEXES enabled add 8 bytes (2 words).
* The task stack size is passed as argument when creating at task. The task stack size is defined in words of 32 bits not in bytes.
for example: osThreadDef(Task_A, Task_A_Function, osPriorityNormal, 0, stacksize);
* FreeRTOS requires to allocate in the heap for each task:
    - number of bytes = TCB_size + (4 x task stack size)
* configMINIMAL_STACK_SIZE defines the minimum stack stack size that can be used in words. The idle task stack size takes automatically this value.
* The necessary task stack size can be fine0tuned using the API uxTaskGetStackHighWaterMark() as follow:
    - Use an initial large stack size allowing the task to run without issue (example 4KB)
    - The API uxTaskGetStackHighWaterMark() returns the minimum number of free bytes (ever encountered) in the task stack. Monitor the return of this function within the task.
    - Calculate the new stack size as the initial stack size minus the minimum stack free bytes.
    - The method requires that the task has been running enough to enter the worst path (in term of stack consumption).

### Semaphores, Mutexes and Queues
* Each semaphore declared by the user application requires 88 bytes to be allocated in the heap.
* Each mutex declared by the user application requires 88 bytes to be allocated in the heap (+8 bytes within TCB of each task).
* To save heap size (i.e. RAM footprint) it is recommended to disable the define configUSE_MUTEXES when mutexes are not used by the application (task TCB static size being reduced)
* FreeRTOS requires to allocate in the heap for each message queue (control block and queue storage area):
    - number of bytes 76 + queue_storage_area.
    - queue_storage_area (in bytes) = (element_size * nb_elements) + 16

### Software Timers
* When Software Timers are enabled (configUSE_TIMERS enabled), the scheduler creates automatically the timers service task (daemon) when started. The timers service task is used to control and monitor (internally) all timers that user will create. The timers task parameters are set through the following defines:
    - configTIMER_TASK_PRIORITY: priority of the timers service task
    - configTIMER_TASK_STACK_DEPTH: timers task stack size (in words)
* The scheduler also creates automatically a message queue used to send commands to the timers service task (timer start, timer stop...)
* The number of the elements of this queue (number of messages that can be hold) are configurable through the define: configTimer_QUEUE_LENGTH.
* FreeRTOS requires to allocate on the heap for software timers:
    1. Timers Daemon Task (in bytes): TCB_size + (4 x configTIMER_TASK_STACK_DEPTH)
    2. Timers message queue: number of bytes = 76 + queue_storage_area (queue_storage_area = 12 * configTIMER_QUEUE_LENGTH) + 16
    3. Each timer created by the user (by calling osTimerCreate()) needs 48 bytes 

### How to reduce RAM footprint
* Optimize stack allocation for each task:
    - uxTaskGetStackHighWaterMark(). This API returns the minimum number of free bytes (ever encountered) in the task stack.
    - vApplicationStackOverflowHook(). This API is a stack overflow callback called when a stack overflow is detected (available when activating the define configCHECK_FOR_STACK_OVERFLOW)
* Recover and minimize the stack used by main and rationlize the number of tasks.
* Use minimal number of priorities (configMAX_PRIORITIES)
    - configMAX_PRIORITIES defines the number of priorities available to the application tasks.
    - Any number of tasks can share the same priority.
    - The scheduler allocates statically the ready task list of size configMAX_PRIORITIES * list entry structure.
* If the application doesn't use any software timers then disable the define configUSE_TIMERS.
* If the application doesn't use mutexes then disable the define configUSE_MUTEXES.
* Adjust heap dimensioning:
    - xPortGetFreeHeapSize(). API that returns the total amount of heap space that remains unallocated. Must be used after created all tasks, messsage queues, semaphores, mutexes in order to check the heap consumption and eventually re-adjust the application define "configTOTAL_HEAP_SIZE".
    - The total amount of heap space that remains unallocated is also available with xFreeBytesRemaining variable for heap management schemes 2 to 5.

## 2. Dynamic memory management:
* Heap_1.c: 
    - Uses first fit algorithm to allocate memory. 
    - Simplest allocation method (deterministic), but does not allow freeing of allocated memory => could be interesting when no memory freeing is necessary.

* Heap_2.c (Obsolete):
    - Not recommended to new projects. Kept due to backward compatibility.
    - Omplements the best fit algorithm for allocation.
    - Allows memory free() operation but doesn't combine adjacent free blocks => risk of fragmentation.

* Heap_3.c
    - Implements simple wrapper for standard C library malloc() and free() wrapper makes these functions thread safe, but makes code increase and not deterministic.
    - It uses linker heap region.
    - configTOTAL_HEAP_SIZE setting has no effect when this model is used.

* Heap_4.c
    - Uses first fit algorithm to allocate memory. It is able to combine adjacent free memory blocks into a single block.
    - To use your own declaration configAPPLICATION_ALLOCATED_HEAP must be set to 1 (within FreeRTOSConfig.h file) and the array must be declared within user code with selected start address and size specified by configTOTAL_HEAP_SIZE.
    - Memory array used by heap_4 is specified as: uint8_t ucHeap[configTOTAL_HEAP_SIZE];
    - Heap is organazed as a linked list: for better efficiency when dynamically allocating/freeing memory. When allocating "N" bytes in the heap memory using "pvPortMalloc" API it consumes:
        - sizeof(BlockLink_t)(structure of the heap linked list): 8 bytes
        - Data to be allocated itself: N bytes
        - Add pading  to total allocated size (N+8) to be 8 bytes aligned: 52Bytes consumes from the heap: 52 + 8 = 60 bytes aligned to 8 bytes gives 64 bytes consumed from the heap.

* Heap_5.c
    - Fit algorithm able to combine adjacent free memory blocks into a single block using the same algorithms like in heap_4, but supporting different memory regions (i.e SRAM1, SRAM2) being not in linear memory space.
    - It is the only memory allocation scheme that must be explicitly initialized before any OS object can be created (before first call of pvPortMalloc()).
    - To initialize this scheme vPortDefineHeapRegions() function should be called.
    - It specifies start address and size of each separate memory area

## 3. Dynamic memory API:
* pvPortMalloc() (heap1, heap2, heap4, heap5)
    - vTaskSuspendAll() - freeze of all tasks.
    - Initialization of OS heap in case of first call to malloc.
    - Check if the requested block size is not bigger than heap top is set.
    - Allocating the memory with proper alignment.
    - vTaskResumeAll() - unfreeze the tasks
    - In case of memory error execute vApplicationMallocFailedHook()

* pvPortFree() (heap2, heap4, heap5)
    - vTaskSuspendAll() - freeze of all tasks
    - Add memory segment to available memory (pointer + memory segment size)
    - vTaskResumeAll() - unfreeze the tasks