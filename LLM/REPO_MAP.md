[Verified] Scope
- `C:\src\FluidNC\FluidNC\` is the firmware, host shim, and host test tree.
- `C:\src\FluidNC\platformio.ini` is the primary build/test matrix.

[Verified] Top-Level Directory Map
- `C:\src\FluidNC\FluidNC\`
  - Core firmware sources, host shims, and tests.
- `C:\src\FluidNC\fixture_tests\`
  - Hardware-backed fixture runner plus `.nc` scenarios and Python tests.
- `C:\src\FluidNC\embedded\`
  - Embedded WebUI asset packaging/build inputs.
- `C:\src\FluidNC\fluidterm\`
  - Host-side terminal utility.
- `C:\src\FluidNC\tools\`
  - Repo tooling, including coverage support.

[Verified] Key Subtrees Under `FluidNC/`
- `C:\src\FluidNC\FluidNC\src`
  - Main implementation root.
- `C:\src\FluidNC\FluidNC\include`
  - Shared headers and Arduino/ESP32 overlays.
- `C:\src\FluidNC\FluidNC\capture`
  - Host-test Arduino/ESP32 shims and simulators.
- `C:\src\FluidNC\FluidNC\posix`
  - POSIX host console support.
- `C:\src\FluidNC\FluidNC\win32`
  - Windows host console support.
- `C:\src\FluidNC\FluidNC\tests`
  - Unit and host integration suites.

[Verified] Main Build/Test Entry Points
- Firmware build matrix: `C:\src\FluidNC\platformio.ini`
- ESP32 runtime entrypoint: `C:\src\FluidNC\FluidNC\src\Main.cpp`
- Host unit test entrypoint: `C:\src\FluidNC\FluidNC\tests\test_unit\test_main.cpp`
- Host integration shared entrypoint shim: `C:\src\FluidNC\FluidNC\tests\support\integration_gtest_main.cpp`
- Coverage driver: `C:\src\FluidNC\coverage.py`
- WebUI composition seam examples:
  - `C:\src\FluidNC\FluidNC\src\WebUI\MdnsRegistration.cpp`
  - `C:\src\FluidNC\FluidNC\src\WebUI\NotificationsServiceRegistration.cpp`
  - `C:\src\FluidNC\FluidNC\src\WebUI\OTARegistration.cpp`

[Verified] Main Commands
- ESP32 default firmware: `pio run -e wifi`
- ESP32 S3 firmware: `pio run -e wifi_s3`
- Host compile-only environments: `pio run -e windows_x86`, `pio run -e posix`
- Unit tests: `pio test -e tests`
- Unit coverage: `pio test -e tests_coverage`
- Host integration all suites: `pio test -e integration -vv`
- Host integration single suite: `pio test -e integration -f test_integration_machine_axes -vv`
- Host integration coverage: `pio test -e integration_coverage -vv`
- Coverage sweep: `python coverage.py`
- Fixture Python tests: `python -m unittest discover -s fixture_tests/tests -v`

[Verified] Important Generated / Usually Ignorable Areas
- `C:\src\FluidNC\.pio\`
  - PlatformIO build output.
- Root `coverage*.html`, `coverage*.json`, `coverage*.txt`
  - Generated coverage artifacts.
- `C:\src\FluidNC\fluidterm\fluidterm.exe`
  - Built binary artifact.

[Verified] How To Navigate This Repo
1. Start with `C:\src\FluidNC\platformio.ini` to identify the exact env and `build_src_filter`.
2. For runtime behavior, trace `C:\src\FluidNC\FluidNC\src\Main.cpp` -> `C:\src\FluidNC\FluidNC\src\Machine\MachineConfig.cpp` -> `C:\src\FluidNC\FluidNC\src\Protocol.cpp`.
3. For config-driven features, read the target class's `group()`, `afterParse()`, and `validate()` methods.
4. For host-safe validation, prefer `pio test -e integration -f test_integration_<suite>` before the full consolidated integration env.
5. Treat `fixture_tests` as hardware-only unless the task explicitly includes fixture execution.
6. For host testability seams, prefer dedicated registration/bootstrap translation units and `capture/` shims over product-file `PIO_UNIT_TESTING` branches.

[Inference] Navigation Heuristic
- Most feature work crosses one structural folder plus shared runtime files like `Settings.*`, `Protocol.cpp`, or `System.*`.
