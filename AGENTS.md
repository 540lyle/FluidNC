# AGENTS.md

## Purpose
- Use this file to bootstrap work in an unfamiliar part of the repository.
- Build a lightweight `LLM/` context layer before making non-trivial changes so future tasks do not need to reload large parts of the codebase.
- Treat `LLM/` files as concise, code-derived working notes for LLMs and humans. They should be useful, but safe to ignore.

## Repository Layout
- `FluidNC/src`: main firmware source.
- `FluidNC/include`: shared headers and Arduino/ESP32 include overlays.
- `FluidNC/tests`: host-side unit and integration-style tests.
- `FluidNC/capture`: host-test shims and captures used by the non-ESP32 test environments.
- `FluidNC/win32`, `FluidNC/posix`: platform support for host-side builds/tests.
- `fixture_tests`: hardware-backed GCode fixture tests and runner.
- `tools`: repository tooling such as coverage guards.
- `platformio.ini`: primary build and test environment definitions.
- `coverage.py`: host-side coverage entrypoint.

## Default Workflow
1. Inspect the relevant repo area before editing code.
2. Create or refresh the minimal `LLM/` documents needed for the task.
3. Verify any summary you plan to rely on against the actual source files you will edit.
4. Make the change.
5. Update the affected `LLM/` notes if you learned something materially new.
6. Run the smallest relevant validation for the touched area.

## LLM Context Layer
Create a root `LLM/` folder when starting substantial work in an unfamiliar area. Use this default structure:
- `LLM/REPO_MAP.md`
- `LLM/ARCHITECTURE.md`
- `LLM/TASK_CONTEXT.md`
- `LLM/COMPONENTS/`

Do not try to summarize the whole repository up front. Build only enough context to safely execute the current task.

### `LLM/REPO_MAP.md`
Keep this short and practical:
- top-level directory map
- primary build/test commands
- major entry points and important subsystem roots
- areas that are generated, vendored, hardware-specific, or usually ignorable

### `LLM/ARCHITECTURE.md`
Describe the system at subsystem level:
- major runtime components
- control flow and data flow
- important boundaries and ownership
- key invariants discovered from code
- clearly labeled unknowns

### `LLM/COMPONENTS/<name>.md`
Create component summaries only for subsystems relevant to the task or clearly central to the repo. Each summary should include:
- purpose
- key files
- inputs and outputs
- invariants
- risk areas
- what to read first before editing

### `LLM/TASK_CONTEXT.md`
Use this as a working brief for the current task:
- task goal
- relevant files
- assumptions
- risks
- validation plan
- open questions

## Writing Rules For `LLM/`
- Prefer short sections and bullets over long prose.
- Include concrete file paths whenever possible.
- Optimize for retrieval and task safety, not completeness.
- Mark statements as `Verified`, `Inference`, or `Unknown` when appropriate.
- Record invariants and sharp edges explicitly.
- Do not restate obvious code; summarize behavior, constraints, and navigation guidance.

## Repo-Specific Guidance
- Start with `platformio.ini` when you need to understand build environments or test targets.
- Treat `FluidNC/src` as the main implementation root and use `FluidNC/tests` plus the host support directories to understand host-side validation paths.
- `fixture_tests` targets real hardware. Do not assume those tests are runnable in the local environment unless the task explicitly involves hardware-backed validation.
- Coverage artifacts in the repo root such as `coverage*.html`, `coverage*.json`, and `coverage*.txt` are generated outputs unless the task is specifically about coverage tooling or results.

## Validation Guidance
- Prefer the smallest relevant validation first.
- For host-side changes, use the appropriate PlatformIO host environment when possible, such as `windows_x86` or `posix`.
- Use `coverage.py` only when the task involves coverage generation or coverage regressions.
- If a task touches fixture behavior, identify the relevant fixture under `fixture_tests/fixtures` and note whether hardware execution was or was not performed.

## Code Review Workflow
When asked to review a branch, PR, or diff:
1. Determine the review base explicitly, typically the merge-base against the target branch.
2. Inspect all changed files plus the immediate surrounding code paths, including important callers, callees, tests, and build/config definitions.
3. Use `LLM/` docs as starting context only; verify all important claims against the diff and current source.
4. Run the smallest relevant validations first, then expand only if the risk profile justifies it.
5. Prioritize finding bugs, regressions, edge cases, and missing tests over summarizing the change.

### Review Priorities
- correctness bugs and behavioral regressions
- machine state transition issues involving idle, alarm, hold, resume, homing, unlock, e-stop, probe, spindle, and tool flows when relevant
- protocol and status-report compatibility breaks
- ESP32 vs host-test behavior drift
- error handling and recovery path weaknesses
- memory, ownership, lifetime, and concurrency issues where applicable
- config, persistence, and backward-compatibility problems
- missing, weak, or mis-scoped tests

### Review Output Contract
- Present findings first, ordered by severity.
- For each finding, include:
  - severity as `P0` to `P3`
  - why it is a problem
  - runtime or user-visible impact
  - precise file and line references
  - reasoning or validation evidence
  - suggested fix direction
- After findings, list open questions or assumptions.
- After that, list validations run, results, and remaining gaps.
- If there are no findings, say so explicitly and still report residual risks and unverified areas.

### Review Behavior Rules
- Do not stop at the diff if adjacent logic is necessary to validate behavior.
- Do not lead with a change summary before the findings.
- Do not say the change is safe unless the relevant code paths and validations were actually checked.
- Do not make code changes during a review-only request unless explicitly asked to remediate.
- If hardware-backed validation is relevant but unavailable, name the exact fixture or manual check that should be run and the risk it covers.

## Safety Rules
- Do not trust `LLM/` summaries blindly; verify them before editing.
- Do not present inferred architecture as fact.
- If the task only touches a narrow area, do not broaden the context-building scope unnecessarily.
- If you discover that an `LLM/` document is stale, update it as part of the task.
