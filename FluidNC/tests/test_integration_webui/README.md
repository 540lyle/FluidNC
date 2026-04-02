[Verified] Purpose
- This suite exercises the real WebUI product modules on the native host harness.

[Verified] Build Pattern
- This suite relies on the shared `integration_common.build_src_filter` host surface in `C:\src\FluidNC\platformio.ini`.
- Product code is compiled directly from `FluidNC/src`.
- Host shims are compiled directly from `FluidNC/capture`, including `C:\src\FluidNC\FluidNC\capture\Stage1HostSupport.cpp`.

[Verified] Rules
- Prefer moving ESP32-only registration/bootstrap into dedicated product registration files rather than adding test-mode branches.
- If a test needs additional host shims, add them in `FluidNC/capture` or in the test file itself when the seam is suite-local and behaviorless.
- If the suite needs additional shared product sources, add them to `integration_common.build_src_filter`.
