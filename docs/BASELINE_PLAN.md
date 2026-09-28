# RX 9070 XT Baseline Plan

The first CUDArena benchmark target is **AMD Radeon RX 9070 XT / RDNA 4 / gfx1201**.

This document defines the smallest useful baseline before any public leaderboard or arena website exists.

## Goal

Create approximately 20 reproducible tasks that cover a useful cross-section of CUDA behavior and include both known passing and known failing cases.

The baseline is not intended to estimate all of CUDA. It exists to prove the benchmark and autonomous repair loop.

## Suggested baseline buckets

### A. Kernel fundamentals

1. vector add
2. 1D indexing
3. 2D indexing
4. shared memory
5. block synchronization

### B. Memory/runtime

6. device allocation/free
7. host-to-device copy
8. device-to-host copy
9. device memset
10. asynchronous copy or memset

### C. Execution semantics

11. atomics
12. warp-level shuffle
13. warp vote
14. stream ordering
15. events/timing

### D. Real software

16. one small CUDA sample project
17. one compute-oriented open-source CUDA project
18. one CUDA library-facing example
19. one known currently failing real workload
20. one performance-sensitive workload that already runs correctly

The exact tasks should be selected from evidence, not assumed from this list.

## Required record for each baseline item

```text
Task ID:
Source + pinned revision:
Category:
Expected result:
Build command:
Run command:
RX 9070 XT result:
Status: PASS / FAIL / INVALID
Failure signature:
Notes:
```

## Selecting the first autonomous repair task

Prefer a failure that is:

- deterministic
- small enough to understand in one repository session
- backed by an acceptance test
- likely isolated to one subsystem
- valuable enough to demonstrate real progress

Avoid starting with PyTorch, vLLM, a full CUDA library, or a giant compiler feature.

## Success condition

CUDArena M0 is complete when:

1. a task is confirmed failing on the RX 9070 XT;
2. an autonomous coding agent receives a frozen task and repository state;
3. the agent produces a patch without a human writing the solution;
4. the patch passes cheap checks;
5. the RX 9070 XT verifies the target task as fixed;
6. regression tests remain green;
7. the run metadata and result are recorded.

One clean red → green result is worth more than a hundred speculative tasks.
