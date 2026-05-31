# theoOS — Project Specification

> **Transparency notice**
> This specification was drafted with the assistance of Claude Sonnet 4.6 (Anthropic) based on a structured conversation about the project goals, background, and roadmap. The architecture decisions, learning objectives, and implementation choices are the author's own. All code will be written by hand, without AI code generation.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Author Background & Motivation](#2-author-background--motivation)
3. [Hardware Target](#3-hardware-target)
4. [Goals & Non-Goals](#4-goals--non-goals)
5. [Repository Structure](#5-repository-structure)
6. [Git Workflow](#6-git-workflow)
7. [Milestones & Phases](#7-milestones--phases)
8. [Phase Details](#8-phase-details)
9. [Coding Philosophy](#9-coding-philosophy)
10. [Resources](#10-resources)

---

## 1. Project Overview

**theoOS** is a bare-metal mini-OS written in C, targeting the STM32H750B-DK Discovery Kit (STM32H750XIH6, ARM Cortex-M7). It is built entirely from scratch — no HAL, no CubeMX, no RTOS library — for the explicit purpose of deeply understanding how computers work at the lowest level.

The project starts from nothing (startup code, linker script, toolchain) and incrementally builds toward a functional kernel capable of running multiple tasks. The final objective is to use this OS as the actual runtime for personal embedded projects (IoT, sensors, automation).

This is a **solo, long-term, learning-first project**. Deadlines are indicative. Progress is non-linear. What matters is that every box gets checked eventually, with genuine understanding.

**Start date:** 01 June 2026
**Estimated completion:** May 2027 (at ~5 hours/week)

---

## 2. Author Background & Motivation

- M2 MIAGE IA (Université Côte d'Azur) + exchange semester at Université Laval (Québec) — Robotics, Computer Vision, Software Architecture
- 3 years of alternance: 1 year fullstack (Java/Angular), 2 years backend (C++)
- Embedded side projects: ESP32 (FFT/LED), Raspberry Pi dashcam (OpenCV/OCR), Arduino aquaponics
- Honest assessment: previous embedded work involved significant copy-paste and AI-assisted code; this project is a deliberate reset to learn things properly

**Motivation:** Transition toward a Graduate/Junior Embedded Software Engineer role (target market: UK — London, Cambridge). Building this OS is both a deep learning exercise and a concrete portfolio artifact.

---

## 3. Hardware Target

| Item | Details |
|------|---------|
| Board | STM32H750B Discovery Kit (STM32H750B-DK) |
| MCU | STM32H750XIH6 |
| Core | ARM Cortex-M7 @ up to 480 MHz |
| Architecture | ARMv7E-M / Thumb-2 + DSP + FPU (double precision) |
| RAM | 1 MB internal SRAM (+ 128 MB external SDRAM on board) |
| Flash | 128 KB internal flash (+ 128 MB external QSPI flash) |
| Onboard peripherals | LCD display, camera connector, microphone, Ethernet, USB HS, SD card |
| Debug interface | ST-Link V3 (on-board) |
| Flash tool | OpenOCD |
| Compiler | `arm-none-eabi-gcc` |
| Build system | GNU Make (handwritten Makefiles, no CMake) |

> **Note on flash:** The STM32H750 has only 128 KB of internal flash, which is small by design — ST expects code to run from external QSPI flash (XIP mode). Phase 1 will include understanding this constraint and deciding the boot/execution strategy (internal flash only for the kernel, or XIP from the start).

> **Note on the display:** The onboard LCD is explicitly a non-goal for v1. It may become a Phase 5 stretch goal if the core OS is solid.

---

## 4. Goals & Non-Goals

### Goals

- Write a working ARM startup sequence (vector table, Reset_Handler) from scratch
- Control GPIO, UART, and interrupts by writing directly to registers — no HAL
- Implement a heap allocator from scratch
- Implement cooperative then preemptive multitasking with a context switch via PendSV
- Build a round-robin scheduler driven by SysTick
- Implement basic synchronisation primitives (mutex, semaphore)
- Add a driver abstraction layer (open/read/write/close interface)
- Implement inter-task communication (message queues, event flags)
- Build a minimal UART shell for runtime inspection
- Port at least one real personal project onto the OS as a task

### Non-Goals

- MPU (Memory Protection Unit) usage — the Cortex-M7 has an MPU but configuring it is out of scope for v1
- File system
- USB or Ethernet stack
- Graphical interface
- POSIX compliance
- Supporting multiple MCU families in v1

---

## 5. Repository Structure

```
theoos/
├── SPEC.md                  # This file
├── README.md                # Project overview + build instructions
├── Makefile                 # Top-level build entry point
├── linker/
│   └── stm32h750.ld         # Linker script (written by hand)
├── startup/
│   └── startup.c            # Vector table + Reset_Handler
├── kernel/
│   ├── scheduler.c/h        # Round-robin scheduler
│   ├── task.c/h             # Task control blocks + context switch
│   ├── mutex.c/h            # Mutex primitive
│   ├── semaphore.c/h        # Semaphore primitive
│   ├── queue.c/h            # Message queue
│   └── timer.c/h            # Software timers
├── drivers/
│   ├── driver.h             # Generic driver interface (open/read/write/close)
│   ├── gpio.c/h
│   ├── uart.c/h
│   └── systick.c/h
├── lib/
│   └── heap.c/h             # Custom malloc/free
├── shell/
│   └── shell.c/h            # UART command shell
├── apps/
│   └── <project>/           # Real project ported onto the OS (Phase 5)
└── docs/
    ├── architecture.md      # Architecture decisions and diagrams
    ├── context_switch.md    # Explanation of context switch implementation
    └── references.md        # Annotated reading list
```

---

## 6. Git Workflow

### Branch prefix convention

Branch names use a prefix separated by `/` to categorize the type of work at a glance. Git ignores the prefix entirely — it is purely a human convention for navigating the repo on GitHub.

| Prefix | Meaning | Example |
|--------|---------|---------|
| `phase/` | Long-running branch for an entire phase — may live for weeks | `phase/bare-metal` |
| `feat/` | A single, specific feature being added | `feat/gpio-registers` |
| `fix/` | A bug fix | `fix/stack-alignment` |
| `docs/` | Documentation only, no code change | `docs/context-switch-notes` |
| `chore/` | Cleanup, tooling, no functional change | `chore/makefile-cleanup` |

The typical flow: create a `feat/` branch off the current `phase/` branch → work on it → open a PR back into the `phase/` branch (PR description = learning note) → merge when working → merge the `phase/` branch into `main` only when the full phase is complete.

### Visualising progress

**GitHub Projects (Board or Roadmap view)** is the recommended way to get an overview of all phases and their completion. It connects directly to milestones and issues, giving a Kanban-style view (To do / In progress / Done) or a timeline. Takes ~10 minutes to set up and requires no external tools.

The **Insights → Network** tab on GitHub also shows a visual graph of all branches and merges, which is useful for reviewing history but less useful for tracking current progress.

### Commit conventions

Commits follow a simple prefix convention:

```
feat: implement SysTick IRQ handler
fix: correct stack alignment in context switch
docs: add explanation of PendSV usage
refactor: extract UART register macros to header
chore: update Makefile for new linker script path
```

### GitHub Milestones & Issues

Each phase maps to a **GitHub Milestone** with a target date. Within each milestone, checklist items from this spec are opened as **Issues** tagged with the phase label. This is not bureaucracy — it is a searchable history of what was learned and when.

Labels used: `phase-1` through `phase-5`, `blocked`, `learning-note`, `hardware`.

---

## 7. Milestones & Phases

| # | Phase | Start | Target end | GitHub Milestone |
|---|-------|-------|------------|-----------------|
| 1 | Fondations C & outillage | 01 Jun 2026 | 13 Jul 2026 | `milestone/phase-1` |
| 2 | Bare-metal STM32 from scratch | 14 Jul 2026 | 21 Sep 2026 | `milestone/phase-2` |
| 3 | Kernel minimal | 22 Sep 2026 | 11 Jan 2027 | `milestone/phase-3` |
| 4 | Services OS (drivers & IPC) | 12 Jan 2027 | 22 Mar 2027 | `milestone/phase-4` |
| 5 | Porter un projet réel | 23 Mar 2027 | 31 May 2027 | `milestone/phase-5` |

> Dates are estimates based on ~5 hours/week. Phases may overlap or slip. What matters is completing every task, not hitting every date.

---

## 8. Phase Details

### Phase 1 — Fondations C & outillage
**Branch:** `phase/foundations` | **Duration:** 6 weeks

The goal is not to learn C from scratch (that happens in parallel, self-directed). The goal is to have a working, understood toolchain before touching the hardware.

#### Tasks

**1.1 — Toolchain setup** (`feat/toolchain-setup`)
- Install `arm-none-eabi-gcc`, `arm-none-eabi-gdb`, `openocd`, `make`
- Verify the full chain: write a minimal `main.c`, compile it for ARM, inspect the ELF output
- Document every flag used in the Makefile and why it exists
- Deliverable: a Makefile that compiles a "hello ARM" binary with no warnings

**1.2 — Linker script & memory map** (`feat/linker-script`)
- Read the memory map section of the STM32 Reference Manual
- Write a minimal `.ld` linker script defining FLASH and SRAM regions
- Understand `.text`, `.data`, `.bss` sections and how `.data` is copied from flash to RAM at startup
- Deliverable: a linker script that places code and data correctly, verified with `arm-none-eabi-objdump`

**1.3 — ELF & binary inspection** (`feat/binary-anatomy`)
- Use `objdump`, `readelf`, `nm`, `size` to inspect the output binary
- Understand the difference between the ELF file and the raw binary flashed to the chip
- Deliverable: a `docs/architecture.md` entry explaining what is in the binary and where it lives in memory

**Phase 1 done when:** Toolchain compiles, links, and produces a correct ELF. Every Makefile flag is understood and documented.

---

### Phase 2 — Bare-metal STM32 from scratch
**Branch:** `phase/bare-metal` | **Duration:** 10 weeks

No HAL. No CubeMX. Every register write is done with the address taken directly from the Reference Manual.

#### Tasks

**2.1 — Startup code & vector table** (`feat/startup-vector-table`)
- Write `startup.c` containing the vector table as a C array of function pointers
- Implement `Reset_Handler`: copy `.data` from flash to SRAM, zero-fill `.bss`, call `main()`
- Understand the initial stack pointer value and how it is placed at vector table offset 0
- Deliverable: board boots to `main()` with correct memory layout, verified via debugger

**2.2 — GPIO from registers** (`feat/gpio-registers`)
- Read the GPIO and RCC chapters of the Reference Manual
- Enable the GPIO clock via RCC registers
- Configure a GPIO pin as output and toggle the onboard LED
- Write a `gpio.h` defining register addresses as typed macros (no magic numbers in `.c` files)
- Deliverable: LED blinks at a fixed interval, zero HAL

**2.3 — SysTick & NVIC** (`feat/systick-irq`)
- Configure SysTick to fire at 1 kHz (1 ms tick)
- Write a `SysTick_Handler` that increments a global tick counter
- Understand NVIC priority registers and how the Cortex-M exception model works
- Deliverable: a `get_tick()` function used to implement a busy-wait `delay_ms()`

**2.4 — UART from registers** (`feat/uart-registers`)
- Configure USART2 (or equivalent on your Nucleo) using only register writes
- Implement `uart_putc()`, `uart_puts()`, and redirect `printf()` via `_write` syscall stub
- Deliverable: "Hello from NucleoOS" appears on a serial terminal at the correct baud rate

**2.5 — ARM call stack & Thumb-2** (`feat/arm-architecture-notes`)
- Read relevant chapters of the ARM Cortex-M Definitive Guide (Joseph Yiu)
- Understand the register file: r0–r12, SP (r13), LR (r14), PC (r15), xPSR
- Understand PSP vs MSP, and how exception entry/exit saves/restores registers
- Deliverable: a `docs/context_switch.md` entry explaining these concepts in your own words — written before implementing the context switch

**Phase 2 done when:** Board boots from custom startup code, LED blinks, UART prints, SysTick fires. Every register write is traceable to a page in the Reference Manual.

---

### Phase 3 — Kernel minimal
**Branch:** `phase/kernel` | **Duration:** 15 weeks

This is the hardest phase. The context switch alone may take several weeks to debug. That is expected and normal.

#### Tasks

**3.1 — Heap allocator** (`feat/heap-allocator`)
- Implement a first-fit allocator: `heap_init()`, `heap_alloc()`, `heap_free()`
- Use a linked list of free blocks; track metadata (size, next pointer) in block headers
- Write a test harness that allocates, frees, and checks for memory corruption
- Read: OSTEP chapter "Free Space Management"
- Deliverable: allocator passes all test cases, no memory leaks, metadata verified with `printf`

**3.2 — Task control blocks** (`feat/task-structs`)
- Define `task_t`: stack pointer, stack base, stack size, state (READY/RUNNING/BLOCKED), task function pointer
- Write `task_create()`: allocates a stack, sets up initial stack frame so the task starts correctly on first context switch
- Understand the exact layout of the initial stack frame expected by the Cortex-M exception return mechanism
- Deliverable: two static tasks defined, their stacks inspectable via debugger

**3.3 — Context switch via PendSV** (`feat/context-switch`)
- Implement `PendSV_Handler` in C (with `__attribute__((naked))`) or minimal assembly
- Save/restore r4–r11 (callee-saved registers not auto-saved by the CPU)
- Switch the PSP to the next task's stack
- Trigger the switch from SysTick by pending PendSV
- Deliverable: two tasks run in alternation, each printing its name over UART

**3.4 — Round-robin scheduler** (`feat/scheduler-round-robin`)
- Implement a scheduler that cycles through READY tasks
- Add priority levels (optional but recommended for CV value)
- Handle the idle task (runs when all other tasks are blocked)
- Deliverable: three tasks with different blink rates all running concurrently

**3.5 — Mutex** (`feat/mutex`)
- Implement `mutex_t`, `mutex_lock()`, `mutex_unlock()`
- Handle the case where a task tries to lock an already-locked mutex: move it to BLOCKED state
- Implement priority inheritance (optional stretch goal)
- Read: OSTEP chapter "Locks"
- Deliverable: two tasks share a UART mutex without garbled output

**3.6 — Semaphore** (`feat/semaphore`)
- Implement counting semaphore: `sem_init()`, `sem_wait()`, `sem_post()`
- Deliverable: producer/consumer demo over UART

**Phase 3 done when:** Multiple tasks run, scheduler switches between them, mutex prevents concurrent access. This is a functional mini-RTOS.

---

### Phase 4 — Services OS (drivers & IPC)
**Branch:** `phase/services` | **Duration:** 10 weeks

With the kernel working, this phase adds the infrastructure that makes the OS usable for real projects.

#### Tasks

**4.1 — Driver interface abstraction** (`feat/driver-interface`)
- Define a generic driver interface in `driver.h`: `open()`, `read()`, `write()`, `close()`, `ioctl()`
- Refactor existing GPIO and UART implementations to conform to this interface
- Deliverable: LED and UART accessible through the same `driver_t *` abstraction

**4.2 — SPI driver** (`feat/driver-spi`)
- Implement SPI using registers, behind the driver interface
- Needed for future projects (sensors, peripherals)
- Deliverable: SPI transfer verified with a logic analyser or loopback test

**4.3 — Message queues** (`feat/ipc-queue`)
- Implement `queue_t`: circular buffer, `queue_send()`, `queue_receive()`
- Blocking variants: task blocks if queue is full (send) or empty (receive)
- Deliverable: two tasks communicate via a queue without shared globals

**4.4 — Event flags** (`feat/ipc-events`)
- Implement event flag groups: `event_set()`, `event_wait()` with AND/OR semantics
- Deliverable: an IRQ sets an event flag; a task wakes on it

**4.5 — Software timers** (`feat/soft-timers`)
- Implement `timer_create()`, `timer_start()`, `timer_stop()` backed by SysTick
- One-shot and periodic modes
- Deliverable: a periodic callback fires every 500 ms without blocking any task

**4.6 — UART shell** (`feat/uart-shell`)
- Line editor over UART: backspace, echo, newline handling
- Built-in commands: `tasks` (list all tasks + state + stack usage), `mem` (heap stats), `regs` (dump key registers)
- Deliverable: interactive shell accessible from any serial terminal at 115200 baud

**Phase 4 done when:** Drivers are abstracted, tasks communicate via queues and events, timers fire on schedule, and the shell provides runtime visibility into the system.

---

### Phase 5 — Porter un projet réel sur l'OS
**Branch:** `phase/real-project` | **Duration:** 10 weeks

The OS proves its worth by running a real application. This is also the deliverable for the UK portfolio.

#### Tasks

**5.1 — Choose and port a project** (`feat/app-<name>`)
- Candidates: aquaponics controller (sensors + actuators + scheduling), LED sync (FFT audio processing as a task), or a new project designed specifically for NucleoOS
- Rewrite the application as one or more OS tasks using the driver interface and IPC primitives
- The application must not bypass the OS (no direct register writes from app code)
- Deliverable: the application runs stably for >1 hour without crash or memory leak

**5.2 — README & architecture documentation** (`docs/readme-final`)
- `README.md`: what the OS is, how to build it, how to flash it, how to write a new task
- `docs/architecture.md`: full architecture diagram, explanation of scheduler, context switch, memory layout
- `docs/context_switch.md`: step-by-step explanation of the context switch implementation
- All documentation written for a reader who is a competent embedded engineer but unfamiliar with this codebase
- Deliverable: a stranger can build, flash, and understand NucleoOS from the docs alone

**5.3 — Demo video + technical write-up** (`docs/demo`)
- Short video (2–3 min): boot sequence, shell, tasks running, real application in action
- One technical article (LinkedIn or personal blog) on a specific concept implemented in NucleoOS (context switch, or scheduler, or heap allocator)
- Deliverable: public-facing artifacts demonstrating engineering depth

**Phase 5 done when:** A real project runs on NucleoOS, documentation is complete, and at least one public technical artifact exists.

---

## 9. Coding Philosophy

**No vibe coding.** Every line written must be understood. If a line cannot be explained out loud, it is not written yet.

**No HAL, no CubeMX, no generated code.** Registers are accessed directly. Peripheral addresses come from the Reference Manual.

**No AI code generation.** Claude was used to design the roadmap and draft this specification. It will not be used to generate implementation code. The point of this project is to learn by struggling.

**Slow is smooth, smooth is fast.** A week spent truly understanding the context switch is worth more than a week of copy-pasting code that happens to work.

**Comments explain why, not what.** Any comment that restates the code is noise. Comments exist to explain intent, tradeoffs, and Reference Manual page references.

**Every magic number has a name.** Register addresses, bit masks, and peripheral base addresses are defined as named constants in headers. No unexplained hex literals in `.c` files.

---

## 10. Resources

### Primary references

| Resource | Usage |
|----------|-------|
| STM32 Reference Manual (RM) | Primary hardware reference — register addresses, peripheral configuration |
| STM32 Datasheet | Pin mapping, electrical characteristics |
| ARM Cortex-M Definitive Guide — Joseph Yiu | Architecture: registers, exception model, instruction set |
| ARM Architecture Reference Manual (ARMv7E-M) | Authoritative but dense — use for specific questions |

### Books & online

| Resource | Phase relevance |
|----------|----------------|
| *Operating Systems: Three Easy Pieces* (OSTEP) — free online | Phases 3–4: scheduler, memory, synchronisation |
| *Modern C* — Jens Gustedt — free PDF | Phase 1: C language reference |
| OSDev Wiki (wiki.osdev.org) | General bare-metal reference, forum archive |
| Embedded Artistry blog | Architecture patterns, driver abstraction |

### Tooling

| Tool | Purpose |
|------|---------|
| `arm-none-eabi-gcc` | Cross-compiler |
| `arm-none-eabi-gdb` | Debugger |
| `openocd` | Flash + debug server over ST-Link |
| `arm-none-eabi-objdump` | Binary inspection |
| `arm-none-eabi-readelf` | ELF section inspection |
| `arm-none-eabi-nm` | Symbol table inspection |
| `arm-none-eabi-size` | Section size report |
| minicom / screen / PuTTY | UART serial terminal |

---

*Last updated: 31st May 2026. Generated with AI assistance (Claude, Anthropic) — see transparency notice at top.*