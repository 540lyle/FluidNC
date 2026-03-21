[Verified] Purpose
- Orchestrate startup, channel polling, protocol command handling, and queued output delivery.

[Verified] Key Files
- `C:\src\FluidNC\FluidNC\src\Main.cpp`
- `C:\src\FluidNC\FluidNC\src\Protocol.cpp`
- `C:\src\FluidNC\FluidNC\src\Protocol.h`
- `C:\src\FluidNC\FluidNC\src\System.h`
- `C:\src\FluidNC\FluidNC\src\State.h`
- `C:\src\FluidNC\FluidNC\src\Job.cpp`

[Verified] Inputs
- Realtime chars and line-oriented commands from channels.
- Job-file lines from the active job channel.
- Module polling hooks from `Modules()`.
- Runtime state changes via `sys`, alarms, and unwind flags.

[Verified] Outputs
- Protocol actions and state transitions.
- Messages printed through channel output queue.
- Job completion/abort handling.
- Startup event broadcast after setup completes.

[Verified] Invariants
- `Main.cpp` encodes the boot init order explicitly; later subsystems assume earlier ones already exist.
- `protocol_main_loop()` starts polling before entering its main idle/dispatch loop.
- If a job is active, normal line input is taken only from the job channel until the job unwinds or completes.
- Polling pauses during binary upload (`pollingPaused` path).
- More than one non-user-abort failure in `loop()` causes a permanent stall instead of infinite restart attempts.

[Verified] Failure / Risk Areas
- Shared globals across tasks (`activeChannel`, `activeLine`, message queue, task handles) make concurrency-sensitive edits risky.
- Alarm/config-alarm/critical states can asynchronously unwind jobs.
- Output is decoupled through a queue and task; direct assumptions about synchronous logging are unsafe.

[Verified] What To Read First Before Editing
1. `C:\src\FluidNC\FluidNC\src\Main.cpp`
2. `C:\src\FluidNC\FluidNC\src\Protocol.cpp`
3. `C:\src\FluidNC\FluidNC\src\System.h`
4. The owning subsystem for the command/event being changed

[Unknown] Not Yet Traced
- Detailed command dispatch from input line to planner/stepper execution for all gcode/realtime paths.
