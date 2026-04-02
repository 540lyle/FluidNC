[Verified] Purpose
- This suite covers host-safe machine bus behavior without pulling in the full machine runtime surface.

[Verified] Build Pattern
- This suite relies on the shared `integration_common.build_src_filter` host surface in `C:\src\FluidNC\platformio.ini`.
- Product bus and pin code is compiled directly from `FluidNC/src`.
- Shared host shims are compiled directly from `FluidNC/capture`, including `C:\src\FluidNC\FluidNC\capture\Stage1HostSupport.cpp`.

[Verified] Rules
- Keep this suite focused on bus and pin behavior; broader machine config behavior belongs in dedicated machine/config suites.
- Avoid suite-local product reimplementations.
- If host execution needs a new seam, prefer `capture/` or a product registration/bootstrap split over `PIO_UNIT_TESTING`.
- If the suite needs additional shared product sources, add them to `integration_common.build_src_filter`.
