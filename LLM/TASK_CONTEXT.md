[Verified] Task Goal
- Refactor host integration testing to use one PlatformIO `integration` env and one `integration_coverage` env.
- Keep suites independently runnable through PlatformIO discovery.
- Preserve per-suite and per-test-case reporting for local runs and CI/JUnit output.
- Keep stage-1 infrastructure aligned with enterprise-style production code:
  - platform shims in `capture/`
  - composition/bootstrap in dedicated registration files
  - no product-code behavior forks for host tests

[Verified] Files Inspected For This Pass
- `C:\src\FluidNC\platformio.ini`
- `C:\src\FluidNC\coverage.py`
- `C:\src\FluidNC\.github\workflows\ci.yml`
- `C:\src\FluidNC\FluidNC\tests\TEST_PLAN.md`
- `C:\src\FluidNC\FluidNC\capture\TestStubs.cpp`
- `C:\src\FluidNC\FluidNC\tests\test_integration_machine_buses\test_MachineBusIntegrationTest.cpp`
- `C:\src\FluidNC\FluidNC\tests\test_integration_webui\test_WebUiNativeIntegrationTest.cpp`

[Verified] Current Task Files
- `C:\src\FluidNC\platformio.ini`
- `C:\src\FluidNC\.github\workflows\ci.yml`
- `C:\src\FluidNC\coverage.py`
- `C:\src\FluidNC\FluidNC\tests\support\integration_gtest_main.cpp`
- `C:\src\FluidNC\FluidNC\tests\test_unit\`
- `C:\src\FluidNC\FluidNC\tests\test_integration_*`
- `C:\src\FluidNC\FluidNC\tests\support\`
- `C:\src\FluidNC\FluidNC\capture\TestStubs.cpp`
- `C:\src\FluidNC\FluidNC\tests\TEST_PLAN.md`
- `C:\src\FluidNC\FluidNC\src\WebUI\Mdns.cpp`
- `C:\src\FluidNC\FluidNC\src\WebUI\MdnsRegistration.cpp`
- `C:\src\FluidNC\FluidNC\src\WebUI\NotificationsService.cpp`
- `C:\src\FluidNC\FluidNC\src\WebUI\NotificationsServiceRegistration.cpp`
- `C:\src\FluidNC\FluidNC\src\WebUI\OTA.cpp`
- `C:\src\FluidNC\FluidNC\src\WebUI\OTARegistration.cpp`

[Verified] Findings Confirmed So Far
- The old per-suite `integration_*` and `integration_*_coverage` env blocks were replaced with shared `integration_common`, `integration`, `integration_coverage`, and `integration_asan`.
- PlatformIO suite discovery now runs against top-level `FluidNC/tests/test_integration_*` directories.
- Shared integration startup now comes from `C:\src\FluidNC\FluidNC\tests\support\integration_gtest_main.cpp`.
- `coverage.py` now maps skip flags to suite directories and drives `pio test -e integration_coverage -f test_integration_*`.
- CI now runs one `pio test -e integration -vv --junit-output-path ...` job for host integration reporting.
- `test_integration_machine_buses` keeps two `MachineConfig::afterParse()` cases skipped on the native host because equivalent coverage exists in dedicated machine/config suites and the native harness path remained unstable.
- `test_integration_webui` now exercises the real `WebUI::Mdns`, `WebUI::NotificationsService`, and `WebUI::OTA` classes; the prior OTA test-local product copy was removed.
- `tools/coverage_guard.py` now evaluates active-host called coverage against `called_percent_of_total` when available so missing active-host files cannot be hidden by a reduced denominator.
- WebUI registration/bootstrap was moved out of leaf product files into dedicated `*Registration.cpp` translation units, removing `PIO_UNIT_TESTING` branches from `FluidNC/src`.
- Stage 1 now uses one shared host integration surface from `integration_common.build_src_filter`.
- Suite-local runtime shims were extracted into `C:\src\FluidNC\FluidNC\capture\Stage1HostSupport.cpp`, leaving the test files focused on fixtures, fake objects, and assertions.

[Verified] Assumptions
- Host integration is intended to run through PlatformIO `pio test`, not hand-run `pio run` binaries.
- A single shared integration build configuration is acceptable if suite-local source shims/stubs absorb host-only differences.
- The local worktree is dirty; unrelated changes must be left intact.

[Verified] Risks
- Native-host behavior for some ESP32-oriented codepaths still depends on suite-local stubs and selective skips.
- `integration_asan` was not runnable in this Windows environment because the local MinGW toolchain could not link `-lasan`.
- Stage 1 intentionally covers only machine-bus and WebUI host slices; coverage thresholds must remain aligned with the selected suite set rather than the historic full-integration surface.

[Inference] Sharp Edges To Re-check Before Future Code Changes
- New host integration suites should default to `FluidNC/tests/test_integration_<name>/` on top of the shared `integration_common` host surface rather than adding another PlatformIO env.
- New host seams should default to composition-root files and `capture/` shims, not `PIO_UNIT_TESTING` branches in product `.cpp` files.
- Any source added to `integration_common.build_src_filter` affects coverage inventory and all integration suites.
- For discovered suites, verify the shared host surface includes the real product `.cpp` files under test; copied implementations or placeholder wrappers silently invalidate coverage claims.
- Shared helpers belong in `FluidNC/tests/support/` or `FluidNC/capture/`; discoverable suite directories should only contain suite-local sources.

[Unknown] Open Questions
- Whether the skipped `MachineConfig::afterParse()` bus tests should be rehomed into a dedicated machine/config suite or re-enabled with a more complete native harness later.
- Whether more WebUI coverage should move shared host shims out of `test_integration_webui` into `capture/` if additional suites begin to exercise the same codepaths.
- Whether stage 2 should introduce additional shared host support modules beyond `Stage1HostSupport.cpp` as more subsystems join the integration surface.

[Verified] Validation Run For This Task
- `python -m unittest discover -s tools -p "test_coverage_guard.py" -v`
- `python -m unittest tools.test_build_manifests -v`
- `pio test -e integration -f test_integration_machine_buses -vv`
- `pio test -e integration -f test_integration_webui -vv`
- `pio test -e tests -f test_unit -vv`
- `pio test -e integration -f test_integration_config -vv`
- `pio test -e integration -f test_integration_suite -vv`
- `pio test -e integration -f test_integration_machine_buses -vv`
- `pio test -e integration -f test_integration_webui -vv`
- `pio test -e integration -vv`
- `pio test -e integration_coverage -f test_integration_config -vv`
- `pio test -e integration -f test_integration_config -vv --junit-output-path C:\src\FluidNC\integration-junit.xml`
