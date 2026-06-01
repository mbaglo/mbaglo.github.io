---
layout: post
title: "Building a Bank Teller Simulation with FreeRTOS on STM32"
date: 2026-06-01
description: "A real-time systems project using STM32 and FreeRTOS to simulate bank tellers, customer queues, breaks, and performance metrics."
excerpt: "In this project, I built a bank teller simulation using STM32 and FreeRTOS. The system models customer arrivals, teller activity, scheduled and forced breaks, queue behavior, and end-of-day performance metrics."
---

In this project, I built a **bank teller simulation** using **STM32** and **FreeRTOS** to model customer arrivals, teller service behavior, break scheduling, queue activity, and end-of-day reporting in a real-time embedded environment.

The goal of the project was to apply RTOS concepts in a practical way by simulating a system with multiple concurrent tasks and shared resources. Instead of treating task scheduling as an abstract idea, this project made it possible to observe how semaphores, queues, mutexes, timers, and interrupts work together in a realistic embedded application.

## Project Overview

The simulation models a small bank with:
- a customer arrival task,
- three teller tasks,
- separate break-management tasks,
- a metrics task for monitoring and final reporting,
- and a random-number task for generating variable timing behavior.

Customers are generated at random intervals and placed into a message queue. Tellers wait for customers, process transactions with randomized service times, and update queue and service statistics. Break tasks allow tellers to take both scheduled and forced breaks, while a metrics task continuously reports system status and generates an end-of-day summary once the bank closes.

## RTOS Concepts Used

This project gave me hands-on practice with several important RTOS mechanisms:

### Tasks
Multiple tasks run concurrently, each with a specific role:
- **Bank task** for customer generation
- **Teller tasks** for customer service
- **Break tasks** for teller break control
- **Metrics task** for reporting queue and teller statistics
- **Random number task** for timing variability

This modular task structure made the simulation easier to organize and helped separate system behavior into manageable components.

### Message Queues
Customer arrivals are stored in a FreeRTOS message queue. Each queued item represents a customer arrival time, which is later used to compute wait time before service.

This queue-based design made it possible to simulate realistic customer buildup and teller response under different timing conditions.

### Semaphores
Semaphores are used to coordinate several parts of the system:
- signaling when customers are ready,
- tracking teller availability,
- handling UART transmission completion,
- and triggering forced breaks for individual tellers.

This helped reinforce the difference between event signaling and mutual exclusion in embedded multitasking systems.

### Mutexes
Mutexes protect shared resources such as:
- queue statistics,
- teller statistics,
- and UART output.

Because multiple tasks update shared state, mutex protection is necessary to avoid inconsistent data and race conditions.

## Simulation Features

A few features made the simulation especially interesting to build:

### Randomized Customer Arrivals and Service Times
The system uses the STM32 hardware RNG to generate variable arrival times, transaction lengths, and break timing. This makes the simulation more dynamic and allows the queue depth and teller utilization to change over time.

### Teller Break Handling
Each teller can take:
- **scheduled breaks**, based on randomly assigned intervals and durations,
- or **forced breaks**, triggered through external button interrupts.

This introduced an interactive hardware element into the simulation and made the system behavior more realistic.

### Real-Time Metrics Reporting
The metrics task periodically reports:
- simulated clock time,
- number of customers in queue,
- teller status,
- customers served,
- wait times,
- service times,
- break statistics,
- and system idle time.

At the end of the simulated workday, the project generates an end-of-day summary to show how the bank performed overall.

### 7-Segment Queue Display
The project also displays the number of customers waiting in the queue on a 7-segment display, which adds a useful visual hardware interface to the simulation.

## What I Learned

This project helped me strengthen my understanding of embedded real-time system design in several ways.

First, it showed how a system with many moving parts can be organized through modular task design. Separating bank logic, teller behavior, break handling, and metrics collection made the code easier to reason about and debug.

Second, it highlighted the importance of synchronization. A project like this quickly becomes unreliable if shared resources are not protected correctly or if task coordination is not handled carefully.

Third, it demonstrated how embedded systems can combine software logic with physical interaction. The use of button-triggered breaks, UART output, timers, and 7-segment display output made the simulation feel much closer to a real embedded control system than a purely software-only exercise.

## Reflection

What I liked most about this project was that it connected RTOS theory to implementation. Concepts like queues, semaphores, idle time, and concurrency became much more meaningful once I could see them affecting system behavior in a complete embedded application.

This project also gave me more confidence working with STM32-based systems that combine scheduling, timing, peripherals, and shared-state coordination.

As I continue building embedded and sensing systems, projects like this help reinforce the design discipline required for larger real-time applications.

[View project code](https://github.com/mbaglo/MyProjects/tree/main/Real%20Time%20%26%20Embedded%20System/Project%203_Bank%20Teller%20Simulation)

[← Back to Blog](/blog/)
