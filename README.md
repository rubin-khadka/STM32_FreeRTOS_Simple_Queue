# STM32 FreeRTOS Simple Queue

## Overview

This project demonstrates inter-task communication using queues in an STM32 microcontroller with FreeRTOS. Queues are used to safely pass data between tasks and even from interrupt service routines (ISRs) to tasks.

The application creates three tasks with different priorities:
- Sender_HPT_Task (Highest Priority: 3) - Sends value 222 every 2000ms
- Sender_LPT_Task (Medium Priority: 2) - Sends value 111 every 1000ms
- Receiver_Task (Lowest Priority: 1) - Receives and displays values every 5000ms

A queue with capacity of 5 integers is used to pass data between sender tasks and the receiver task. This demonstrates how FreeRTOS queues provide thread-safe communication between tasks.

> This project uses native FreeRTOS APIs directly, not the CMSIS-RTOS wrapper.

## Key Concept: Multiple Producers - Single Consumer
- Two Producer Tasks: Both sender tasks continuously put data into the queue
- One Consumer Task: Receiver task takes data out of the queue
- ISR as Special Producer: UART interrupt can also send data when user types 'r'
- Queue as Buffer: Temporarily stores messages when consumer is busy

## UART Output

https://github.com/user-attachments/assets/f28b0cab-46fc-4087-8805-57ace04abe2b

## Hardware Requirements
- STM32 development board (STM32F103C8T6 "Blue Pill")
- USB-to-UART converter (for viewing debug messages and sending commands)
- UART connection (PA9/PA10 for USART1 on STM32F103)

## Pin Configuration
| Pin | Function | Description |
|-----|----------|-------------|
| PA9 | USART1 TX | Debug output from STM32 |
| PA10 | USART1 RX | Receive 'r' command to trigger ISR send |

## Task Priorities and Behavior
| Task | Priority | Data Sent | Send/Receive Rate | Role |
|------|----------|-----------|-------------------|------|
| Sender_HPT_Task | 3 (Highest) | 222 | Every 2000ms | Producer |
| Sender_LPT_Task | 2 | 111 | Every 1000ms | Producer |
| Receiver_Task | 1 (Lowest) | N/A (Receives) | Every 5000ms | Consumer |

## Queue Configuration
- Queue Length: 5 elements
- Element Size: sizeof(int) (4 bytes)
- Total Size: 20 bytes
- Access Method: FIFO (First In, First Out)

## Queue Behavior
1. Thread-Safe Communication
    - Multiple tasks can safely send to the same queue
    - No data corruption or race conditions
    - Automatic mutual exclusion handled by FreeRTOS
2. Task Synchronization
    - Receiver blocks when queue is empty
    - Senders block when queue is full (portMAX_DELAY)
    - Tasks automatically unblock when condition changes
3. Priority-Based Scheduling
    - Higher priority sender tasks run more frequently
    - Lower priority receiver processes data when available
4. ISR to Task Communication
    - UART ISR can send data to queue (special FromISR function)
    - Demonstrates safe interrupt-to-task data transfer
    - Shows proper context switching from ISR

## Queue Operations Comparison
| Operation | Task Context | ISR Context | Blocking Behavior |
|-----------|--------------|-------------|-------------------|
| Send | `xQueueSend()` | `xQueueSendFromISR()` | Blocks if queue full |
| Send to Back | `xQueueSendToBack()` | `xQueueSendToBackFromISR()` | Adds to end of queue |
| Send to Front | `xQueueSendToFront()` | `xQueueSendToFrontFromISR()` | Adds to front of queue |
| Receive | `xQueueReceive()` | `xQueueReceiveFromISR()` | Blocks if queue empty |
| Peek | `xQueuePeek()` | `xQueuePeekFromISR()` | Views without removing |

## Special Feature: ISR Triggered Send
When you type 'r' in the serial terminal:
1. UART ISR is triggered
2. ISR sends value 12345 to the queue using xQueueSendToBackFromISR()
3. Proper context switching is handled with portEND_SWITCHING_ISR()
4. Receiver task will process this data when it runs

This demonstrates:
- Interrupt safety: Special FromISR functions must be used
- Context switching: Higher priority tasks may need to run immediately
- Real-time response: Interrupts can communicate with tasks

## Task vs ISR Queue Operations
| Aspect | Task Operation | ISR Operation |
|--------|----------------|---------------|
| Send Function | `xQueueSend()` | `xQueueSendFromISR()` |
| Receive Function | `xQueueReceive()` | `xQueueReceiveFromISR()` |
| Blocking | Can block with timeouts | Cannot block (returns immediately) |
| Context Switch | Automatic when blocked | Manual with `portEND_SWITCHING_ISR()` |
| Use Case | Normal task communication | Interrupt-driven events |

## Queue States and Task Behavior

When Queue Has Space (Not Full)
- Sender tasks: Send immediately, no blocking
- Receiver task: Receives immediately if data available

When Queue is Full
- Sender tasks: Block until space becomes available
- Receiver task: Must take data to free space

When Queue is Empty
- Sender tasks: Send immediately
- Receiver task: Blocks until data arrives

## Contact
**Rubin Khadka Chhetri**  
📧 rubinkhadka84@gmail.com <br>
🐙 GitHub: https://github.com/rubin-khadka
