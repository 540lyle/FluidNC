[Verified] Purpose
- Load machine YAML into a runtime object graph and validate/default missing sections before the rest of firmware initialization uses it.

[Verified] Key Files
- `C:\src\FluidNC\FluidNC\src\Machine\MachineConfig.cpp`
- `C:\src\FluidNC\FluidNC\src\Machine\MachineConfig.h`
- `C:\src\FluidNC\FluidNC\src\Configuration\Parser.cpp`
- `C:\src\FluidNC\FluidNC\src\Configuration\AfterParse.cpp`
- `C:\src\FluidNC\FluidNC\src\Configuration\Validator.cpp`
- `C:\src\FluidNC\FluidNC\src\Configuration\_Overview.md`
- `C:\src\FluidNC\FluidNC\src\SettingsDefinitions.h`

[Verified] Inputs
- Config filename from settings (`config_filename` usage in `MachineConfig.cpp`).
- YAML text from local FS or builtin default config text.
- Factory-registered subsystem types exposed through `ConfigurableModuleFactory`, `SpindleFactory`, `ATCFactory`, `SysListenerFactory`.

[Verified] Outputs
- Global `config` pointer updated to the current `MachineConfig` instance.
- Default objects created for omitted structural sections.
- Config/log errors and possible `State::ConfigAlarm`.

[Verified] Invariants
- `MachineConfig::group()` defines the parse tree and config field ownership boundaries.
- `load_yaml()` replaces the singleton instance before parsing into it.
- `afterParse()` runs before validation.
- At least one spindle object exists after `afterParse()`.
- Spindle tool numbers are sorted and normalized so the first spindle tool is `0`.

[Verified] Failure / Risk Areas
- Panic recovery skips the file and forces builtin default config.
- File-open/read failure logs config errors and falls back to builtin default config.
- Global factories and singleton replacement make partial initialization bugs easy to introduce.
- `MachineConfig` destructor only visibly deletes a subset of owned pointers in the inspected code; do not assume lifetime rules without re-reading surrounding code.

[Verified] What To Read First Before Editing
1. `C:\src\FluidNC\FluidNC\src\Machine\MachineConfig.cpp`
2. `C:\src\FluidNC\FluidNC\src\Machine\MachineConfig.h`
3. The target component's `group()`, `afterParse()`, and `validate()` methods
4. `C:\src\FluidNC\FluidNC\src\Configuration\Parser.cpp` and `C:\src\FluidNC\FluidNC\src\Configuration\Validator.cpp` if the change affects parse semantics

[Inference] Edit Strategy
- For most config feature work, start at the owning `Machine` or feature class and only then widen into `Configuration/` internals if the problem is parser/factory behavior rather than field wiring.
