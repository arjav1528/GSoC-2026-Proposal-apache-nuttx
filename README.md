# GSoC Proposal 2026 : Arjav Patel

# Micro-ROS Integration on NuttX

**Name :** Arjav Patel

**Degree :** B.E. Electronics & Instrumentation Engineering

**College :** BITS Pilani Goa Campus

**Email :** [arjav1528@gmail.com](mailto:arjav1528@gmail.com)

**GitHub :** https://github.com/arjav1528

**Issue Thread :** https://github.com/apache/nuttx/issues/18508

**JIRA Issue :** https://issues.apache.org/jira/browse/NUTTX-14

**Potential Mentors :** Alan Carvalho de Assis ([acassis@apache.org](mailto:acassis@apache.org))

---

## **A Quick Look at NuttX (and Why It Matters)**

**Apache NuttX** is one of the RTOS that we don’t usually see in textbooks; it actually shows up in real systems. It is specially designed for constrained hardware, but at same time, it tries to give you a dev experience that is very much familiar with Unix-like systems.

This is what makes it interesting.

On the other hand, you get real-time guarantees and the ability to run it on **microcontrollers.** Further, you still have POSIX-like APIs, a clean structure, and a fairly clean architecture. With the help of this, instead of writing everything from scratch, you can build structured applications even with constrained hardware.

From what I’ve learnt and understood, this balance is the reason why NuttX is used in places where it actually matters, whether it’s IoT platforms or even space systems. Today, its not just about “running on small devices”, but doing that in a way that remains maintainable as system grows.

Because of this, NuttX improvements have an real impact. If anything is missing, deprecated, it would directly affect the devs trying to build applications on top of NuttX.

---

## **Why Micro-ROS Belongs on NuttX**

If you look at how robotics are structured today, ROS2 is almost somewhere in the picture, handling communication between the sensors, controllers, and higher-level logic.

The main problem is that ROS2, by default, doesnt extend all the way down to microcontrollers. That’s where [micro-ROS](https://micro.ros.org/) comes in, bringing a ROS2 comaptible programming model to resource constrained devices, such that even small MCU can participate in large ROS2.

micro-ROS is designed in such a way to run on top of an RTOS, & this is where NuttX comes into picture.

NuttX already provides many of the things that micro-ROS expects from an RTOS:

- a strong POSIX-style API surface
- deterministic scheduling and real-time behaviour
- support for networking and communication stacks
- and the ability to scale across different microcontroller platforms

This is exactly the kind of environment where NuttX is often used: embedded control nodes that need to be predictable and reliable.

By the way, Micro-ROS already supports RTOS like FreeRTOS, Zephyr, and NuttX, and the core API remain largely consistent across them.

So my project, which is **integrating micro-ROS to NuttX**, is not about reinventing anything; its about bringing it to same level of usability and stability as other RTOS.

---

## Project Summary

At very high level, this project brings micro-ROS on NuttX back to a state where it actually feels usable with current eco-system.

Micro-ROS itself is in its good place, as it follows ROS2 architecture, resuses a significant amount of its stack and uses DDS-XRCE as lightweight middleware to make ROS2 communication possible on microcontrollers. The design is simple yet effective: a compact local client connects to a specialized agent, which acts as a gateway into the complete ROS 2 network.

My assumption behind this design is that there is a capable RTOS underneath that provides basic POSIX-like abstractions, deterministic, scheduling, and communication support. That’s exactly why micro-ROS officially supports ROTSes like FreeRTOS, Zephyr and NuttX.

However, FreeRTOS and Zehpyr integrations have kept evolving; the NuttX side hasnt seen enough maintenance. But over that gap, micro-ROS stack is up-to-date, but the NuttX specific integration i.e, build system, examples, transports has fallen behind.

From what I’ve read so far, the issue is not that the integration is fundamentally broken, but it’s that it has drifted out of sync with both NuttX and micro-ROS as they evolved independently.

**So the goal here is not to redesign anything from scratch.**

Instead, the goal is to carefully update the integration layer to align with the current state of both ecosystems:

- ensure it builds cleanly against current NuttX
- verify that transports like serial and UDP work reliably
- resolve long-standing issues that block usability
- extend support to more relevant, modern boards

An important part of this work is making the system approachable again. Micro-ROS already provides a consistent programming model across RTOSes, so using it on NuttX should feel as straightforward as on Zephyr or FreeRTOS.

I also want to ensure this doesn't become outdated again in a few months. That means adding CI support and aligning the structure with how other maintained ports are organized, so future updates don't silently break NuttX support.

In short, this project is about restoring alignment—between micro-ROS, NuttX, and the expectations developers have when they use them together.

---

## Implementation Plan (Overview)

| ***Phase*** | ***Focus Area*** | ***Key Goals*** | ***Deliverables*** |
| --- | --- | --- | --- |
| Phase 1 (Week 1-2) | Baseline & Analysis | Understand current breakages | Build report + issue mapping |
| Phase 2 (Week 3-5) | Build System Fixes | Make Project compile on current NuttX | Updated Make + CMake build |
| Phase 3 (Week 6-7) | Transport & Debugging | Ensure runtime communication works | Stable serial + UDP transaport |
| Phase 4 (Week 8-9) | Multi-Board Support | Validate portability across boards | ESP32 + RP2040 |
| Phase 5 (Week 10-11) | Example & Documentation | Improve usablility & onboarding | Updated examples + tutorial |
| Phase 6 (Week 12-14) | CI & Upstreaming | Ensure Long-term stability | CI + Merged Patches |

---

## Implementation Plan (Details)

### Phase 1 : Establishing a Baseline

My first step will be to understand the current state instead of assuming anything works.

Micro-ROS relies on a layered architecture where a lightweight DDS-XRCE client runs on the MCU and communicates with an agent that bridges it into the ROS 2 ecosystem.

Because of this, even small inconsistencies in the build system or OS integration can completely break the workflow.

So my approach here is simple:

- try building `micro_ros_nuttx_app` against current NuttX
- log every failure (build errors, missing headers)
- categorise issues into tags like build system issues, API changes, and micro-ROS compatibility issues

At the same time, I’ll also look up into **Zephyr** & **FreeRTOS** to study how thier ports are structured, as they are currently actively maintained.

> **Outcome : A clear map of breakages and dependencies which will guide me for my further phases**
> 

### Phase 2 : Fixing the Build System

From my initial exploration, the biggest issue is that the build system is outdated relative to how NuttX is currently structured.

This phase will focus on aligning the integration with modern expectations:

- updating Makefiles and `Make.defs`
- fixing Kconfig integration
- updating `toolchain.cmake`
- adding proper **CMake support** (important since NuttX is moving toward it)

The goal isn't just to make it compile—it's to ensure:

> the project builds the same way other modern NuttX apps do.
> 

### Phase 3 : Transport Validation and Debugging

Once the system builds, the next step is ensuring it actually works.

Micro-ROS depends on transports (serial, UDP, etc.) to communicate with the agent using a client–server model. If this layer is unstable, the entire system breaks.

This phase includes:

- validating **serial (TERMIOS)** communication
- validating **UDP transport**
- reproducing known issues:
    - `arm_hardfault`
    - `DIR` redefinition

This will likely be the most debugging-heavy phase:

- memory constraints (micro-ROS is designed for very low RAM usage)
- stack overflows and alignment issues
- transport-level inconsistencies

> **Outcome : MCU ↔ Agent communication is stable and repeatable**
> 

### Phase 4: Expanding Board Support

Once everything is on one board, next step would is generalization.

Micro-ROS is designed to be portable across ROTS and hardware platforms with plauggable OS and transport layers. So NuttX support should reflect that flexibility.

In this phase:

- add support for ESP32 and RP2040
- test build + basic functionality
- document the board specific quirks

My goal is not covering every board, but proving that

> **integration is not tied to a single setup**
> 

### Phase 5: Examples and Documentation

At this stage, the things should work, but obviously usability matters just as much.

Micro-ROS already provides a consistent programming model across RTOSes, so ideally:

> **using it on NuttX should feel just as easy as on Zephyr and FreeRTOS**
> 

This phase will focus on:

- updating the example app
- adding pub/sub example and service example
- rewriting outdated tutorials

> **Outcome: A new user should be able to see the data flowing without debugging ginternals when he/she follows the steps and run the example**
> 

### Phase 6: CI and Long-Term Stability

One of the main reasons this integration fell behind is the lack of continuous validation.

This final phase ensures long-term sustainability:

- set up CI to build against current NuttX
- ensure compatibility doesn't silently break
- submit patches upstream (micro-ROS / NuttX)

> **Outcome: Not just a working system, but a system that stays working.**
> 

I’m fully expecting some 'interesting' behaviour from these different boards, and I’m ready to get my hands dirty to make it all work seamlessly.

So instead of treating it as a rigid sequence, I’ll follow an iterative process which includes fixing a layer, then validating and then moving forward. This ensures that each phase builds on something stable instead of stacking assumptions.

---

## Deliverables

By the end of this project, I expect to have:

### Code

- A fully updated `micro_ros_nuttx_app` that builds cleanly with current NuttX
- Modernized build system:
    - Updated Makefiles and Kconfig
    - CMake support aligned with current NuttX workflows
- Stable transport implementations:
    - Serial (TERMIOS)
    - UDP
- Fixes for known issues (including `arm_hardfault` and `DIR` redefinition, if reproducible)
- Board support for:
    - STM32 (baseline)
    - ESP32
    - RP2040

### Examples

- Updated example application compatible with current NuttX APIs
- Minimal **publisher/subscriber example**
- Simple **service example**
- All examples tested with a working Micro-ROS Agent setup

### Documentation

- Updated Micro-ROS NuttX tutorial (aligned with current versions)
- Transport setup and debugging notes
- Clear build + run instructions for supported boards
- A short migration/summary guide explaining key changes

### Testing and Validation

- Verified builds on multiple boards
- Confirmed communication between MCU and agent (serial + UDP)
- Basic functional validation of pub/sub and service flows

### Upstream Contributions

- Patches submitted to micro-ROS / NuttX repositories
- Iteration based on maintainer feedback
- CI setup to build against current NuttX (to avoid future drift)

---

## Why this Project

What I find particularly intresting about this project is that it sits at the boundary between embedded systems and scaled distributed systems.

Micro-ROS extends the ROS 2 ecosystem to microcontrollers using a lightweight DDS-XRCE-based client–server model. The device communicates with an external agent that represents it in the ROS 2 network, making it possible to integrate even highly constrained devices into larger systems.

NuttX provides the RTOS environment this model needs—predictable scheduling, a POSIX-like interface, and support for multiple communication mechanisms.

This project is interesting not just because of the individual components, but because of how they fit together.

Improving this integration means:

- making it easier to connect embedded nodes into ROS 2 systems
- reducing friction for developers choosing NuttX
- ensuring that a well-designed system is actually usable in practice

---

## About Me

I'm a third-year undergraduate at BITS Pilani Goa, studying Electronics and Instrumentation Engineering.

Over the past few months, I've focused on embedded systems and real-time software—specifically how low-level systems integrate with larger software stacks. I work comfortably in C and have hands-on experience with build systems like Make and CMake in embedded contexts.

What I enjoy most about this space is that it’s very hands-on — when something doesn’t work, you have to dig through layers (build system, OS, hardware), and when it finally works, it’s very satisfying because it’s real.

---

## Why Me

I believe I'm well-suited for this project for several reasons.

First, I'm genuinely interested in the problem space—not just Micro-ROS or NuttX individually, but the challenge of making different systems work together reliably.

Second, I've contributed to open-source projects in this ecosystem, including Apache NuttX and Apache Airflow. This experience has made me comfortable with real-world codebases, review processes, and collaborative development. Some of my contributions include:

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

Third, I'm comfortable working through debugging-heavy problems. Much of this project will involve tracking down compatibility issues across layers—build system, RTOS APIs, and transports—and I'm prepared to iterate through that process.

Finally, I'm focused on **maintainability**, not just short-term fixes. The goal isn't simply to get things working once, but to ensure they stay working as both NuttX and Micro-ROS evolve.