---
layout: post
title: "Building a Bank Teller Simulation with FreeRTOS on STM32"
date: 2026-06-01
description: "A detailed real-time systems project using STM32 and FreeRTOS to simulate bank tellers, customer queues, break handling, synchronization, and performance monitoring in an embedded environment."
excerpt: "This project presents a detailed bank teller simulation built on STM32 with FreeRTOS. It models concurrent customer arrivals, teller servicing, scheduled and forced breaks, queue management, and end-of-day performance reporting using tasks, semaphores, mutexes, interrupts, and message queues."
---

This project presents a **bank teller simulation** implemented on **STM32** using **FreeRTOS** to model concurrent customer arrivals, teller servicing, break scheduling, queue dynamics, and end-of-day reporting in a real-time embedded environment.

The work was designed as a practical exploration of real-time operating system concepts in a structured embedded application. Rather than demonstrating RTOS primitives in isolation, the project integrates **tasks, message queues, semaphores, mutexes, timers, interrupts, UART communication, and display output** into a single system that reflects the behavior of a small service environment.

## Project Objective

The primary objective of the simulation is to demonstrate how a real-time embedded system can coordinate multiple concurrent activities while maintaining correct shared-state management and predictable system behavior. The project models a bank environment in which customers arrive over time, join a queue, are served by one of three tellers, and contribute to system-wide statistics such as wait time, service time, queue depth, idle time, and break duration.

In addition to software coordination, the project incorporates hardware interaction through UART logging, timer-driven simulated time, button-triggered teller breaks, and 7-segment display output for queue depth. This makes the implementation more representative of a complete embedded system rather than a purely algorithmic simulation.

## System Architecture

The simulation is organized as a set of cooperating RTOS tasks, each with a clearly defined responsibility:

- **Bank task**: generates customers at randomized time intervals and places them into the queue.
- **Teller tasks**: represent three concurrent service agents that wait for customers, process transactions, and update performance statistics.
- **Break tasks**: monitor each teller for scheduled or forced breaks and manage break timing.
- **Metrics task**: periodically reports system status and generates the final end-of-day summary.
- **Random number task**: supports timing variability for simulation behavior.

This modular design improves readability, simplifies debugging, and reflects a common embedded-software practice of separating responsibilities into focused concurrent units.

## Customer Queue and Arrival Modeling

Customer arrivals are modeled by the bank task, which runs continuously while the bank remains open. The system checks whether the simulated time falls within operating hours and, when open, generates customers at randomized intervals using the STM32 hardware random number generator.

Each customer is represented by an arrival timestamp. That timestamp is placed into a FreeRTOS message queue, which serves as the central waiting line for the bank. The queue is bounded, which allows the project to simulate the practical effects of limited service capacity. If the queue reaches its maximum size, the system logs that no additional customers can be accepted until space becomes available.

This approach provides a realistic way to model congestion and enables later calculation of wait times based on the difference between a customer’s arrival time and the time service actually begins.

## Teller Operation and Service Logic

Three teller tasks operate concurrently and represent the available service stations. Each teller waits for a signal indicating that a customer is ready in the queue. Once a customer is retrieved from the message queue, the teller simulates a transaction with a randomized service duration.

During customer service, the system updates multiple performance metrics, including:
- number of customers served,
- total service time,
- maximum service time,
- total idle time,
- and maximum idle time.

The teller logic also measures how long each customer waited before service. These wait times contribute to aggregate queue statistics such as total wait time and maximum observed wait time. By combining queue-level and teller-level metrics, the project provides a more complete view of operational performance over the simulated workday.

## Break Scheduling and Forced Break Control

One of the most distinctive features of the project is its break-management subsystem. Each teller has a dedicated break task that monitors whether it is time for a scheduled break or whether a forced break has been triggered externally.

Scheduled breaks are determined using randomized intervals and durations. This introduces variability into teller availability and helps simulate the types of interruptions that affect real service systems.

Forced breaks are triggered using hardware button interrupts. When a button is pressed, the corresponding teller enters a break state, and the event is reflected in system logging and LED output. When the button state changes, the teller can return to active operation.

This design demonstrates how RTOS task coordination can be integrated with external interrupt handling to create interactive embedded behavior. It also shows the additional complexity introduced when resource availability changes asynchronously.

## Synchronization and Shared Resource Protection

Because the simulation involves multiple tasks operating on shared data, synchronization is essential. The project uses several RTOS synchronization primitives to maintain correctness:

### Message Queues
A FreeRTOS message queue is used to hold pending customers. This provides orderly communication between the customer-generation logic and the teller-service logic.

### Semaphores
Semaphores are used to signal important events and manage availability, including:
- customer readiness,
- teller availability,
- UART transmission completion,
- and break activation for each teller.

These semaphores allow tasks to coordinate efficiently without constant polling.

### Mutexes
Mutexes are used to protect shared resources such as:
- queue statistics,
- teller statistics,
- and UART communication.

Without mutual exclusion, multiple tasks updating these resources simultaneously could produce inconsistent counters, corrupted output, or race conditions. The project therefore demonstrates an important embedded-systems principle: concurrency must be paired with disciplined synchronization.

## Simulated Time and Real-Time Behavior

The project uses a timer-driven mechanism to advance simulated time while running on real hardware. A periodic timer interrupt updates the simulated bank clock, and helper functions convert that time into a readable clock representation.

This allows the simulation to operate on an accelerated timescale while still preserving a clear sense of operating hours, transaction duration, and break intervals. For example, the bank can open, process a full workday of events, and close within a practical demonstration period on the hardware platform.

The use of simulated time is especially important for performance reporting because it provides meaningful metrics such as average wait time, service duration, and break duration in application-specific units rather than raw tick values alone.

## Monitoring and End-of-Day Reporting

The metrics task provides continuous visibility into system behavior. At regular intervals, it reports:
- current simulated time,
- customers waiting in queue,
- teller status,
- teller service counts,
- and related operating information.

In addition, the queue depth is displayed on a 7-segment display, adding a visual hardware-based summary of current load.

Once the bank closes and all customers have been served, the system generates an end-of-day report. This report includes:
- total customers served,
- customers served by each teller,
- average and maximum customer wait times,
- teller average service times,
- teller idle times,
- maximum transaction times,
- number of breaks taken,
- average break times,
- longest and shortest break durations,
- maximum queue depth,
- and idle-hook statistics for overall system idle time.

This reporting stage transforms the simulation from a simple concurrency exercise into a performance-analysis tool. It allows the project to capture not only what the system did, but also how effectively it operated under changing conditions.

## Embedded Interfaces and Hardware Interaction

A notable strength of the project is its integration of software scheduling with physical hardware features. Several interfaces contribute to this:

- **UART output** is used for logging simulation events and reporting results.
- **Button interrupts** allow manual triggering of teller breaks.
- **LED indicators** provide immediate visual feedback for teller break states.
- **7-segment display output** shows queue occupancy.
- **STM32 hardware RNG** introduces realistic variability in arrivals, transactions, and breaks.

These features help bridge the gap between abstract RTOS theory and practical embedded implementation. The system is not only concurrent in software, but also interactive and observable through the hardware platform.

## Technical Significance

From a systems perspective, the project demonstrates several important design principles:

- decomposition of behavior into concurrent tasks,
- safe communication between producers and consumers,
- protection of shared state,
- interaction between interrupt-driven and task-driven execution,
- performance monitoring in a resource-constrained environment,
- and integration of hardware peripherals into a real-time application.

The project also highlights how even a conceptually simple service model becomes significantly more complex when concurrency, timing variability, asynchronous events, and shared resources are involved. That complexity makes the project a useful exercise in disciplined embedded design.

## Conclusion

This bank teller simulation serves as a detailed example of how **STM32** and **FreeRTOS** can be used to build a structured, interactive, and measurable real-time embedded application. By combining concurrent task scheduling with queue management, synchronization, hardware interrupts, and runtime metrics, the system provides a strong demonstration of applied RTOS principles.

Beyond simulating a bank environment, the project shows how embedded systems can be designed to manage multiple independent activities while preserving correctness, visibility, and responsiveness. As a result, it offers value both as a learning exercise in real-time systems and as a foundation for thinking about more advanced embedded monitoring and control applications.

[View project code](https://github.com/mbaglo/MyProjects/tree/main/Real%20Time%20%26%20Embedded%20System/Project%203_Bank%20Teller%20Simulation)

[← Back to Blog](/blog/)
