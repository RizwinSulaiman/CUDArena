# Contributing to CUDArena

CUDArena welcomes both autonomous-agent runs and normal human contributions, but they are tracked separately.

## Before contributing

1. Read `README.md`.
2. Read `BENCHMARK_SPEC.md`.
3. Check existing issues before creating a duplicate task.
4. Prefer small reproducible failures over broad “support X” requests.
5. Reuse existing open-source CUDA/AMD compatibility work where possible instead of rebuilding blindly.

## Two contribution tracks

### Verified agent contribution

A verified agent run must preserve enough provenance to show what the agent actually did.

Include:

- model name/version when available
- agent harness
- starting commit
- exact task/prompt
- relevant limits or budget
- generated patch
- tests run by the agent
- final hardware verification result

A human may set up the task and environment, but should not secretly write the solution while claiming an autonomous solve.

### Community contribution

Normal human-authored fixes, tests, docs, research, and infrastructure are welcome. They improve the project but are not counted on the autonomous-agent leaderboard.

## Good benchmark tasks

Good:

- one CUDA sample fails with a deterministic compiler error
- one runtime API returns incorrect results
- one synchronization semantic differs from the reference behavior
- one real application fails at a reproducible step
- one working workload is dramatically slower for a specific measurable reason

Bad:

- “make PyTorch work”
- “support all of CUDA graphs”
- “optimize everything”
- failures that cannot be reproduced
- benchmarks whose expected result is unknown

## Pull requests

A useful PR should say:

- what failed before
- why the change fixes it
- which tests prove the fix
- which regression tests were run
- which AMD GPU/architecture verified it, if GPU verification was required
- whether the patch was agent-authored, human-authored, or mixed

## Correctness first

A faster wrong answer is a failure.

Performance work is scored only after correctness passes.

## Clean-room and licensing

Only contribute code and material you have the right to contribute. Do not copy proprietary CUDA/NVIDIA implementation code, leaked source, or confidential material. Compatibility work should rely on public behavior, public documentation, permitted testing, and appropriately licensed open-source dependencies.
