---
name: esp32-init
description: Use when starting a new ESP32 project from scratch before writing any application code. Do NOT use for adding features to an existing project.
---

# ESP32 Project Initialization

Use this skill when the user wants to start a new ESP-IDF project. Follow the ordered workflow below. Do not skip pre-work.

## Workflow

### Phase 1: System Pre-work (one-time per IDF environment)

1. Install `esp-bmgr-assist` in the IDF Python environment so `idf.py bmgr` is available as a CLI action:
    ```bash
    pip install esp-bmgr-assist
    ```
2. Ask the user for:
    - Chip variant (e.g. esp32, esp32s3, esp32c6)
    - Module type (WROOM / WROVER)
    - Board model name (e.g. CYD / ESP32-2432S028R, ESP32-DevKitC, M5Stack Core2)

### Phase 2: Hardware Verification

1. Ask the user to connect the board via USB and identify the serial port (`/dev/ttyUSB0`, `/dev/ttyACM0`, COM3, etc.)
2. Run `idf.py -p <port> monitor` briefly to confirm the device responds (boot log visible, no error)
3. Exit monitor (`Ctrl+]`)
4. Only proceed if hardware connection is confirmed. If no device found, stop.

### Phase 3: Project Creation

1. `idf.py create-project --cpp <project_name>` — creates C++ project with `.cpp` entry point
2. `idf.py set-target <chip>` — e.g. esp32, esp32s3, esp32c6
3. Add BMGR dependency:
    ```bash
    idf.py add-dependency "espressif/esp_board_manager"
    ```
    This auto-pulls `espressif/esp_boards` (official Espressif board definitions).
4. Set up Python virtual environment for test tooling (runs on host, not on device):
    - `uv init --no-project` — creates `pyproject.toml` without a project type
    - `uv add --dev pytest-embedded[serial]` — adds test framework as dev dependency

### Phase 4: Board Discovery

1. Scan available boards:
    ```bash
    idf.py bmgr -l
    ```
2. **If the user's board is listed:** Select it:
    ```bash
    idf.py bmgr -b <board_name_or_index>
    ```
    BMGR generates init code under `components/gen_bmgr_codes/`. Proceed to Phase 5.
3. **If the board is NOT listed:** Try adding community board definitions:
    ```yaml
    # main/idf_component.yml
    dependencies:
      espressif/esp_friends_boards: "*"
    ```
    Then re-scan (`idf.py bmgr -l`). If the board now appears, select it.
4. **If still not found (custom/unknown board):**
    - Fetch canonical pinout from a known source (vendor GitHub repo, Espressif docs, manufacturer schematic PDF)
    - Manually populate the three BMGR YAML files under `components/<board_name>/`:
        - `board_info.yaml` — metadata (board name, chip, version, manufacturer)
        - `board_peripherals.yaml` — low-level bus/gpio/peripheral pin assignments
        - `board_devices.yaml` — functional devices with I2C addresses, SPI modes, init params
    - Board names must use only lowercase letters, digits, and underscores (hyphens are rejected by BMGR)
    - Run BMGR to validate and generate:
        ```bash
        idf.py bmgr -b <board_name>
        ```
        BMGR automatically validates the YAMLs: SoC capabilities, hardware limits, I/O validity, I/O conflicts, name matching, and peripheral dependency resolution.
    - Create `BOARD_NOTES.md` in the board directory documenting any peripherals or devices that could not be configured via BMGR (wrong chip, unsupported driver, known pin conflict, missing component, etc.). Use terse bullet points:
        ```markdown
        # Board: cyd_2432s028r — Known Issues

        - **Touch (XPT2046)**: Not in BMGR device matrix. Init manually in app.
        - **RGB LEDs (GPIO 4, 16, 17)**: Active-low, no BMGR device type. Init via `gpio_set_level` in app.
        - **Light sensor (GPIO 34)**: Input-only pin, no BMGR device. Read via ADC in app.
        ```
5. **STOP. Present the pinout/board selection and known issues to the user. Get explicit agreement before proceeding.**

### Phase 5: Configure and Build

1. `idf.py menuconfig` — configure:
    - Partition table (factory vs OTA)
    - Flash size / mode
    - WiFi / Bluetooth if needed
    - Component-specific Kconfig options
2. `idf.py save-defconfig` — creates `sdkconfig.defaults`
3. Write `project.env` at project root — agent manifest referencing canonical files:
    ```ini
    ESP_PROJECT_NAME=<name>
    ESP_TARGET=<target>                # also in sdkconfig.defaults: CONFIG_IDF_TARGET
    ESP_IDF_VERSION=v6.1              # from `eim list`
    ESP_BOARD=<board>
    ESP_PORT=<port>                    # only place this is persisted
    ESP_SDKCONFIG_DEFAULTS=sdkconfig.defaults
    ESP_MAIN_IDF_COMPONENT_YML=main/idf_component.yml
    ESP_BOARD_DIR=components/<board>/
    ESP_BOARD_NOTES=components/<board>/BOARD_NOTES.md
    ```
    Keep it small (pointers, not data) — avoid polluting agent context.
4. `idf.py build` — verify compilation succeeds

### Phase 6: Flash and Verify

1. `idf.py -p <port> flash`
2. `idf.py -p <port> monitor` — confirm app runs on device

## Generated Project Structure

```
<project_name>/
├── CMakeLists.txt              # project-level build config
├── main/
│   ├── CMakeLists.txt          # main component registration
│   └── <project_name>.cpp      # app_main() entry point
├── components/
│   ├── <board_name>/           # board definition (Phase 4 — only for custom boards)
│   │   ├── board_info.yaml
│   │   ├── board_peripherals.yaml
│   │   ├── board_devices.yaml
│   │   ├── BOARD_NOTES.md          # known issues, unresolved peripherals
│   │   └── sdkconfig.defaults.board  (optional)
│   └── gen_bmgr_codes/         # auto-generated by bmgr (do not edit)
├── sdkconfig                   # generated by menuconfig (gitignore)
├── sdkconfig.defaults          # project defaults (commit this)
├── build/                      # compiled output (gitignore)
├── pyproject.toml              # uv-managed Python tooling config
├── .venv/                      # virtual env (gitignore)
├── project.env                  # agent manifest (commit this)
└── main/idf_component.yml      # component registry dependencies
```

## Common Mistakes

- **Skipping `pip install esp-bmgr-assist`**: Without this, `idf.py bmgr` action may not be available. Do this once per IDF env in Phase 1.
- **Forgot `espressif/esp_board_manager` dependency**: `idf.py bmgr` silently fails. Add via `idf.py add-dependency "espressif/esp_board_manager"` before `idf.py bmgr -l`.
- **Skipped `set-target` before `bmgr -b`**: BMGR may select wrong chip, or `board_manager.defaults` chip won't match `sdkconfig`. Always `set-target` first.
- **`idf.py set-target` after `bmgr -b`**: This wipes `sdkconfig` and regenerates, requiring you to re-run `bmgr -b` afterward.
- **Board name with hyphens**: BMGR rejects them. Use underscores only.
- **Jumping straight to manual YAML**: Always run `idf.py bmgr -l` first — the board may already be defined in `esp_boards` or `esp_friends_boards`, saving you the work of writing YAMLs.
- **Not using BMGR validation**: After writing custom YAMLs, `idf.py bmgr -b <board_name>` validates SoC capabilities, I/O conflicts, pin ranges, and peripheral deps. Run it even if you think the YAMLs are correct.
- **Not documenting unresolved peripherals**: If a component can't be expressed in BMGR YAML (touch controller, ADC sensor, custom LED), note it in `BOARD_NOTES.md` so the next agent doesn't waste time trying to shove it into YAML.