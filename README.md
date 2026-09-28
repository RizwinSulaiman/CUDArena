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

## First milestone — M0

Build a small RX 9070 XT baseline suite of known passing and failing CUDA workloads, then prove that at least one agent can take a failing test, produce a patch, and make the real GPU verifier turn it green.

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

A headline score may exist later, but the raw compatibility matrix must remain visible.

## Current status

| Component | Status |
|---|---|
| Project definition | ✅ |
| Benchmark specification | ✅ |
| RX 9070 XT baseline design | ✅ |
| First reproducible task set | ⏳ |
| GPU verifier | ⏳ |
| First verified autonomous fix | ⏳ |
| Distributed runners | Later |
| Public arena / leaderboard | Later |

The immediate goal is to prove the agent → patch → RX 9070 XT → verified result loop before building the larger CUDArena website.

## License

Apache-2.0.
