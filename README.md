# GSoC Proposal: Apache NuttX

## Micro-ROS Integration on NuttX

---

- **Jira Issue:** [NUTTX-14](https://issues.apache.org/jira/browse/NUTTX-14)
- **GitHub Discussion:** [#18508](https://github.com/apache/nuttx/issues/18508)
- **Name:** Arjav Patel
- **Email:** [arjav1528@gmail.com](mailto:arjav1528@gmail.com)
- **GitHub:** [https://github.com/arjav1528](https://github.com/arjav1528)
- **Duration:** 12–14 weeks (~350 hours, large)

---

## Project Summary

This project aims to **revive and modernize Micro-ROS support on Apache NuttX**. [Micro-ROS](https://micro.ros.org) brings ROS 2 to microcontrollers—it uses a lightweight DDS-XRCE client on the MCU that communicates with an Agent on the host, bridging into the full ROS 2 network. Micro-ROS was originally developed on NuttX by Bosch and EU partners, and NuttX remains officially supported. However, after FreeRTOS and Zephyr support was added, the NuttX port stopped receiving attention and has fallen behind.

The work will focus on **bringing the `micro_ros_nuttx_app` codebase up to date** with current NuttX (master), fixing the build system (Make and CMake), validating and repairing transports (serial TERMIOS, UDP), resolving known open issues (e.g., `arm_hardfault` crash, `DIR` type redefinition), adding defconfigs for modern boards (STM32, ESP32, RP2040), improving examples and documentation, and establishing CI that builds against current NuttX.

---

## Background and Context

### Micro-ROS Architecture

Micro-ROS runs a **DDS-XRCE (Data Distribution Service for eXtremely Resource Constrained Environments)** client on the microcontroller. This client connects over a transport (typically serial/UART or UDP) to a **Micro-ROS Agent** on a host (Linux, etc.). The Agent acts as a bridge between the MCU and the full ROS 2 DDS network, enabling microcontrollers to publish/subscribe to topics, call services, and participate in the ROS 2 ecosystem.

### Current State of NuttX Support

- **Library layer:** The `micro_ros_lib` build files (`Makefile`, `colcon.meta`, `toolchain.cmake.in`) in `micro_ros_nuttx_app` are maintained—updated for rolling releases and bumped to Kilted (mid-2025). The library itself is actively developed.
- **NuttX integration:** The NuttX-specific glue has **not been updated since August 2021** (~4.5 years):
  - Top-level `Makefile`, `Make.defs`, and the entire `example_app/` (Kconfig, `Make.defs`, `microros_main.c`) have not been touched
  - No `CMakeLists.txt` for NuttX’s CMake build system
  - Tutorial still references NuttX 10.0.0 docs
  - CI only builds against NuttX 10.3 (not current master)
- **Known issues:** Two notable open issues in [micro_ros_nuttx_app](https://github.com/micro-ROS/micro_ros_nuttx_app): [#17](https://github.com/micro-ROS/micro_ros_nuttx_app/issues/17) (unresolved `arm_hardfault` crash, 2021) and [#31](https://github.com/micro-ROS/micro_ros_nuttx_app/issues/31) (Roberto Bucher’s modifications for NuttX RTOS, including Nucleo-144 with UDP and the `DIR` type redefinition bug).

---

## Problem Statements

- **Outdated build system:** The NuttX integration uses stale Makefiles, Kconfig, and `toolchain.cmake`. There is no support for NuttX’s CMake build, which is increasingly the default for new boards.
- **Unverified compatibility:** The last documented test was on NuttX 10.3. No systematic effort has been made to build or run `micro_ros_nuttx_app` against current NuttX master, so breakages are undocumented.
- **Transport reliability:** Serial (TERMIOS) and UDP transports have not been validated on recent NuttX; the `arm_hardfault` and `DIR` redefinition issues suggest deeper compatibility problems.
- **Limited board coverage:** Defconfigs and examples primarily target older STM32 boards. Modern, popular platforms (ESP32, RP2040) lack documented support.
- **Documentation gap:** Tutorials and docs reference old NuttX versions; there is no up-to-date guide for building and running Micro-ROS on current NuttX.

---

## Proposed Solution (High Level)

1. **Establish a baseline**
  - Build `micro_ros_nuttx_app` against current NuttX master and document every breakage. Use Zephyr and FreeRTOS ports as references for expected behavior and structure.
2. **Fix the build system**
  - Update Makefiles, `Make.defs`, Kconfig, and `toolchain.cmake` so the app builds cleanly. Add `CMakeLists.txt` for NuttX’s CMake build system.
3. **Validate and fix transports**
  - Test serial (TERMIOS) and UDP transports on current NuttX. Investigate and fix the `arm_hardfault` and `DIR` type redefinition issues.
4. **Expand board support**
  - Add defconfigs and test on ESP32 and RP2040, in addition to STM32.
5. **Improve examples and documentation**
  - Update the example app, add pub/sub and service examples, and update the Micro-ROS NuttX tutorial for current NuttX.
6. **CI and long-term maintenance**
  - Improve CI to build against current NuttX master and ensure future changes do not regress NuttX support.

---

## Implementation Steps (How I Will Solve It)

### Phase 1: Baseline and Orientation (Weeks 1–2)


| Step | Task                                                                                              | Deliverable                                     |
| ---- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| 1.1  | Attempt baseline build of `micro_ros_nuttx_app` against NuttX master; log all errors and warnings | Baseline build report with documented breakages |
| 1.2  | Study Zephyr and FreeRTOS Micro-ROS ports (build layout, transport glue, example structure)       | Notes on structure and patterns to adopt        |
| 1.3  | Sync with mentors on scope and priorities (mailing list / GitHub issue)                           | Agreed baseline and phasing                     |


### Phase 2: Build System Fixes (Weeks 3–5)


| Step | Task                                                                                            | Deliverable                               |
| ---- | ----------------------------------------------------------------------------------------------- | ----------------------------------------- |
| 2.1  | Update top-level Makefile, `Make.defs`, and Kconfig to work with current NuttX                  | Patched `micro_ros_nuttx_app` build files |
| 2.2  | Update `toolchain.cmake.in` and library build configuration for current toolchain/NuttX headers | Patched `micro_ros_lib` build             |
| 2.3  | Add `CMakeLists.txt` for NuttX CMake build system                                               | New CMake integration                     |
| 2.4  | Achieve first clean build on at least one STM32 board                                           | Working build + defconfig                 |
| 2.5  | Run nxstyle and fix any style issues                                                            | Clean, compliant patches                  |


### Phase 3: Transport Validation and Bug Fixes (Weeks 6–7)


| Step | Task                                                                | Deliverable                         |
| ---- | ------------------------------------------------------------------- | ----------------------------------- |
| 3.1  | Test serial (TERMIOS) transport on STM32; fix issues                | Verified serial transport           |
| 3.2  | Test UDP transport; reproduce and fix `DIR` redefinition if present | Verified UDP transport              |
| 3.3  | Investigate `arm_hardfault` issue (reproduce, isolate, propose fix) | Analysis + patch or upstream report |
| 3.4  | Document transport usage and any NuttX-specific notes               | Transport documentation             |


### Phase 4: Multi-Board Support (Weeks 8–9)


| Step | Task                                                      | Deliverable                           |
| ---- | --------------------------------------------------------- | ------------------------------------- |
| 4.1  | Add defconfig for ESP32 (or equivalent)                   | ESP32 defconfig + build instructions  |
| 4.2  | Add defconfig for RP2040 (or equivalent)                  | RP2040 defconfig + build instructions |
| 4.3  | Test Micro-ROS Agent connectivity for at least two boards | Verification report                   |


### Phase 5: Examples and Documentation (Weeks 10–11)


| Step | Task                                                          | Deliverable             |
| ---- | ------------------------------------------------------------- | ----------------------- |
| 5.1  | Update existing example app to current NuttX API and patterns | Updated `example_app/`  |
| 5.2  | Add pub/sub example (e.g., publisher and subscriber nodes)    | New example(s)          |
| 5.3  | Add service example (client and/or server)                    | New example(s)          |
| 5.4  | Update Micro-ROS NuttX tutorial for current NuttX             | Revised tutorial / docs |


### Phase 6: CI and Upstream (Weeks 12–14)


| Step | Task                                                              | Deliverable                    |
| ---- | ----------------------------------------------------------------- | ------------------------------ |
| 6.1  | Update or add CI job to build against NuttX master                | CI configuration change        |
| 6.2  | Prepare summary document (changes, migration notes for users)     | Summary document               |
| 6.3  | Submit patches to micro-ROS and/or NuttX; address review feedback | Patches submitted and iterated |


---

## Deliverables

- **Code:** Patched `micro_ros_nuttx_app` (Makefiles, `Make.defs`, Kconfig, `toolchain.cmake`), new `CMakeLists.txt`, updated `example_app/`, new pub/sub and service examples, defconfigs for STM32, ESP32, RP2040 (or equivalent boards).
- **Documentation:** Updated Micro-ROS NuttX tutorial, transport notes, and migration/summary document for current NuttX.
- **Testing:** Verified builds on at least 3 boards; validated serial and UDP transports; CI building against NuttX master.
- **Upstream:** Patches submitted to micro-ROS and/or NuttX mailing lists / GitHub, with review feedback addressed.

---

## Resources

- [micro_ros_nuttx_app](https://github.com/micro-ROS/micro_ros_nuttx_app) – NuttX application repository
- [Micro-ROS](https://micro.ros.org) – Project website and documentation
- [Micro-ROS NuttX Tutorial](https://micro.ros.org/docs/tutorials/core/first_application_rtos/nuttx/) – First micro-ROS application on NuttX (to be updated)
- [DDS-XRCE Specification](https://www.omg.org/spec/DDS-XRCE/1.0/About-DDS-XRCE) – Protocol reference (OMG, v1.0)

---

## About Me

**Relevant Experience:** I have been exploring embedded systems and real-time software. I am familiar with C, build systems (Make, CMake), and RTOS concepts. I have started contributing to the NuttX ecosystem and opened the discussion issue [#18508](https://github.com/apache/nuttx/issues/18508) for this project to gather feedback and align with mentors.

**Why This Project:** Micro-ROS brings ROS 2’s robotics middleware to resource-constrained devices, and NuttX is a mature, POSIX-compliant RTOS used in real IoT and robotics products. Restoring and modernizing the NuttX port will help robotics developers choose NuttX without sacrificing ROS 2 integration. I want to contribute to a project that sits at the intersection of embedded systems and robotics, where clarity, compatibility, and maintainability matter. I am motivated to learn from the micro-ROS and NuttX communities and to produce work that remains useful beyond GSoC.