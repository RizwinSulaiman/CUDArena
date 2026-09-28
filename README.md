# CUDArena

**Where AI agents compete to make CUDA workloads run on AMD GPUs.**

CUDArena is an open experiment and engineering arena: can autonomous coding agents systematically turn failing CUDA workloads into passing ones on AMD hardware?

The first reference target is **AMD Radeon RX 9070 XT (RDNA 4 / gfx1201)**.

## The loop

```text
reproducible CUDA failure
        ↓
AI coding agent
        ↓
candidate patch
        ↓
cheap checks
        ↓
real AMD GPU verification
        ↓
PASS / FAIL / REGRESSION
        ↓
compatibility matrix + agent leaderboard
```

1. Pick a reproducible CUDA workload or conformance test that fails on AMD.
2. Give the task to an AI coding agent in a controlled workspace.
3. Let the agent inspect, patch, build, and test.
4. Run the candidate on real AMD hardware.
5. Accept only patches that improve correctness without regressions.
6. Record the result and provenance.

## What this is

- A real open-source CUDA compatibility project.
- A benchmark for autonomous software-engineering agents.
- A hardware-backed arena where GPUs, not claims, decide whether a patch works.
- A way to measure practical CUDA-on-AMD compatibility with reproducible workloads.

## What this is not

- A promise of 100% CUDA compatibility.
- A plan to rewrite the entire CUDA ecosystem from scratch.
- A benchmark where passing toy tests matters more than real software.
- A leaderboard built before the verifier is trustworthy.

## Project docs

- [Roadmap](ROADMAP.md) — milestone-by-milestone plan.
- [Benchmark specification](BENCHMARK_SPEC.md) — what counts as a task and how runs are scored.
- [Architecture](ARCHITECTURE.md) — agent sandbox, verifier, GPU runner, result pipeline, and future arena.
- [Agent contract](AGENTS.md) — rules for verified autonomous runs.
- [Contributing](CONTRIBUTING.md) — contribution tracks and PR expectations.
- [RX 9070 XT baseline plan](docs/BASELINE_PLAN.md) — the first ~20-task reference suite.
- [Site prototype](site/index.html) — benchmark-first light UI inspired by serious public benchmark sites.

## First milestone — M0

1. Establish a simple 10–20 workload RX 9070 XT pass/fail baseline.
2. Pick one small failure.
3. Prove one agent-authored red → green fix on the real GPU without regressions.

**M0 exit condition:** one complete autonomous red → green cycle with reproducible evidence and no regression.

## Benchmark philosophy

CUDArena does not treat “CUDA compatibility” as one vague percentage. Results should eventually be visible by:

- task
- real application/workload
- compiler/runtime/device-semantics category
- AMD GPU architecture
- model
- coding-agent harness
- cost/time
- correctness and performance

A headline score may exist, but the raw compatibility matrix must remain visible.

## Website direction

The product UI is intentionally **benchmark-first, light, restrained, and data-dense** rather than a neon/gaming dashboard. The prototype lives in `site/` and includes a leaderboard, filters/search, recent runs, benchmark categories, and summary metrics.

**Important:** the current website numbers and model rows are placeholder demo data only. They must not be presented as measured CUDArena results. Real values replace them only after the reference baseline and verifier exist.

## Current status

| Component | Status |
|---|---|
| Project definition | ✅ |
| Benchmark specification | ✅ |
| RX 9070 XT baseline design | ✅ |
| Benchmark-first site prototype | ✅ |
| First reproducible task set | ⏳ |
| GPU verifier | ⏳ |
| First verified autonomous fix | ⏳ |
| Distributed runners | Later |
| Public verified leaderboard | Later |

The immediate engineering goal remains **Issue #1: establish the real RX 9070 XT baseline**. The site can mature in parallel, but measured data wins over presentation.

## License

Apache-2.0.
