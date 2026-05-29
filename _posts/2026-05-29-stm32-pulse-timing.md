---
layout: post
title: "Building a Pulse Timing Measurement Tool with STM32 Input Capture"
date: 2026-05-29
description: "A blog post about measuring pulse timing in microseconds with STM32 timers, input capture interrupts, UART interaction, and histogram-based signal analysis."
tags: [STM32, Embedded Systems, C, Microcontrollers, UART, Timers]
---

Accurate timing is one of the most important parts of embedded systems design. In this project, I built a pulse timing measurement tool using an STM32 microcontroller to detect an incoming signal, measure the time between rising edges, and analyze the consistency of those measurements in microseconds.

The project uses **TIM2 input capture** to record two consecutive rising edges and calculate the time difference between them. Since the timer prescaler is configured so the counter runs at **1 MHz**, each timer tick represents **1 microsecond**, which makes the measured pulse timing easy to interpret directly.

## Project Goal

The goal of this project was to create a simple embedded measurement system that could:

- detect whether an input signal is present,
- measure pulse timing with microsecond resolution,
- allow the user to define a timing range through UART,
- collect 1000 valid measurements, and
- summarize the results in a histogram-style table.

This makes the project useful for evaluating how stable or noisy a repeated signal is over time.

## How It Works

The firmware begins with a **power-on self-test (POST)**. During this step, the program starts the input capture peripheral and waits briefly for a signal. If a capture event occurs, the system reports success through UART. If no signal is detected, the user is asked whether to retry or continue.

After POST, the program displays the current lower and upper pulse-width limits through **UART2**. By default, the lower limit is **950 µs** and the upper limit is **1050 µs**, but the user can enter a new lower limit at runtime. The program then automatically sets the upper limit to 100 microseconds above the lower limit.

Once configuration is complete, the STM32 starts collecting input capture events. The interrupt callback stores two captured timer values:

- the first rising edge timestamp,
- the second rising edge timestamp.

The difference between these two values represents the measured pulse period.

## Timing and Signal Processing

A key part of this project is the timer setup. With the prescaler set to **79**, the timer clock is reduced so that the counter increments once every microsecond. That means the difference between capture values can be interpreted directly as pulse width in microseconds, without requiring extra conversion logic.

When a pulse measurement falls within the selected timing window, it is stored in one of **101 histogram buckets**. Each bucket corresponds to a specific microsecond value between the lower and upper bounds.

This design allows the program to do more than just calculate an average. Instead, it shows the full distribution of valid pulse widths, which makes it easier to identify timing variation, clustering, or instability in the signal.

## Output

After **1000 valid measurements** are collected, the program stops input capture and prints the results as a UART table showing:

- the time value in microseconds,
- the number of times that value occurred.

This output gives a quick summary of signal behavior and helps visualize how consistent the timing is.

## What I Learned

This project helped me strengthen my understanding of several important embedded systems concepts:

- configuring STM32 timers for precise measurement,
- using **input capture interrupts** for event timing,
- building a simple **UART-based user interface**,
- organizing measurement data into histogram buckets,
- and combining hardware peripherals to create a practical diagnostic tool.

It also showed how low-level peripherals like timers and UART can be combined to build a complete measurement application with both data collection and user interaction.

## Possible Improvements

If I continue developing this project, I would like to add:

- explicit handling for timer counter overflow,
- input filtering for noisy or unstable signals,
- configurable upper and lower bounds,
- CSV-style serial output for easier logging,
- and possibly a graphical visualization tool on the PC side.

## Final Thoughts

Overall, this project is a strong example of how embedded systems can be used for precise real-time measurement. By combining **STM32 timers**, **interrupt-based capture**, and **UART communication**, I was able to build a small but effective tool for pulse timing analysis.

Projects like this are a great way to move beyond basic microcontroller exercises and start building systems that solve real measurement and debugging problems.
