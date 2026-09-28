# CUDArena Roadmap

CUDArena is an agent-driven compatibility arena: AI coding agents attempt reproducible CUDA-on-AMD tasks, and real hardware decides whether their patches work.

The roadmap is intentionally milestone-based. We do not chase “100% CUDA compatibility” as a single goal.

## M0 — Prove the loop

Target hardware: AMD Radeon RX 9070 XT / RDNA 4 / gfx1201.

- Define a small baseline of reproducible CUDA compatibility tests.
- Record pass/fail results and exact reproduction steps.
- Select at least one small, known failure.
- Give that failure to an autonomous coding agent.
- Verify the candidate patch on real AMD hardware.
- Accept only if the target failure is fixed without regressions.

**Exit condition:** one complete task goes from red → agent patch → real GPU verification → green.

## M1 — Build the benchmark

- Grow from toy kernels to representative real workloads.
- Classify tasks by compiler, runtime, PTX/device semantics, libraries, performance, and application compatibility.
- Add deterministic scoring.
- Record model, harness, budget, attempts, wall-clock time, and outcome for verified agent runs.
- Keep human-authored fixes separate from verified autonomous runs.

**Exit condition:** enough tasks exist to compare coding agents meaningfully.

## M2 — Distributed hardware verification

- Define a runner protocol for contributed AMD GPUs.
- Record GPU model, architecture target, driver/runtime versions, and test provenance.
- Require reproducible result artifacts.
- Detect flaky or inconsistent runners.
- Expand beyond gfx1201 without weakening the original reference baseline.

**Exit condition:** the same task can be independently verified on multiple AMD systems.

## M3 — Real software

Prioritize useful workloads over obscure API-count chasing.

Candidate classes:

- CUDA sample programs
- CUDA-heavy open-source tools
- numerical/scientific kernels
- rendering workloads
- ML inference/training components
- custom CUDA extensions

**Exit condition:** compatibility improvements visibly unlock real software.

## M4 — Public arena

Only after the underlying benchmark is trustworthy:

- public task board
- model/harness leaderboard
- compatibility dashboard
- per-GPU results
- patch provenance
- cost-efficiency metrics
- regression history

The website is the interface. The benchmark and verifier are the project.

## Principles

1. Real workloads beat vanity percentages.
2. Reproducibility beats screenshots.
3. Hardware verification beats agent claims.
4. Small falsifiable tasks beat giant vague missions.
5. Existing open work should be reused before rebuilding.
6. Correctness comes before performance; performance becomes a scored dimension once correctness passes.
