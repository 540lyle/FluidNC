[Verified] Purpose
- Map the host-side validation layers so future tasks can choose the smallest relevant path quickly.

[Verified] Key Files
- `C:\src\FluidNC\platformio.ini`
- `C:\src\FluidNC\FluidNC\tests\test_unit\test_main.cpp`
- `C:\src\FluidNC\FluidNC\tests\support\integration_gtest_main.cpp`
- `C:\src\FluidNC\FluidNC\tests\TEST_PLAN.md`
- `C:\src\FluidNC\coverage.py`
- `C:\src\FluidNC\FluidNC\capture\`
- `C:\src\FluidNC\fixture_tests\`

[Verified] Inputs
- PlatformIO env-specific `build_src_filter` lists.
- `capture/` shims and simulators for host builds.
- GTest entrypoints for unit and integration executables.
- Python fixture runner inputs from `fixture_tests/fixtures/*.nc`.

[Verified] Outputs
- Unit test executables via `pio test`.
- Directory-discovered integration executables via `pio test -e integration`.
- Coverage reports via `coverage.py`.
- Hardware-oriented serial scenario checks via `fixture_tests`.

[Verified] Invariants
- Host unit and integration envs use explicit source allowlists; adding coverage or new host tests often still requires `platformio.ini` edits.
- Integration suite selection is by top-level suite directory name (`test_integration_*`), not per-suite PlatformIO env.
- Discovered integration suites run on one shared host build surface from `integration_common.build_src_filter`.
- Shared host runtime shims for the current stage live in `C:\src\FluidNC\FluidNC\capture\Stage1HostSupport.cpp`.
- Windows integration builds also use `C:\src\FluidNC\tools\integration_path_aliases.py` and the compiler/archive wrappers in `C:\src\FluidNC\tools\` to absolutize PlatformIO-relative paths.
- `coverage.py` expects `tests_coverage` plus one shared `integration_coverage` env and maps build outputs back to source files.
- Fixture tests are a separate Python/hardware layer, not part of the PlatformIO host suites.
- Product behavior files should keep a single behavior path; host-specific registration/setup belongs in composition files or `capture/` shims.

[Verified] Failure / Risk Areas
- A feature can compile in firmware but still be absent from a host env because of `build_src_filter`.
- A suite can appear to cover a subsystem while actually exercising host-local shadow implementations unless the suite-local wrappers include the real `.cpp` files.
- Native host tests sometimes need suite-local source shims or stubs to avoid ESP32-specific static initialization.
- Native host builds can also need build-tool wrappers when PlatformIO emits relative source/object/archive paths that are not valid from the compiler cwd.
- Coverage artifacts in the repo root are generated output and can become stale.
- Registration side effects are a common source of host drift; prefer moving them into dedicated bootstrap TUs so tests can instantiate real modules directly.

[Verified] What To Read First Before Editing
1. `C:\src\FluidNC\platformio.ini`
2. `C:\src\FluidNC\FluidNC\tests\TEST_PLAN.md`
3. The subsystem's registration/bootstrap files, if they exist, before editing leaf modules
4. The nearest existing `test_integration_*` suite directory and the shared `integration_common.build_src_filter`
5. `C:\src\FluidNC\coverage.py` if the task affects coverage or suite membership

[Inference] Validation Heuristic
- Prefer `tests` for pure helpers/parsers, `pio test -e integration -f test_integration_<suite>` for subsystem interaction, and `fixture_tests` only for behavior that depends on real controller/runtime timing.
- If a test requires changing product code solely for host execution, first look for a composition-root or shim-based alternative.
