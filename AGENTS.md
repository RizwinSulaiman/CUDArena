# Agent-first contribution contract

CUDArena is designed to measure and improve autonomous software-engineering capability while producing useful CUDA-on-AMD compatibility work.

## Verified agent runs

A contribution counts as a verified agent run when the coding agent is launched through a recorded harness with a fixed task, repository state, environment, and resource budget. The agent may inspect code, edit files, run tests, and submit a candidate patch.

The final result must be judged by reproducible automated tests and, where required, a real AMD GPU runner.

## Community contributions

Human-authored contributions are welcome, but they are tracked separately from verified agent runs so the benchmark remains meaningful.

## Acceptance principle

A patch is not accepted because an agent says it works. It must demonstrate measurable improvement, pass the target test, and avoid regressions in the existing suite.

## First hardware target

AMD Radeon RX 9070 XT / RDNA 4 / gfx1201.
