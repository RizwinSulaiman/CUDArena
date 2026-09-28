# Warp2Wave

**An agent-driven arena for expanding CUDA workload compatibility on AMD GPUs.**

Warp2Wave is an open experiment: can autonomous coding agents systematically turn failing CUDA workloads into passing ones on AMD hardware?

The first reference target is **AMD Radeon RX 9070 XT (RDNA 4 / gfx1201)**.

## The loop

1. Pick a reproducible CUDA workload or conformance test that fails on AMD.
2. Give the task to an AI coding agent in a controlled workspace.
3. Let the agent inspect, patch, build, and test.
4. Run the candidate on real AMD hardware.
5. Accept only patches that improve correctness without regressions.
6. Record the result on an agent/model leaderboard.

## What this is

- A real open-source compatibility project.
- A benchmark for autonomous software-engineering agents.
- A hardware-backed test arena where GPUs, not vibes, decide whether a patch works.
- A way to measure practical CUDA-on-AMD compatibility with reproducible workloads.

## What this is not

- A promise of 100% CUDA compatibility.
- A plan to rewrite the entire CUDA ecosystem from scratch.
- A benchmark where passing toy tests matters more than real software.

## First milestone

Build a small RX 9070 XT baseline suite of known passing and failing CUDA workloads, then prove that at least one agent can take a failing test, produce a patch, and make the real GPU verifier turn it green.

## Status

**Bootstrap / experiment design.**

The immediate goal is to prove the agent -> patch -> RX 9070 XT -> verified result loop before building the larger arena website.

## License

Apache-2.0.
