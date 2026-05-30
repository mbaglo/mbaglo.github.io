---
layout: post
title: "Servo Control on STM32: Implementing State Machines and Recipe Execution"
date: 2026-05-30
description: "A comprehensive guide to building a dual-servo control system on STM32, featuring robust state machine management, UART-based user commands, and error handling."
excerpt: "How to build a robust dual-servo control system on STM32 using state machines, UART commands, recipes, and reliable real-time techniques."
tags: [STM32, Embedded Systems, C, Servo Control, State Machine, UART, Timers, PWM, Real Time]
---

## Project Overview

This educational project demonstrates how to design and implement a robust dual-servo control system using an STM32 microcontroller. By utilizing structured state machines, PWM timer channels, user command input via UART, and rigorous error handling, you can control complex servo movements reliably and extend the system for custom routines.

## Hardware and Peripherals

- **MCU:** STM32 (tested on STM32L4 series)
- **Servos:** Two generic hobby servos (PWM-controlled)
- **UART:** UART2 for user command input/output
- **Timers:** TIM2 for dual PWM output; TIM4 for periodic (100ms) interrupt
- **GPIO:** Two output pins for status LEDs

## Software Architecture

### 1. State Machine Framework
The core of the servo control project is a finite state machine (FSM) for each servo (see `state_machine.h/.c`). Servo states include *paused, moving, waiting, looping, error*, and transitions occur in response to user commands or recipe execution.

### 2. Recipe Execution
A "recipe" is a programmatic instruction set stored as a byte array, interpreted at runtime. Recipes can move the servo to positions, pause, implement loops, or reverse direction. Example:

```c
unsigned char recipe1[] = {
    MOV + 0,
    MOV + 5,
    LOOP + 1,
    MOV + 1,
    MOV + 4,
    END_LOOP,
    RECIPE_END
};
```

Commands include `MOV`, `WAIT`, `LOOP`, `END_LOOP`, `REVERSE`, `RECIPE_END`.

### 3. UART User Commands
UART2 is set for interrupt-driven communication. The system displays command options and accepts two-letter inputs (one per servo) for pause (P), continue (C), move R/L, begin recipe (B), cancel (X), and reverse (V).

| Command | Action                 |
|---------|-----------------------|
| P       | Pause Recipe Execution|
| C       | Continue/Resume       |
| R/L     | Move Right/Left       |
| B       | Begin Assigned Recipe |
| X       | Cancel/Reset Recipe   |
| V       | Reverse Direction     |

### 4. Timer Logic and Interrupts

- **TIM2** provides PWM generation for both servos (channels 1 and 2).
- **TIM4** triggers every 100ms, acting as the main scheduler for FSM updates and recipe execution.
- UART reception/timer flags handled via interrupt callbacks for responsiveness.

## Implementation Highlights

#### Initialization and Main Scheduler
```c
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_USART2_UART_Init();
  MX_TIM2_Init();
  MX_TIM4_Init();
  HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
  HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_2);
  HAL_TIM_Base_Start_IT(&htim4);
  HAL_UART_Receive_IT(&huart2, (uint8_t *)rx_buffer, 2);
  usart2_transmit("Welcome to the Servo Control System!...\n");
  while (1) {
      if (timer_flag) {
          timer_flag = 0;
          if (rx_flag) {
              process_user_commands(rx_buffer);
              HAL_UART_Receive_IT(&huart2, (uint8_t *)rx_buffer, 2);
              rx_flag = 0;
          }
          // FSM and LED updates
      }
  }
}
```

#### Key Functions
- `process_user_commands()` — parses UART buffer & triggers events for servo FSMs
- `process_servo_recipe()` — executes instructions: move, wait, loop, reverse
- `update_status_leds()` — reflects running, paused, error state
- `set_servo_position()` — maps position (0–5) to appropriate PWM
- Robust error handling: any invalid state moves system into safe error state

## Experimentation and Extension
- Write custom servo recipes for complex motion
- Test error handling with nested/invalid loops
- Extend command set or allow synchronization
- Try different wait/blocking modes or servo models

## Educational Value
This project demonstrates:
- State machine architecture for reliability
- PWM/timer config for embedded control
- UART user interface design
- Structured modular code and robust error handling

## Source Code & Resources
- [Servo Control Source Code](https://github.com/mbaglo/MyProjects/tree/main/Real%20Time%20%26%20Embedded%20System/Project%202_Servo%20Control)

## Conclusion
A structured state machine, flexible UI, and rigorous error handling illustrate best practices in STM32 servo/motor control. The code and concepts provide an excellent springboard for further robust embedded development.
