---
layout: post
title: "Pulse Timing Measurement on STM32: Input Capture, UART UI, and Statistical Analysis"
date: 2026-05-29
description: "A detailed guide to developing a pulse-timing measurement tool on STM32 using input capture interrupts, microsecond-scale timing, UART user interface, and histogram analysis of signal stability."
tags: [STM32, Embedded Systems, C, Microcontrollers, Timers, UART, Measurement, Signal Analysis]
---

## Project Overview

This project demonstrates how to design a high-precision pulse timing measurement system on an STM32 microcontroller. Leveraging the hardware timer's input capture mode, along with a UART-based user interface and real-time statistical analysis, this project can evaluate the stability and quality of digital pulse signals with microsecond resolution.

---

## Hardware and Peripherals

- **Microcontroller:** STM32L4 series (compatible with other STM32s with timer capture)
- **Signal Input:** Digital signal routed to a timer input capture pin
- **UART2:** Used for configuration and displaying results
- **Timers:** TIM2 in input capture mode
- **No display required:** Results via UART

---

## System Architecture

The firmware for this project is modular, dividing key functionality as follows:

### 1. Power-On Self-Test (POST)
- At startup, the STM32 initializes all hardware and begins a brief POST.
- The program waits for a valid pulse signal on the input capture pin, ensuring hardware readiness before measurements begin.
- If no signal is detected, the system can notify the user and halt further operation.

### 2. User Configuration (via UART)
- After POST, the user is prompted—over UART—to configure a pulse width window of interest (lower and upper bounds, in microseconds).
- Default window is 950 μs – 1050 μs, but these can be altered via UART commands before measurement begins.

**UART menu excerpt:**
```
Enter lower and upper pulse width limits (in microseconds): 
(Default: 950 1050) 
```

### 3. Input Capture & Measurement

- **Timer Setup:** TIM2 is configured so that, after prescaling, one timer count = 1 μs.
- **Input Capture Interrupt:** Each rising edge triggers the capture callback, storing the timer value.
- The difference between two consecutive rising edges yields the pulse period in microseconds.

**Timer and ISR code excerpt:**
```c
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
    if (Is_First_Captured == 0) {
        IC_Val1 = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);
        Is_First_Captured = 1;
    } else if (Is_First_Captured == 1) {
        IC_Val2 = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);
        Is_Second_Captured = 1;
    }
}
```

- If the measured period is within the configured window, it is saved to a histogram bucket.

### 4. Histogram and Signal Statistics

- The firmware maintains a histogram array (101 buckets by default) corresponding to each possible pulse width in the selected range.
- For each valid measurement, the count in the appropriate bucket is incremented.
- Once a preset number of valid measurements (default: 1000) is reached, the system summarizes and prints the entire histogram via UART.

**Analysis code snippet:**
```c
if (period >= lower_limit && period <= upper_limit) {
    bucket[period - lower_limit]++;
    capture_done++;
}
```

---

## Output Format

Once data collection is complete, the firmware outputs a formatted summary (over UART) showing:

- Each pulse width (in μs) within the configured window
- The number of times it was observed

Sample output (UART):

```
Pulse Width (us)    Count
950                 0
951                 2
...
1000                700
...
1050                0
```

This allows rapid, statistical analysis of signal quality and repeatability.

---

## Educational Value & Technical Insights

This project covers important embedded systems concepts:
- **Timer/Counter configuration** for microsecond precision
- **Input capture interrupts** for event-driven measurement
- **UART menu interaction** for flexible, user-configurable data acquisition
- **Statistical data collection** (histograms) to analyze signal jitter, instability, or noise
- **Robust startup procedures** (POST) for verifying hardware function before use

### Block Diagram

```
+------------------+            +------------------------------+
| Signal Source    |--(GPIO)->--| [STM32] TIM2 Input Capture   |
+------------------+            +-----------+------------------+
                                         |
         +-------------------------------+
         |
+-----------------------+
|      UART Output      |---(PC/Terminal)
+-----------------------+
```

---

## Possible Improvements

For further extension and robustness, consider:
- **Timer overflow/rollover detection** for handling long periods
- **Noise rejection/filtering** for unstable signals
- **Data export in CSV format** for easier PC-based plotting
- **Real-time graphical display** (e.g., plotting histograms)
- **Calibration UI** for automatic window finding

---

## Source Code and Resources

- [Pulse Timing Project Code (GitHub)](https://github.com/mbaglo/MyProjects/tree/main/Real%20Time%20%26%20Embedded%20System/Project%201_Timing)

---

## Conclusion

This STM32-based real-time measurement system is a practical example of applying hardware timers, interrupt-driven acquisition, and statistical analysis to a fundamental embedded problem. The open design and parameterization allow anyone to adapt it for new timing measurement challenges, signal analysis, or as a diagnostic tool in more complex systems.
