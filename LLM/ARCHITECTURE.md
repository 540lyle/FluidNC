[Verified] Runtime Boot Sequence
- `C:\src\FluidNC\FluidNC\src\Main.cpp`
  - `setup()` performs platform preinit, sets `State::Starting`, initializes timing/settings/console/channels/protocol, mounts local FS, loads machine config, then initializes buses, extenders, listeners, stepping, planner, user IO, axes, control, kinematics, limits, modules, ATCs, spindles, coolant, probe, and proxy objects.
  - After setup it polls GPIOs once, marks channels ready, removes `startupLog` from channel registration, and sends a startup event.
- `loop()` calls `protocol_main_loop()` and stalls permanently after more than one non-user-abort failure.

[Verified] Major Runtime Components
- Boot/orchestration
  - `C:\src\FluidNC\FluidNC\src\Main.cpp`
- Protocol and IO polling
  - `C:\src\FluidNC\FluidNC\src\Protocol.cpp`
  - `C:\src\FluidNC\FluidNC\src\Job.cpp`
- Machine configuration and structure
  - `C:\src\FluidNC\FluidNC\src\Machine\MachineConfig.cpp`
  - `C:\src\FluidNC\FluidNC\src\Configuration\*.cpp`
- Persistent settings/commands
  - `C:\src\FluidNC\FluidNC\src\Settings.cpp`
  - `C:\src\FluidNC\FluidNC\src\Settings.h`
- Global runtime state
  - `C:\src\FluidNC\FluidNC\src\System.h`
  - `C:\src\FluidNC\FluidNC\src\State.h`
- Pluggable machine subsystems
  - `C:\src\FluidNC\FluidNC\src\Machine\`
  - `C:\src\FluidNC\FluidNC\src\Motors\`
  - `C:\src\FluidNC\FluidNC\src\Spindles\`
  - `C:\src\FluidNC\FluidNC\src\Kinematics\`
  - `C:\src\FluidNC\FluidNC\src\Listeners\`
  - `C:\src\FluidNC\FluidNC\src\ToolChangers\`

[Verified] Control Flow / Data Flow
- Boot path
  - NVS-backed settings initialize first via `settings_init()`.
  - Local FS config filename comes from settings; `MachineConfig::load()` then parses YAML into the global `config`.
  - Hardware/logic subsystems are initialized only after config load.
- Input path
  - `Protocol.cpp` starts a polling task.
  - Polling reads realtime chars and line-oriented input from channels.
  - If a job is active, the active job channel has priority over other line-oriented input.
- Output path
  - `Protocol.cpp` uses a message queue and a dedicated output task to print through channels.
- Host-test path
  - PlatformIO host envs build selected firmware sources plus `capture/` shims and a gtest `main()`.

[Verified] Configuration Pipeline
- `MachineConfig::load()`:
  - Falls back to a builtin default config after a panic or file-open failure.
  - Uses `State::ConfigAlarm` on panic/missing-config fallback paths.
- `MachineConfig::load_yaml()`:
  - Replaces the singleton config instance.
  - Parses with `Configuration::Parser` and `ParserHandler`.
  - Runs `afterParse()` callbacks before `validate()`.
- `MachineConfig::afterParse()`:
  - Auto-creates defaults for missing structural sections like axes, control, stepping, coolant, probe, user IO, parking.
  - Ensures at least one spindle exists by adding `NullSpindle` if needed.
  - Sorts spindles by tool and forces the first spindle tool number to `0`.

[Verified] State / Invariants
- `C:\src\FluidNC\FluidNC\src\State.h`
  - `State::Idle` must remain zero.
- `C:\src\FluidNC\FluidNC\src\Main.cpp`
  - Initialization order is dependency-driven and encoded explicitly in `setup()`.
- `C:\src\FluidNC\FluidNC\src\Protocol.cpp`
  - Polling is paused during xmodem-style binary upload.
  - Alarm/config-alarm/critical state while a job is active triggers job unwind/abort logic.
- `C:\src\FluidNC\FluidNC\src\Settings.cpp`
  - Setting names longer than 15 chars are hashed for NVS storage keys.
  - `Setting::check_state()` currently allows writes in `Idle`, `Alarm`, `ConfigAlarm`, `SafetyDoor`, and `Critical`; it rejects other states.
- `C:\src\FluidNC\FluidNC\src\System.h`
  - `system_t` tracks dirty flags and calls registered change handlers when state/override fields mutate.

[Verified] Subsystem Boundaries
- `Configuration/`
  - Generic parser, validator, factories, and config traversal mechanics.
- `Machine/`
  - Structural machine definition and shared machine IO pieces.
- `Motors/`, `Spindles/`, `Kinematics/`, `Listeners/`, `ToolChangers/`
  - Feature families registered through factory-style patterns.
- `capture/`, `posix/`, `win32/`
  - Host-only support layers; not MCU runtime code.
- `fixture_tests/`
  - External hardware validation layer, separate from PlatformIO host suites.

[Verified] Host-Test Boundaries
- `C:\src\FluidNC\FluidNC\capture\`
  - Preferred location for Arduino/ESP32/FreeRTOS host shims.
- `C:\src\FluidNC\FluidNC\src\WebUI\*Registration.cpp`
  - Composition/bootstrap boundary for module registration and setting/command wiring.
- `C:\src\FluidNC\FluidNC\src\WebUI\*.cpp`
  - Leaf product modules now keep one runtime behavior path and should not carry host-test-only branches.
- `C:\src\FluidNC\FluidNC\tests\test_integration_*`
  - Suite-local wrappers still exist for translation-unit selection, but tests should instantiate real product modules where possible instead of reimplementing them.

[Verified] Composition Invariant
- WebUI stage-1 host coverage now follows this split:
  - behavior in `Mdns.cpp`, `NotificationsService.cpp`, `OTA.cpp`
  - registration/bootstrap in `MdnsRegistration.cpp`, `NotificationsServiceRegistration.cpp`, `OTARegistration.cpp`
- This keeps module factory side effects and setting registration out of the behavior files, which reduces ESP32-vs-host drift.

[Inference] Architectural Pattern
- The repo is organized around config-built object graphs plus factory-registered feature families, with global runtime singletons used to connect protocol, machine state, and IO.

[Unknown] Not Yet Verified In This Pass
- Detailed planner/stepper execution flow after protocol command dispatch.
- Exact ownership/lifetime rules for every config-created object beyond the visible `MachineConfig` destructor.
- Whether the ESP-IDF CMake source list is currently fully aligned with the PlatformIO source filters on this branch.
