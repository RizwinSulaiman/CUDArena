# CUDArena Benchmark Specification

CUDArena measures whether AI coding agents can turn reproducible CUDA-on-AMD failures into verified fixes.

## What counts as a benchmark task

Every task must include:

- unique task ID
- short problem statement
- pinned source revision when external code is involved
- target GPU and architecture
- exact build command
- exact run command
- expected behavior
- current failing behavior
- acceptance tests
- regression tests
- optional performance target

A task is invalid if success depends only on subjective review.

## Example

```yaml
id: runtime-0001
title: Fix asynchronous memset behavior
category: runtime
target:
  gpu: RX 9070 XT
  arch: gfx1201
baseline:
  status: fail
acceptance:
  - target test passes
  - stream ordering test passes
  - existing passing tests remain passing
scoring:
  correctness: required
  regressions: required
  performance: optional
```

## Categories

### Compiler
CUDA C++ language support, code generation, builtins, diagnostics, templates, device compilation, and linking.

### Runtime
Memory management, launches, streams, events, synchronization, device properties, graphs, errors, and related behavior.

### Device semantics
Warp/wave behavior, atomics, barriers, memory ordering, shared/local memory, intrinsics, texture/surface behavior, and architecture-sensitive semantics.

### Libraries
Compatibility involving common CUDA-facing libraries and interfaces.

### Applications
End-to-end open-source CUDA workloads.

### Performance
Correct workloads that run but have a measurable compatibility-layer performance defect.

## Result states

- `PASS` — acceptance criteria met and regression checks clear.
- `PARTIAL` — measurable progress, but acceptance criteria are not fully met.
- `FAIL` — no qualifying improvement.
- `REGRESSION` — target improves but previously passing behavior breaks.
- `INVALID` — task or run is not reproducible enough to score.

## Verified agent run

A verified run must record:

- model
- agent harness
- starting commit
- task prompt
- tool permissions
- budget or limits
- attempt count
- produced patch
- test output
- hardware result
- final verdict

Human-authored fixes are welcome, but they do not count toward the autonomous-agent leaderboard.

## Scoring

Correctness is mandatory. A simple initial score can be:

- +100: target task passes
- +25: adds a useful regression test
- +25: fixes the same issue on an additional supported AMD architecture
- +0 to +50: meaningful measured performance improvement
- 0: partial or failed result
- disqualified: regression, unverifiable run, or falsified metadata

The scoring system can evolve only when the benchmark has enough real tasks to justify it.

## Compatibility percentage

CUDArena must never publish a single “CUDA compatibility %” without defining its denominator.

Possible views include:

- task pass rate
- application pass rate
- weighted real-workload score
- per-category score
- per-GPU score

The raw task matrix must always remain visible behind any headline number.
