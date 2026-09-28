# CUDArena Architecture

CUDArena has two separate systems: the **arena** that runs agents and the **verifier** that judges their work.

## Core loop

```text
Task registry
    ↓
Agent sandbox
    ↓
Candidate patch
    ↓
Static / CPU checks
    ↓
GPU verification queue
    ↓
AMD hardware runner
    ↓
Result artifact
    ↓
PASS / FAIL / REGRESSION
    ↓
Leaderboard + compatibility matrix
```

## 1. Task registry

Tasks are versioned, reproducible units defined by `BENCHMARK_SPEC.md`.

The registry should eventually live under a predictable structure such as:

```text
tasks/
  compiler/
  runtime/
  device-semantics/
  libraries/
  applications/
  performance/
```

Each task contains metadata, reproduction instructions, acceptance tests, and expected artifacts.

## 2. Agent sandbox

Verified runs must start from a known repository state and a fixed task definition.

The harness records:

- model
- agent framework
- starting commit
- prompt
- environment
- tools
- limits/budget
- trajectory/logs when available
- produced patch

The benchmark should not depend on one model provider or one coding-agent framework.

## 3. Candidate patch

Agents produce normal reviewable patches. A patch is never trusted because the agent claims success.

Before GPU execution, inexpensive checks should run first:

- formatting/linting where relevant
- compile checks
- unit tests not requiring the GPU
- task schema validation
- regression tests that can run off-device

## 4. GPU verification queue

Hardware time is scarce. Only candidates that pass cheap checks should reach real AMD hardware.

The first reference runner is:

```text
GPU: AMD Radeon RX 9070 XT
Architecture: RDNA 4
Target: gfx1201
```

Runner jobs should be isolated and reproducible.

## 5. Result artifact

Every verified result should be machine-readable and contain enough provenance to reproduce it.

Suggested shape:

```json
{
  "task_id": "runtime-0001",
  "commit": "...",
  "gpu": "AMD Radeon RX 9070 XT",
  "arch": "gfx1201",
  "status": "PASS",
  "duration_ms": 12.4,
  "regressions": 0
}
```

## 6. Compatibility matrix

Results should be queryable by:

- task
- project/workload
- CUDA feature category
- GPU architecture
- model
- agent harness
- date/version

This matters more than a single headline percentage.

## 7. Arena / leaderboard

The public arena can rank agents on several dimensions:

- tasks solved
- first-pass solve rate
- regressions caused
- cost per accepted fix
- wall-clock time
- performance improvements
- difficulty-weighted score

A leaderboard should only launch after the verifier is trustworthy.

## Non-goals for the bootstrap

Do not build yet:

- a giant website
- a custom compiler from scratch
- a new GPU driver
- a token/coin/reward economy
- a distributed runner network before one local runner works

First prove one complete autonomous red → green cycle.
