# GSoC Proposal 2026 : Arjav Patel

# Micro-ROS Integration on NuttX

**Name:** Arjav Patel

**Degree:** B.E. Electronics & Instrumentation Engineering

**College:** BITS Pilani Goa Campus

**Email:** [arjav1528@gmail.com](mailto:arjav1528@gmail.com)

**GitHub:** https://github.com/arjav1528

**Issue Thread:** [https://github.com/apache/nuttx/issues/18508](https://github.com/apache/nuttx/issues/18508)

**JIRA Issue:** https://issues.apache.org/jira/browse/NUTTX-14

**Potential Mentors:** Alan Carvalho de Assis ([acassis@apache.org](mailto:acassis@apache.org))

## Internship Experience

- **Software Engineering Intern — Nudron IoT Solutions** (Aug 2025 – Dec 2025)
    - Refurbished the UI of an existing Flutter codebase, improving usability and design consistency.
    - Integrated cross-platform support by merging functionality into a unified app for iOS, Android, and Windows.
- **Software Engineering Intern — TechStaX** (Jan 2026 – Mar 2026)
    - Reduced memory leaks in the application and optimized app size by ~50%.
    - Integrated a Flutter application with Supabase to build a full-stack matrimony application with chat and video calling.

## Position of Responsibility

- **App Development Head — Developers’ Society, BITS Goa** (May 2025 – Present)
    - Leading a team of 50+ junior developers and 10 senior developers.
    - Responsible for maintaining 4+ applications used in BITS Goa (e.g., BPGC One, fest management apps).

---

## A Quick Look at NuttX (and Why It Matters)

Apache NuttX is an RTOS that we don’t usually see in textbooks, but it shows up in real systems. It is specially designed for constrained hardware, but at the same time, it tries to provide a developer experience that feels familiar to anyone who has worked with Unix-like systems.

This is what makes it interesting.

On the one hand, you get real-time guarantees and the ability to run on microcontrollers. Further, you still have POSIX-like APIs, a clean structure, and a fairly clean architecture. With this, instead of writing everything from scratch, you can build structured applications even on constrained hardware.

From what I’ve learned and understood, this balance is why NuttX is used in places where it matters—whether it’s IoT platforms or even space systems. Today, it’s not just about “running on small devices”, but doing that in a way that remains maintainable as the system grows.

Because of this, improvements to NuttX have real impact. If anything is missing or deprecated, it directly affects developers trying to build applications on top of NuttX.

---

## Why micro-ROS Belongs on NuttX

If you look at how robotics systems are structured today, ROS 2 is usually in the picture, handling communication between sensors, controllers, and higher-level logic.

The main problem is that ROS 2, by default, doesn’t extend all the way down to microcontrollers. That’s where micro-ROS comes in, bringing a ROS 2-compatible programming model to resource-constrained devices, so that even small MCUs can participate in a larger ROS 2 system.

micro-ROS is designed to run on top of an RTOS, and this is where NuttX comes into the picture.

NuttX already provides many of the things that micro-ROS expects from an RTOS:

- a strong POSIX-style API surface
- deterministic scheduling and real-time behavior
- support for networking and communication stacks
- the ability to scale across different microcontroller platforms

This is exactly the kind of environment where NuttX is often used: embedded control nodes that need to be predictable and reliable.

By the way, micro-ROS already supports RTOSes like FreeRTOS, Zephyr, and NuttX, and the core API remains largely consistent across them.

So my project—integrating micro-ROS with NuttX—is not about reinventing anything; it’s about bringing NuttX support to the same level of usability and stability as other RTOSes.

---

## Project Summary

At a high level, this project brings micro-ROS on NuttX back to a state where it feels usable with the current ecosystem.

micro-ROS itself is in a good place: it follows ROS 2 architecture, reuses a significant portion of its stack, and uses DDS-XRCE as lightweight middleware to make ROS 2 communication possible on microcontrollers. The design is simple yet effective: a compact local client connects to a specialized agent, which acts as a gateway into the full ROS 2 network.

This design assumes there is a capable RTOS underneath that provides basic POSIX-like abstractions, deterministic scheduling, and communication support. That’s exactly why micro-ROS officially supports RTOSes like FreeRTOS, Zephyr, and NuttX.

However, while the FreeRTOS and Zephyr integrations have kept evolving, the NuttX side hasn’t seen enough maintenance. Over that gap, the micro-ROS stack has stayed up to date, but the NuttX-specific integration (build system, examples, transports) has fallen behind.

From what I’ve read so far, the issue is not that the integration is fundamentally broken, but that it has drifted out of sync with both NuttX and micro-ROS as they evolved independently.

**So the goal here is not to redesign anything from scratch.**

Instead, the goal is to carefully update the integration layer to align with the current state of both ecosystems:

- ensure it builds cleanly against current NuttX
- verify that transports like serial and UDP work reliably
- resolve long-standing issues that block usability
- extend support to more relevant, modern boards
- integrate support directly into NuttX mainline so the build process is fully part of the main repository

An important part of this work is making the system approachable again. micro-ROS already provides a consistent programming model across RTOSes, so using it on NuttX should feel as straightforward as on Zephyr or FreeRTOS.

I also want to ensure this doesn’t become outdated again in a few months. That means adding CI support within NuttX mainline—so that if something stops working, everyone will know. The structure will align with how other maintained ports are organized, so future updates don’t silently break NuttX support.

A key deliverable is a `sim:microros` board configuration that works with the NuttX simulator, enabling testing and validation without physical hardware and making CI practical.

In short, this project is about restoring alignment between micro-ROS, NuttX, and the expectations developers have when they use them together—and doing so inside the mainline tree for long-term maintainability.

---

## Implementation Plan (Overview)

| ***Phase*** | ***Focus Area*** | ***Key Goals*** | ***Deliverables*** |
| --- | --- | --- | --- |
| Phase 1 (Weeks 1–2) | Baseline & Analysis | Understand current breakages | Build report + issue mapping |
| Phase 2 (Weeks 3–5) | Build System & Mainline | Integrate into NuttX mainline, add `sim:microros` | Mainline build + `sim:microros` config |
| Phase 3 (Weeks 6–7) | Transport & Real-World Testing | Ensure runtime communication works; validate in real scenarios | Stable serial + UDP; tested end-to-end |
| Phase 4 (Weeks 8–9) | Multi-Board Support | Validate portability across boards | ESP32 + RP2040 + `sim:microros` |
| Phase 5 (Weeks 10–11) | Examples & Documentation | Improve usability & onboarding | Updated examples + tutorial |
| Phase 6 (Weeks 12–14) | CI & Regression Testing | NuttX mainline CI; if something breaks, everyone knows | CI in mainline + merged patches |

---

## Implementation Plan (Details)

### Phase 1: Establishing a Baseline

My first step will be to understand the current state instead of assuming anything works.

micro-ROS relies on a layered architecture where a lightweight DDS-XRCE client runs on the MCU and communicates with an agent that bridges it into the ROS 2 ecosystem.

Because of this, even small inconsistencies in the build system or OS integration can completely break the workflow.

So my approach here is simple:

- try building `micro_ros_nuttx_app` against current NuttX
- log every failure (build errors, missing headers)
- categorize issues into tags like build-system issues, API changes, and micro-ROS compatibility issues

At the same time, I’ll also look into Zephyr and FreeRTOS to study how their ports are structured, since they are actively maintained.

> **Outcome: A clear map of breakages and dependencies that will guide the later phases.**
> 

### Phase 2: Fixing the Build System & Mainline Integration

From my initial exploration, the biggest issue is that the build system is outdated relative to how NuttX is currently structured.

This phase will focus on aligning the integration with modern expectations and integrating directly into NuttX mainline:

- updating Makefiles and `Make.defs`
- fixing Kconfig integration
- updating `toolchain.cmake`
- adding proper CMake support (important since NuttX is moving toward it)
- contributing the build setup into the NuttX main repository (not as a separate external tree)
- creating a `sim:microros` board configuration that works with the NuttX simulator

The goal isn’t just to make it compile—it’s to ensure:

> the project builds the same way other modern NuttX apps do, and lives inside the mainline tree for full integration.
> 

### Phase 3: Transport Validation and Real-World Testing

Once the system builds, the next step is ensuring it actually works.

micro-ROS depends on transports (serial, UDP, etc.) to communicate with the agent using a client–server model. If this layer is unstable, the entire system breaks.

This phase includes:

- validating serial (TERMIOS) communication
- validating UDP transport
- reproducing known issues:
    - `arm_hardfault`
    - `DIR` redefinition

This will likely be the most debugging-heavy phase:

- memory constraints (micro-ROS is designed for very low RAM usage)
- stack overflows and alignment issues
- transport-level inconsistencies

**Real-world testing:** I will define and execute tests in realistic scenarios:

- **Simulator:** Using `sim:microros`, run the micro-ROS client against a micro-ROS Agent on the host (serial or UDP) and verify pub/sub and service flows end-to-end.
- **Hardware:** On STM32 (or another available board), connect MCU → Agent → ROS 2 host; publish topics from the MCU and consume them on the host.
- **Documented workflow:** A repeatable test procedure so anyone can verify the integration works in a real scenario.

> **Outcome: MCU ↔ Agent communication is stable and repeatable, with explicit real-world validation.**
> 

### Phase 4: Expanding Board Support

Once everything works on one board, the next step is generalization.

micro-ROS is designed to be portable across RTOSes and hardware platforms with pluggable OS and transport layers. So NuttX support should reflect that flexibility.

In this phase:

- `sim:microros` (simulator)—validate first so CI and development can run without hardware
- add support for ESP32 and RP2040
- test builds + basic functionality
- document board-specific quirks

My goal is not to cover every board, but to prove that the

> integration is not tied to a single setup.
> 

### Phase 5: Examples and Documentation

At this stage, things should work, but usability matters just as much.

micro-ROS already provides a consistent programming model across RTOSes, so ideally:

> using it on NuttX should feel just as easy as on Zephyr and FreeRTOS.
> 

This phase will focus on:

- updating the example app
- adding a pub/sub example and a service example
- rewriting outdated tutorials

> **Outcome: A new user should be able to see the data flowing without debugging internals when they follow the steps and run the example.**
> 

### Phase 6: CI, Regression Testing, and Long-Term Stability

One of the main reasons this integration fell behind is the lack of continuous validation.

This final phase ensures long-term sustainability by integrating CI into NuttX mainline:

- add a CI job inside the NuttX main repository that builds the micro-ROS configuration (e.g., `sim:microros`)
- if something stops working, everyone will know—regressions surface as CI failures for the whole project, not as silent breakage
- submit patches upstream (micro-ROS / NuttX mainline)
- ensure compatibility doesn’t silently break as NuttX or micro-ROS evolve

> **Outcome: Not just a working system, but a system that stays working—with regression visibility for the entire community.**
> 

I’m fully expecting some “interesting” behavior from these different boards, and I’m ready to get my hands dirty to make it all work seamlessly.

Instead of treating it as a rigid sequence, I’ll follow an iterative process: fix a layer, validate it, and then move forward. This ensures that each phase builds on something stable instead of stacking assumptions.

---

## Deliverables

By the end of this project, I expect to have:

### Code

- a fully updated `micro_ros_nuttx_app` that builds cleanly with current NuttX
- micro-ROS build support integrated into NuttX mainline (not as a separate external tree)
- a `sim:microros` board configuration for the NuttX simulator
- a modernized build system:
    - updated Makefiles and Kconfig
    - CMake support aligned with current NuttX workflows
- stable transport implementations:
    - serial (TERMIOS)
    - UDP
- fixes for known issues (including `arm_hardfault` and `DIR` redefinition, if reproducible)
- board support for:
    - **`sim:microros`**(simulator—enables CI and hardware-free development)
    - STM32 (baseline)
    - ESP32
    - RP2040

### Examples

- updated example application compatible with current NuttX APIs
- minimal publisher/subscriber example
- simple service example
- all examples tested with a working micro-ROS Agent setup

### Documentation

- updated micro-ROS NuttX tutorial (aligned with current versions)
- transport setup and debugging notes
- clear build + run instructions for supported boards (including `sim:microros`)
- real-world testing guide: step-by-step procedure to validate the integration in a real scenario (MCU + Agent + ROS 2 host)
- a short migration/summary guide explaining key changes

### Testing and Validation

- verified builds on multiple boards (including `sim:microros`)
- real-world scenario validation: documented end-to-end test (MCU ↔ Agent ↔ ROS 2 host)
- confirmed communication between MCU and agent (serial + UDP)
- basic functional validation of pub/sub and service flows
- CI in NuttX mainline that builds the micro-ROS configuration—if something stops working, everyone will know (regression visibility)

### Upstream Contributions

- patches submitted to micro-ROS / NuttX mainline repositories
- iteration based on maintainer feedback
- CI integrated into NuttX mainline to build the micro-ROS configuration (not a separate pipeline)

---

## Why this Project

What I find particularly interesting about this project is that it sits at the boundary between embedded systems and scaled distributed systems.

micro-ROS extends the ROS 2 ecosystem to microcontrollers using a lightweight DDS-XRCE-based client–server model. The device communicates with an external agent that represents it in the ROS 2 network, making it possible to integrate even highly constrained devices into larger systems.

NuttX provides the RTOS environment this model needs—predictable scheduling, a POSIX-like interface, and support for multiple communication mechanisms.

This project is interesting not just because of the individual components, but because of how they fit together.

Improving this integration means:

- making it easier to connect embedded nodes into ROS 2 systems
- reducing friction for developers choosing NuttX
- ensuring that a well-designed system is actually usable in practice

---

## About Me

I’m a third-year undergraduate at BITS Pilani Goa, studying Electronics and Instrumentation Engineering.

Over the past few months, I’ve focused on embedded systems and real-time software—specifically how low-level systems integrate with larger software stacks. I work comfortably in C and have hands-on experience with build systems like Make and CMake in embedded contexts.

What I enjoy most about this space is that it’s very hands-on—when something doesn’t work, you have to dig through layers (build system, OS, hardware), and when it finally works, it’s satisfying because it’s real.

---

## Why Me

I believe I’m well-suited for this project for several reasons.

First, I’m genuinely interested in the problem space—not just micro-ROS or NuttX individually, but the challenge of making different systems work together reliably.

Second, I’ve contributed to open-source projects in this ecosystem, including Apache NuttX and Apache Airflow. This experience has made me comfortable with real-world codebases, review processes, and collaborative development. Some of my contributions include:

### Apache Airflow

- https://github.com/apache/airflow/pull/61095
- https://github.com/apache/airflow/pull/60394
- https://github.com/apache/airflow/pull/60313
- https://github.com/apache/airflow/pull/60118
- https://github.com/apache/airflow/pull/60111
- https://github.com/apache/airflow/pull/60029

### Apache NuttX

- https://github.com/apache/nuttx/pull/18543
- https://github.com/apache/nuttx/pull/18521
- https://github.com/apache/nuttx/pull/18493
- https://github.com/apache/nuttx/pull/18347
- https://github.com/apache/nuttx/pull/18400
- https://github.com/apache/nuttx/pull/18426

### Apache NuttX-Apps

- https://github.com/apache/nuttx-apps/pull/3402

Third, I’m comfortable working through debugging-heavy problems. Much of this project will involve tracking down compatibility issues across layers—build system, RTOS APIs, and transports—and I’m prepared to iterate through that process.

Finally, I’m focused on maintainability, not just short-term fixes. The goal isn’t simply to get things working once, but to ensure they stay working as both NuttX and micro-ROS evolve.