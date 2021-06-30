---
layout: post
title: FreeRTOS Background Part-2
date: 18-06-2021 18:00:00 +0200
description: FreeRTOS Background. # Add post description (optional)
img: FreeRTOS_Logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [RTOS]
---

FreeRTOS Background Part-2
## 1. API conventions:
* Prefixes at variable names:
c - char
s - short
l - long
x - portBASE_TYPE defined in portmacro.h for each platform (in STM32 it is long)
u - insigned
p - pointer

* Functions name structure:
For example: vTaskPrioritySet():
v - prefix, Task - file name, PrioritySet - fuction name

v - void 
x - returns portBASE_TYPE
prv - private

* Macros:
  - Prefixes at macros defined their definition location:
    - port   - (ie. portMAX_DELAY)        -> portable.h
    - task   - (ie. task_ENTER_CRITICAL)  -> task.h
    - pd     - (ie. pdTRUE)               -> projdefs.h
    - config - (ie. configUSE_PREEMPTION) -> FreeRTOSConfig.h
    - err    - (ie. errQUEUE_FULL)        -> projdefs.h

  - Common macro definitions:
    - pdTRUE  1
    - pdFALSE 0
    - pdPASS  1
    - pdFAIL  0

* Each OS component has its own ID:
  - Task: osThreadid_t mapped to TaskHandle_t
  - Queues: osMessageQid_t mapped to QueueHandle_t
  - Semaphores: osSemaphoreid_t mapped to SemaphoreHandle_t
  - Mutexes: osMutexid_t mapped to SemaphoreHandle_t
  - SW timers: osTimerid_t mapped to TimerHandle_t  
  
*  Status code values returned by CMSIS-RTOS functions cmsis_os2.h:
```
typedef enum {
  osOK                      =  0,         ///< Operation completed successfully.
  osError                   = -1,         ///< Unspecified RTOS error: run-time error.
  osErrorTimeout            = -2,         ///< Operation not completed within the timeout.
  osErrorResource           = -3,         ///< Resource not available.
  osErrorParameter          = -4,         ///< Parameter error.
  osErrorNoMemory           = -5,         ///< System is out of memory
  osErrorISR                = -6,         ///< Not allowed in ISR context.
  osStatusReserved          = 0x7FFFFFFF  ///< Prevents enum down-size compiler optimization.
} osStatus_t;
``` 
   
* Thread state cmsis_os2.h:
```
typedef enum {
  osThreadInactive        =  0,         ///< Inactive.
  osThreadReady           =  1,         ///< Ready.
  osThreadRunning         =  2,         ///< Running.
  osThreadBlocked         =  3,         ///< Blocked.
  osThreadTerminated      =  4,         ///< Terminated.
  osThreadError           = -1,         ///< Error.
  osThreadReserved        = 0x7FFFFFFF  ///< Prevents enum down-size compiler optimization.
} osThreadState_t;  
```  
    
* Attributes structure for thread cmsis_os2.h: 
```
typedef struct {
  const char                   *name;   ///< name of the thread
  uint32_t                 attr_bits;   ///< attribute bits
  void                      *cb_mem;    ///< memory for control block
  uint32_t                   cb_size;   ///< size of provided memory for control block
  void                   *stack_mem;    ///< memory for stack
  uint32_t                stack_size;   ///< size of stack
  osPriority_t              priority;   ///< initial thread priority (default: osPriorityNormal)
  TZ_ModuleId_t            tz_module;   ///< TrustZone module identifier
  uint32_t                  reserved;   ///< reserved (must be 0)
} osThreadAttr_t;
```