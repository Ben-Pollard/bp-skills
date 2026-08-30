---
name: esp-idf-debugging
description: Use when an ESP-IDF project has crashes (Guru Meditation, panic, abort), wrong behaviour with no crash, watchdog timeouts, memory corruption, or peripheral non-response. Use when reading serial monitor output, core dumps, or configuring debug Kconfig options.
---

# ESP-IDF Debugging

## Overview

Systematic diagnostic ladder for ESP-IDF on embedded targets. Work from lowest-cost signal (log) to highest-fidelity insight (JTAG). At every rung, produce a hypothesis before escalating.

## Pre-flight — Context Gathering

Before the ladder, collect context. If `project.env` exists at project root (from `esp32-init`), read it — answers most questions below.

### 1. Confirm this is an ESP-IDF project

Check `project.env` exists (fastest signal). If absent, fall back to `CMakeLists.txt` containing `project()` and `sdkconfig` or `sdkconfig.defaults`.

| When you see | Conclusion |
|---|---|
| `project.env` `ESP_TARGET=esp32s3` | Chip is ESP32-S3, port is in `ESP_PORT`, board is in `ESP_BOARD` |
| No `project.env` but `CMakeLists.txt` + `sdkconfig.defaults` | Project exists, probe `CONFIG_IDF_TARGET` from sdkconfig |
| Neither | Stop — not an ESP-IDF project |

### 2. Activate the IDF environment

```bash
eim list                              # see installed versions
eim select <major-version>            # e.g. v6.1 or v5.3
source <path-to-esp-idf>/export.sh    # activates idf.py, gdb, openocd in PATH
```

If `eim` not found, try `source` directly on a known `export.sh` path or check `ESP_IDF_VERSION` from `project.env`.

### 3. Confirm device connection

Read `ESP_PORT` from `project.env`:

```bash
idf.py -p <ESP_PORT> monitor  # boot log appears → connected
```

If no boot log, try `ls /dev/tty*` (Linux) or `ls /dev/cu.*` (macOS) to find the port. If no port found at all, Rungs 3–4 are unreachable. Rungs 1–2 (log analysis, Kconfig recommendations) still work.

### 4. Confirm debug symbol availability

`build/<ESP_PROJECT_NAME>.elf` must exist and be newer than source. Without it, core dump decode and GDB fail.

```bash
ls -l build/*.elf              # exists?
stat -c %Y build/*.elf         # when was it built?
```

If stale, `idf.py build` first.

### 5. Determine toolchain prefix from target

| `ESP_TARGET` | GDB binary |
|---|---|
| esp32, esp32s2, esp32s3 | `xtensa-esp32-elf-gdb` |
| esp32c3, esp32c6, esp32h2, esp32p4 | `riscv32-esp-elf-gdb` |

Used in Rung 3 (GDB Stub) and Rung 4 (OpenOCD). If `project.env` is missing, read `CONFIG_IDF_TARGET` from `sdkconfig`. If neither exists, guess from `build/bootloader/` ELF names.

## When to Use

- Any ESP-IDF crash, hang, or misbehaviour on real hardware
- Reviewing serial monitor output or core dump data
- Any symptom in the table below — match your symptom to the first step

Do NOT use for: host-side Linux unit tests (no target involved), build errors (compiler/linker), debugging Python test infrastructure.

## Symptom-to-Technique Reference

| Symptom | First step | Escalate to |
|---|---|---|
| Boot loop / panic | `idf.py -p ${ESP_PORT} monitor` — read panic reason, EXCVADDR, backtrace | Core dump decode (`idf.py coredump-info` or `idf.py coredump-debug`) |
| Wrong behaviour, no crash | `ESP_LOGW/TAG` at key points; check Kconfig log level | GDB Stub runtime (`CONFIG_ESP_SYSTEM_GDBSTUB_RUNTIME`) via `${TOOLCHAIN_PREFIX}-gdb -batch` over UART, or OpenOCD script |
| Task starvation / watchdog | Check TWDT config (`CONFIG_ESP_TASK_WDT`); add `vTaskDelay()` or yield | `vTaskGetRunTimeStats()` or `xTaskGetTickCount()` deltas in log |
| Memory corruption | Enable `CONFIG_HEAP_CORRUPTION_DETECTION` (strong), stack watchpoints, heap poisoning | Core dump analysis or `heap_caps_get_info()` dumps |
| Peripheral not responding | `ioctl`-style register read via monitor; verify init ordering and GPIO mux | OpenOCD register inspection script |
| Intermittent / timing-sensitive | Reduce optimisation (`-Og`); narrow window with conditional `ESP_LOGD` | OpenOCD hardware breakpoint via script |

## Core Pattern: Diagnostic Ladder

```
Log analysis  →  Core dump  →  GDB Stub runtime  →  OpenOCD scripted JTAG
(lowest cost)                                     (highest fidelity)
```

**At each rung, STOP and form a hypothesis.** Do not escalate without a theory that the next rung could disprove.

No hardware attached? You still have the monitor log. The log IS the data. Log analysis (Rung 1) and Kconfig recommendations require no hardware connection.

### Rung 1 — Log Analysis (always start here)

Every ESP-IDF panic log has four parts. Parse all four:

1. **Error cause** — e.g. `StoreProhibited`, `IllegalInstruction`, `LoadProhibited`, `InstrFetchProhibited`
2. **Register dump** — EXCVADDR tells you the fault address. If `0x00000000`, it is a NULL dereference. Compare A2/A3/A4 register values against expected pointers.
3. **Backtrace (raw hex)** — function return addresses
4. **Decoded backtrace** — `idf.py monitor` auto-decodes to file:line. Trace the call chain.

Other log signals: `assert failed: ...`, `abort() was called`, `TWDT timeout`, interrupt watchdog, cache error messages.

### Rung 2 — Core Dump

Enable `CONFIG_ESP_COREDUMP_FLASH` or `CONFIG_ESP_COREDUMP_UART` under `Component config → Core dump`.

After a crash, retrieve and decode:

```bash
idf.py coredump-info    # print task states, backtraces, registers
idf.py coredump-debug   # launch batch GDB session on the dump
```

Core dump captures ALL tasks' backtraces, not just the crashing one — useful for finding secondaries that caused the crash.

### Rung 3 — GDB Stub Runtime

When logs and core dump are insufficient, enable GDB Stub for interactive-ish debugging over UART (no JTAG needed):

```
CONFIG_ESP_SYSTEM_GDBSTUB_RUNTIME=y
```

Run monitor, send `Ctrl+C`, then launch batch GDB:

```bash
${TOOLCHAIN_PREFIX}-gdb -batch \
  -ex "target remote ${ESP_PORT}" \
  -ex "thread apply all bt" \
  -ex "p temp_sensor" \
  build/${ESP_PROJECT_NAME}.elf
```

Batch commands: `bt`, `info locals`, `p <var>`, `thread <id>`, `list`, `x/<fmt> <addr>`.

### Rung 4 — OpenOCD Scripted JTAG

For timing-sensitive bugs or hardware-level inspection. Board config depends on target — find it under `openocd-esp32/share/openocd/scripts/board/`:

```bash
openocd -f board/${ESP_TARGET}-builtin.cfg & \
  sleep 2 && \
  echo "halt; reg; resume; exit" | telnet localhost 4444
```

Scripted commands: `halt`, `resume`, `reg` (register dump), `mdw <addr>` (memory read word), `wp <addr> <len> <type>` (watchpoint), `bp <addr> hw` (hardware breakpoint).

## Kconfig Debug Knobs (turn before reproducing)

| Kconfig | Effect |
|---|---|
| `CONFIG_ESP_SYSTEM_PANIC_PRINT_REBOOT` | Default — print and reboot (fastest cycle) |
| `CONFIG_ESP_SYSTEM_PANIC_GDBSTUB` | Enter GDB stub on panic (postmortem) |
| `CONFIG_ESP_SYSTEM_GDBSTUB_RUNTIME` | Ctrl+C breaks into GDB stub at any time |
| `CONFIG_HEAP_CORRUPTION_DETECTION` | `weak` (free check only) / `strong` (poison every alloc) |
| `CONFIG_ESP_DEBUG_OCDAWARE` | Auto-detect JTAG debugger and halt on panic |
| `CONFIG_ESP_TASK_WDT` | Enable task watchdog with configurable timeout |
| `CONFIG_ESP_TASK_WDT_CHECK_IDLE_TASK` | Check idle task — catches total lockups |

## Common Mistakes

- **Skipping the log.** The panic log contains EXCVADDR and the decoded backtrace. Read it fully before doing anything else.
- **Escalating without a hypothesis.** Do not enable GDB Stub or OpenOCD until you know what question you are trying to answer.
- **Reading code instead of reading the log.** The source tells you what you INTENDED. The log tells you what ACTUALLY happened. Start with the log, not the code.
- **Missing the race.** `xTaskCreate` makes the task schedulable immediately. If the task dereferences a pointer that `app_main` assigns after creation, it races. Check init ordering.
- **Forgetting to re-enable watchdogs.** JTAG breakpoints disable hardware watchdogs. When debugging with OpenOCD, watchdog behaviour changes — do not rely on watchdog-triggered panics.
- **Postmortem vs runtime confusion.** GDB Stub postmortem (`CONFIG_ESP_SYSTEM_PANIC_GDBSTUB`) only lets you examine state after a crash. Runtime (`CONFIG_ESP_SYSTEM_GDBSTUB_RUNTIME`) lets you break in at will. Use the right mode.

## Red Flags — STOP and Re-apply the Ladder

| Rationalization | Reality |
|---|---|
| "I know what the bug is from reading the code" | You have a HYPOTHESIS. Test it against the log. Add a confirming log line and re-flash. |
| "There's no crash log so I need JTAG first" | Start with `ESP_LOGx` at key points. You cannot skip log analysis. |
| "I have no hardware right now, I can't debug" | You can analyse the log, form a hypothesis, and recommend Kconfig changes for the next build. Do that. |
| "The crash is in BT so I'll focus on BT code" | The crash SITE is BT. The ROOT CAUSE may be elsewhere (e.g. heap corrupt in MQTT that BT later trips over). Trace the call chain. |
| "This is intermittent so I need OpenOCD" | Start by reducing optimisation (`-Og`) and adding conditional `ESP_LOGD` inside the intermittent window. OpenOCD is the last resort, not the first. |