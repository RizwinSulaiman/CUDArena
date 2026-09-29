# CUDArena — Adversarial Review Dossier for Claude

> **Purpose:** This document intentionally over-exposes the CUDArena idea so an external reviewer can attack it from every angle: technical architecture, benchmark validity, product strategy, site design, agent-evaluation methodology, legal/security risk, and the central thesis that **LLM workloads and LLM coding agents can both be used as meaningful benchmarks around CUDA compatibility**.
>
> **Reviewer instruction:** Do not be polite. Do not preserve the current architecture out of sunk-cost bias. Assume the author is wrong until the design survives attack. If a simpler or fundamentally different approach is better, say so explicitly and redesign it.

---

# 0. The single-sentence goal

**Build an open, adaptive CUDA-compatible platform for AMD/non-NVIDIA GPUs, while using real LLM workloads to measure whether CUDA compatibility is actually useful and using autonomous coding agents to discover and repair compatibility failures.**

The working project name is **CUDArena**.

The intended flywheel is:

```text
real CUDA/LLM workload
        ↓
benchmark discovers incompatibility
        ↓
turn failure into agent mission
        ↓
LLM coding agent attempts repair
        ↓
real AMD GPU verifies patch
        ↓
compatibility improves
        ↓
benchmark reruns
        ↓
score / real software coverage improves
```

The user ultimately wants something that feels like **"Proton for CUDA"**:

```text
CUDA program
    ↓
CUDArena
    ↓
whatever compatible translation/runtime/library route works best
    ↓
AMD GPU
```

The user should not need to care whether a workload was handled through a source compiler, PTX translator, binary compatibility layer, native CUDArena runtime implementation, ROCm-backed library adapter, or a fallback provider.

---

# 1. Important distinction: CUDArena may actually be THREE projects

A recurring source of confusion is mixing these together.

## 1.1 CUDArena Core

The compatibility platform itself.

Potential responsibilities:

- CUDA-facing loader / API front door
- provider discovery
- workload inspection
- execution planning
- capability matching
- common runtime/context abstractions
- memory ownership / allocator coordination
- stream/event coordination
- module/kernel dispatch
- library interception/adapters
- compatibility database
- provider fallback
- eventually native CUDA-compatible implementations

## 1.2 CUDArena Bench

The measurement system.

Questions it should answer:

- Which CUDA workloads run?
- On which AMD GPUs?
- Through which compatibility provider or provider combination?
- With what correctness?
- With what performance relative to a reference?
- Which CUDA features are missing?
- Which real applications are unlocked by a given fix?
- Which regressions appeared?

## 1.3 CUDArena Arena

The autonomous engineering benchmark.

Questions it should answer:

- Can an agent diagnose a real CUDA compatibility failure?
- Can it produce a correct patch?
- Can it do so without human implementation assistance?
- What did it cost?
- How long did it take?
- How many attempts?
- Did it cause regressions?
- Did it unlock additional real workloads?

These three systems feed each other but should not share one vague score.

```text
Bench finds failures
       ↓
Arena creates tasks
       ↓
Agents produce fixes
       ↓
Core/providers improve
       ↓
Bench measures improvement
```

**Adversarial question:** Is this decomposition actually correct, or should the project be split into separate repositories/products from day one?

---

# 2. How the idea evolved

The project began with a naive but ambitious thought:

> "Can we make CUDA for AMD?"

That immediately ran into the obvious problem: CUDA is not one thing. It includes language semantics, runtime APIs, driver APIs, PTX, proprietary libraries, compiler behavior, tooling, binary compatibility, synchronization rules, memory semantics, and years of ecosystem assumptions.

Then the idea shifted into:

> "What if we build a public benchmark/arena where AI agents fix CUDA-on-AMD failures one by one?"

This was stronger because it reframed the huge problem into many small falsifiable tasks.

Then another mistake happened: the bootstrap plan (pick one backend, find one failing test, let an agent fix it) quietly became the assumed long-term architecture.

That is now considered too narrow.

The newer thesis is:

> **CUDArena should remain backend-neutral and become an adaptive compatibility platform that can use multiple execution providers.**

Instead of asking:

> Which CUDA-on-AMD project wins?

ask:

> What capability can each existing project contribute to a unified compatibility stack, and where should CUDArena eventually replace external providers with native components?

---

# 3. North-star architecture: loader + planner + execution providers

The current strongest architecture is inspired conceptually by systems such as driver loaders and execution-provider runtimes.

```text
                    CUDA APPLICATION
                           │
                           ▼
              ┌────────────────────────┐
              │    CUDArena Loader     │
              │ CUDA-compatible front  │
              │ door / ABI surface     │
              └───────────┬────────────┘
                          │
                 inspect requirements
                          │
                          ▼
              ┌────────────────────────┐
              │  Planner / Core State  │
              │ contexts               │
              │ allocations            │
              │ streams/events         │
              │ modules                │
              │ errors                 │
              │ device properties      │
              └───────────┬────────────┘
                          │
                  Provider interface
                          │
       ┌──────────────────┼────────────────────┐
       │                  │                    │
       ▼                  ▼                    ▼
 source/compiler      PTX provider       binary/runtime
 provider             provider           provider
       │                  │                    │
       ├──────────────────┼────────────────────┤
       │                  │                    │
       ▼                  ▼                    ▼
 cuBLAS adapter      cuFFT adapter        cuDNN adapter
 → rocBLAS          → rocFFT             → MIOpen/etc.
       │                  │                    │
       └──────────────────┼────────────────────┘
                          ▼
                    ROCm / HSA / LLVM
                          ▼
                        AMD GPU
```

## 3.1 Key principle

**All-in-one should mean one user experience, not one monolithic implementation.**

The user experience should ideally be:

```text
install CUDArena
run CUDA software
```

Internally CUDArena may contain many providers.

---

# 4. Why NOT simply pick one backend forever?

Candidate projects may target fundamentally different layers.

Working assumptions that the reviewer should independently verify:

- **ZLUDA-like path:** drop-in / runtime / binary-oriented compatibility for existing CUDA software.
- **BarraCUDA-like path:** source/compiler-oriented CUDA C++ → AMD machine code.
- **SCALE-like path:** source-level `nvcc`-style CUDA compiler/runtime targeting AMD; useful as a reference/control even if licensing limits direct reuse.
- **ROCm libraries:** native AMD implementations that may serve as target adapters for CUDA-X-style APIs.
- **LLVM AMDGPU:** powerful existing code-generation foundation.

These are not necessarily substitutes for one another.

A single winner may leave huge capability gaps.

**Reviewer challenge:** Determine whether provider composition creates more complexity than simply choosing one architecture and extending it aggressively.

---

# 5. Why NOT route every CUDA call independently?

A naive dynamic system might do:

```text
cudaMalloc()      → Backend A
cudaMemcpy()      → Backend B
kernel launch     → Backend C
cuBLAS call       → Backend D
cudaEventRecord() → Backend E
```

This is likely disastrous unless all providers share compatible state.

They must agree on:

- context identity
- device identity
- GPU virtual addresses
- allocation ownership
- stream semantics
- event semantics
- synchronization
- module lifetime
- error behavior
- kernel argument ABI
- library handles

Otherwise one provider receives opaque state created by another and fails.

Therefore the current idea is **coarse routing at stable boundaries**, not arbitrary per-call switching.

Potential stable boundaries:

### Application boundary

Route the entire workload through one provider when that is the safest path.

### Build boundary

If source is available, choose a source compiler path.

### Module boundary

Assign a PTX/module unit to a provider.

### Library boundary

Intercept cuBLAS/cuFFT/cuDNN-style calls and route them to native AMD adapters.

### Future finer-grained routing

Only after a shared CUDArena state/runtime layer is proven.

**Reviewer challenge:** Are even these boundaries too ambitious? Which boundary should exist first?

---

# 6. CUDArena execution plan concept

Instead of hidden magic, CUDArena could generate a transparent plan.

Example:

```text
CUDArena Execution Plan

Application:
    ExampleLLM

GPU:
    RX 9070 XT / gfx1201

Requirements detected:
    CUDA Runtime         19 APIs
    Driver API            7 APIs
    PTX                  14 kernels
    cuBLAS                yes
    cuFFT                 no
    cuDNN                 yes
    custom extension      yes

Plan:
    runtime               CUDArena Core
    PTX modules           Provider A
    cuBLAS                rocBLAS adapter
    cuDNN                 provider B
    unsupported calls     fallback provider C

Known risks:
    graph API partial
    cooperative groups unsupported

Expected confidence:
    0.84
```

This could be cached per workload/version/GPU.

Potential future modes:

```text
cudarena inspect ./app
cudarena run ./app
cudarena explain ./app
cudarena benchmark ./app
```

**Reviewer challenge:** Is static inspection of CUDA requirements realistic enough to be useful, or will runtime behavior make planning unreliable?

---

# 7. Provider capability interface

Potential provider manifest:

```json
{
  "provider": "example-provider",
  "version": "0.1.0",
  "targets": ["gfx1201"],
  "capabilities": {
    "cuda_source": true,
    "ptx": true,
    "cuda_binary": false,
    "driver_api": "partial",
    "runtime_api": "partial",
    "streams": true,
    "events": true,
    "graphs": false,
    "warp_shuffle": true,
    "cublas": false
  }
}
```

Potential minimum Provider ABI v0:

```text
probe()
get_capabilities()
can_handle(requirements)
prepare(plan_fragment)
load_module()
launch_kernel()
get_diagnostics()
```

A deliberately tiny v0 could initially only perform **provider selection**, not live mixed execution.

Example bootstrap:

```text
CUDArena inspect workload
   ↓
ZLUDA-like provider: 82% requirement coverage
source provider:      61%
other provider:       43%
   ↓
select one whole-workload provider
```

Then later evolve toward library/module composition.

**Reviewer challenge:** Should the Provider ABI be the first thing CUDArena itself implements, or is that still premature abstraction before observing real workloads?

---

# 8. Common state layer: likely long-term heart of CUDArena

If mixed-provider execution becomes real, CUDArena likely must own canonical representations of:

```text
CUcontext
CUdevice
CUdeviceptr
cudaStream_t
cudaEvent_t
CUmodule
CUfunction
library handles
error state
```

Potential design:

```text
CUDA-compatible API
        ↓
CUDArena handle/state layer
        ↓
provider-specific translation
        ↓
AMD-native resources
```

This layer would manage or coordinate:

- allocation registry
- pointer provenance
- stream/event ordering
- module ownership
- device properties
- synchronization contracts
- error translation

It might eventually enable cross-provider interop.

But it may also become an enormous compatibility nightmare.

**Reviewer challenge:** Is a universal state layer strategically correct, or should CUDArena instead enforce strong provider isolation and avoid cross-provider sharing entirely?

---

# 9. Dependency absorption strategy

CUDArena does not need to rewrite CUDA from scratch on day one.

Proposed progression:

```text
early CUDArena
├── external provider A          heavy reliance
├── external provider B          heavy reliance
├── ROCm adapters                moderate reliance
└── CUDArena-native              small
```

As agents repeatedly solve failures:

```text
mature CUDArena
├── external providers           fallback / edge cases
└── CUDArena-native              majority path
```

Native pieces should be added only when they provide a concrete benefit:

- remove a recurring provider failure
- improve compatibility
- improve performance
- simplify state sharing
- reduce licensing/dependency problems
- unlock important workloads

This avoids the psychologically attractive but probably impossible declaration:

> "We are going to implement all of CUDA ourselves."

Instead, CUDArena gradually absorbs proven capability.

**Reviewer challenge:** Does gradual absorption produce an incoherent codebase and permanent compatibility debt? Would a clean reimplementation actually be better past some threshold?

---

# 10. The user's special benchmark goal: LLMs as a CUDA benchmark

This is extremely important and should not get lost behind microbenchmarks.

There are TWO distinct LLM roles.

## 10.1 LLM workloads benchmark CUDA compatibility

The user's practical question is not merely:

> Does `cudaMalloc()` work?

It is closer to:

> **Can a modern LLM stack that expects CUDA actually run correctly and efficiently on this AMD GPU through CUDArena?**

This makes LLM stacks potentially excellent end-to-end stress tests because they exercise many CUDA assumptions simultaneously.

Candidate LLM workload dimensions:

- llama.cpp CUDA backend
- PyTorch CUDA inference
- custom CUDA extensions
- attention kernels
- GEMM-heavy paths
- quantization kernels
- KV-cache management
- multi-stream execution
- fused kernels
- memory pressure
- graph capture where used
- NCCL-style multi-GPU eventually
- training/finetuning later
- vLLM-like serving later
- Triton-generated paths where relevant

Potential tiers:

### Tier A — tiny deterministic model workload

Small model, tiny context, deterministic output, minimal dependencies.

Goal: prove end-to-end correctness.

### Tier B — representative inference

Real model size that fits the RX 9070 XT.

Measure:

- load success
- tokens/sec
- first-token latency
- memory usage
- numerical/output agreement
- crashes/hangs

### Tier C — modern serving stack

Stress graph/runtime/library behavior.

### Tier D — training/finetuning

Only later because complexity explodes.

The benchmark should answer:

```text
Does this CUDA software stack run?
Is its output correct enough?
How fast relative to NVIDIA reference / native AMD path?
Which provider plan was used?
Which CUDA features were required?
What failed?
```

## 10.2 LLM coding agents benchmark autonomous repair capability

Separately, LLM agents are contestants.

Example mission:

```text
Mission 0042

Workload:
    LLM inference smoke test

Failure:
    warp shuffle semantic mismatch

Target:
    RX 9070 XT / gfx1201

Starting revision:
    pinned

Acceptance:
    target test passes
    deterministic LLM output within tolerance
    baseline regression suite remains green
    no unsupported test bypass
```

The agent receives repository access and tools, attempts diagnosis/patching, and the GPU verifier judges the result.

This creates a powerful dual benchmark:

```text
LLMs benchmark CUDArena by RUNNING workloads

and

LLMs benchmark themselves by FIXING CUDArena
```

This duality may be one of the project's strongest differentiators.

**Reviewer challenge:** Is this genuinely meaningful or merely clever branding? Does it create scientifically useful measurements?

---

# 11. Benchmark philosophy

CUDArena must avoid turning into API-count theater.

Bad headline:

> "CUDArena supports 63% of CUDA."

Unless the denominator and weighting are extremely explicit, this is misleading.

Preferred views:

- CUDArena suite pass rate
- per-category pass rate
- per-provider pass rate
- per-GPU pass rate
- real-application success rate
- LLM workload success rate
- weighted workload coverage
- correctness score
- performance ratio

The raw compatibility matrix should remain visible.

Example:

| Workload | Provider A | Provider B | Hybrid CUDArena | Native NVIDIA reference |
|---|---:|---:|---:|---:|
| vector add | PASS | PASS | PASS | PASS |
| warp shuffle | FAIL | PASS | PASS | PASS |
| llama.cpp smoke | FAIL | PARTIAL | PASS | PASS |
| PyTorch extension | FAIL | FAIL | PARTIAL | PASS |

Possible states:

```text
PASS
PARTIAL
FAIL
REGRESSION
UNSUPPORTED
NOT_APPLICABLE
INVALID
FLAKY
```

**Reviewer challenge:** What benchmark taxonomy prevents gaming and meaningless score inflation?

---

# 12. M0: what should be proven first?

There has been debate about this.

Earlier plan:

```text
pick one backend
10–20 tests
find one FAIL
agent fixes it
```

Current revised thinking:

### M0-A — characterize a few providers

Use perhaps 5–8 tiny common tests:

```text
vector add
shared memory
barrier/sync
atomicAdd
warp shuffle
streams/events
simple GEMM
one tiny real CUDA workload
```

Run through credible available provider paths where applicable.

The goal is not a universal score. It is to learn where the real architecture boundaries are.

### M0-B — choose one genuine repair target

Not "choose one permanent substrate".

Choose the easiest valuable reproducible failure in an open provider.

### M0-C — autonomous red → green proof

```text
confirmed FAIL
      ↓
frozen task
      ↓
agent run
      ↓
patch
      ↓
real RX 9070 XT validation
      ↓
PASS with no regression
```

### M0-D — only then formalize more architecture

Use evidence from the first real run to refine:

- task schema
- result schema
- provider interface
- benchmark weighting
- website data model

**Reviewer challenge:** Should even the provider abstraction wait until after several real failures have been studied?

---

# 13. Reference hardware

Initial reference machine:

```text
AMD Radeon RX 9070 XT
RDNA 4
Target: gfx1201
```

Reasons:

- actual hardware available to the project owner
- modern AMD consumer architecture
- useful test case for whether compatibility stacks support current hardware

Eventually additional GPUs may be added:

```text
RDNA 4
RDNA 3
RDNA 2
MI-series / CDNA possibly
```

But distributed runners should wait until one trustworthy local validation loop exists.

---

# 14. Site/product vision

The website should look like a serious benchmark/research product.

## Desired style

- professional
- light neutral theme by default
- restrained teal/green identity
- highly readable tables
- research/benchmark credibility
- provenance-first
- compact but not cramped
- benchmark data visually dominates marketing copy

## Explicitly avoid

- cyberpunk neon
- giant glowing GPU renders
- crypto aesthetic
- gaming dashboard look
- fake sci-fi diagrams
- excessive gradients
- huge marketing hero sections
- video-game points/trophies as the primary visual metaphor

## Homepage hierarchy

```text
CUDArena wordmark + nav
      ↓
one-sentence mission
      ↓
small benchmark summary
      ↓
primary compatibility / agent table
      ↓
recent verified runs
      ↓
benchmark categories
      ↓
provider/GPU coverage
      ↓
methodology / provenance
```

## Potential primary tables

### Compatibility view

| Workload | CUDArena | Provider A | Provider B | GPU | Correctness | Perf |

### Agent view

| Model/Agent | Verified fixes | Solve rate | Regressions | Cost | Time |

### LLM workload view

| LLM stack | Model | GPU | Provider plan | Status | tok/s | Memory |

The website should support drilling from a score into evidence.

Every real result should expose:

- task/workload ID
- source revision
- model revision where relevant
- provider versions
- GPU
- driver/runtime versions
- exact commands
- logs
- artifact references
- patch/commit
- agent/model/harness
- final verifier status

---

# 15. Current site prototype problem

The repo currently contains a static visual prototype with fabricated demo metrics and real model/vendor names.

That is dangerous because benchmark credibility depends on data integrity.

Even if the page says "prototype" at the bottom, a prominent display like:

```text
39% compatibility
312 / 803 tests
1,284 verified runs
```

can be screenshotted out of context.

Recommended correction before public promotion:

```text
Compatibility Score: —
Verified Runs: —
GPU Runners: 1 reference machine (when true)
Status: PRE-BASELINE
```

Demo rows should use clearly fictional labels:

```text
Demo Agent A
Demo Agent B
```

or the entire page should carry an obvious **DEMO DATA** banner.

**Reviewer challenge:** Should the public site even exist before the first verified hardware result?

---

# 16. Benchmark data integrity

For a benchmark project, credibility is the core product.

Potential rules:

1. No result without reproducible artifact/provenance.
2. No score without a defined denominator.
3. No leaderboard row without a verified run record.
4. Human-authored fixes separate from autonomous-agent fixes.
5. Regressions visible, not silently discarded.
6. Provider/version/GPU pinned.
7. Benchmark changes versioned.
8. Score changes explainable.
9. Failed/invalid runs retained where useful.
10. No marketing number that cannot be regenerated from committed data.

**Reviewer challenge:** What additional governance is required for this to be taken seriously by compiler/GPU researchers?

---

# 17. Agent benchmark integrity

Public coding benchmarks become contaminated quickly.

If task #37 is public and solved in a merged PR, later agents can retrieve the answer.

Long-term split may be required:

```text
PUBLIC DEVELOPMENT SET
known failures
public patches
open collaboration

SEALED EVALUATION SET
unpublished failures
same task format
used for model comparisons
```

Potential intervention classes:

```text
L0 — fully autonomous, no human hints
L1 — automatic retries allowed, no semantic hints
L2 — human steering/prompt updates allowed
L3 — human+agent collaborative engineering
```

Primary leaderboard should probably use only one clearly defined class.

Need to record:

- model exact version
- harness
- system prompt
- task prompt
- tools
- internet/search access
- starting repository state
- attempts
- budget/tokens/cost
- wall-clock time
- human interventions
- final patch

**Reviewer challenge:** Can a public agent leaderboard remain scientifically meaningful as models train on prior CUDArena tasks?

---

# 18. Difficulty and scoring

Naive points are easily gamed.

Example bad system:

```text
+100 fixed task
+25 added test
+50 performance
```

A one-line API stub and a deep compiler correctness bug should not be worth similar amounts.

Potential empirical difficulty signals:

- fraction of agents that solve the task
- median attempts
- median wall time
- median token/cost usage
- number of providers affected
- number of workloads unlocked
- regression surface
- complexity of acceptance suite

Potential agent metrics:

```text
verified solves
first-attempt solve rate
cost per solve
median time to solve
regressions per accepted fix
coverage unlocked
performance improvement
sealed-set score
```

Potential compatibility metrics:

```text
task pass rate
real-workload pass rate
LLM workload pass rate
weighted application coverage
performance ratio vs reference
stability/flakiness
```

**Reviewer challenge:** Propose a scoring model that is difficult to game and still understandable to users.

---

# 19. LLM workload benchmark design in more depth

Because the user's goal is specifically to use **LLMs as a benchmark for CUDA**, the benchmark should probably not stop at toy kernels.

Potential matrix:

## 19.1 Framework axis

```text
llama.cpp
PyTorch
custom CUDA extension package
vLLM-like serving
possibly TensorRT-LLM-like workload only where licensing/environment allows
```

## 19.2 Model axis

Start tiny and deterministic.

```text
very small transformer
small open model
medium model fitting 16 GB VRAM
quantized variants
```

## 19.3 Kernel/feature axis

```text
GEMM
softmax
RMSNorm/layer norm
attention
RoPE
quant/dequant
sampling
KV cache
custom fused kernels
```

## 19.4 Runtime stress axis

```text
single stream
multiple streams
large allocations
fragmented allocations
repeated model load/unload
graph capture if supported
long-context memory pressure
batching
```

## 19.5 Correctness

Do NOT judge success merely by "program did not crash".

Possible checks:

- exact deterministic tokens under fixed seed where possible
- logits tolerance
- tensor checksum/tolerance
- perplexity delta
- kernel-level numerical comparison

## 19.6 Performance

Potential reference choices:

- native NVIDIA CUDA on roughly comparable GPU class
- native AMD ROCm/HIP path on same AMD GPU
- each provider against best-known AMD path

Each reference answers a different question.

### Question A
How close is CUDArena to native AMD performance?

### Question B
How competitive is the AMD GPU running CUDA software relative to NVIDIA CUDA hardware?

### Question C
How much overhead is introduced by compatibility translation?

These must not be collapsed into one number.

**Reviewer challenge:** Which LLM workloads produce the most information per hour of testing on one RX 9070 XT?

---

# 20. Could LLMs themselves become the main compatibility corpus?

Aggressive version of the idea:

Instead of manually inventing hundreds of synthetic CUDA tests, use modern LLM stacks as **coverage generators**.

Process:

```text
LLM framework/workload
      ↓
observe CUDA calls / modules / libraries exercised
      ↓
break failure into minimal reproducer
      ↓
add reproducer to CUDArena Bench
      ↓
agent mission
```

This creates a workload-driven benchmark corpus.

Advantages:

- prioritizes CUDA features real AI software actually uses
- avoids obscure API-count chasing
- keeps project aligned with high-value workloads
- naturally evolves as AI stacks evolve

Risk:

- benchmark overfits to AI and ignores rendering/scientific/HPC workloads
- fast-changing frameworks reduce reproducibility
- massive software stacks make failures hard to localize

Potential solution:

```text
50% real AI/LLM workload driven
25% core CUDA semantics
25% non-AI real applications
```

This ratio is just a thought experiment, not a proposed truth.

**Reviewer challenge:** Should LLM workloads be the center of the compatibility benchmark, or merely one high-value vertical?

---

# 21. Agent-generated test discovery

Agents need not only repair code. They might also help discover benchmark gaps.

Possible agent jobs:

- reduce a large framework crash into a minimal reproducer
- infer which CUDA semantic is violated
- generate conformance tests from public API documentation
- compare NVIDIA reference behavior against AMD-provider behavior
- fuzz API sequences
- identify missing library functionality
- construct regression tests from historical bugs

This may be more valuable than immediately asking agents to write fixes.

Potential pipeline:

```text
real workload failure
      ↓
reproducer agent
      ↓
minimal deterministic failing test
      ↓
review/verifier
      ↓
repair agent
```

**Reviewer challenge:** Should CUDArena benchmark agents on diagnosis/reproduction separately from repair?

---

# 22. Reference oracle problem

To know whether CUDA semantics are correct, CUDArena may need a trusted reference.

Possible oracle sources:

- run same test on NVIDIA CUDA hardware
- public CUDA documentation
- existing conformance behavior
- known outputs from open CUDA applications

Hard questions:

- What if documentation is ambiguous?
- What if behavior differs across NVIDIA GPU generations?
- What if undefined behavior is accidentally relied upon by real software?
- How do we distinguish implementation quirks from required semantics?

Potential task metadata:

```text
reference_kind:
    documented_semantics
    NVIDIA_observed_behavior
    application_expected_output
    numerical_reference
```

**Reviewer challenge:** Define an oracle hierarchy that avoids accidentally cloning undefined NVIDIA quirks as "CUDA semantics."

---

# 23. Performance benchmark traps

A compatibility layer can be correct but unusably slow.

But performance comparison is easy to abuse.

Potential dimensions:

```text
compile time
startup overhead
kernel time
end-to-end latency
throughput
memory usage
power/energy eventually
```

Need warmup rules.
Need repeated runs.
Need variance reporting.
Need pinned clocks/settings where practical.
Need to separate provider overhead from AMD-vs-NVIDIA hardware differences.

Possible score:

```text
compatibility first
performance second
```

Never allow a faster incorrect result to outrank a correct one.

**Reviewer challenge:** How much performance measurement belongs in M0/M1 versus later?

---

# 24. Provider selection algorithm

Possible early heuristic:

```text
score(provider) =
    required_capability_coverage
    - known_failure_penalty
    - incompatibility_risk
    - startup_overhead
    + performance_prior
```

Longer term CUDArena may maintain a compatibility database:

```text
(workload hash, provider version, GPU, driver)
      → observed outcome
```

Then planner becomes evidence-driven rather than purely declarative.

Potential hierarchy:

1. Known-good exact workload plan.
2. Known-good framework/version plan.
3. Capability-based plan.
4. Conservative single-provider fallback.

**Reviewer challenge:** Is adaptive selection worth the complexity, or would explicit user-selectable profiles be safer?

---

# 25. Failure and fallback semantics

If a chosen provider fails at runtime, can CUDArena safely retry another provider?

Possibilities:

### Safe before execution

```text
compile/provider prepare fails
→ try alternative
```

### Dangerous after state mutation

```text
half the application executed
GPU memory mutated
streams active
provider crashes
→ cannot trivially switch
```

Therefore fallback likely needs phase boundaries or process restart.

Potential policy:

```text
provider failure before stateful execution:
    retry next plan

provider failure after stateful execution:
    terminate/restart workload with alternate plan
```

**Reviewer challenge:** What fallback semantics are realistic without corrupting application state?

---

# 26. Packaging / user experience

Possible final UX:

```text
cudarena install
cudarena doctor
cudarena inspect app
cudarena run app
cudarena explain app
```

Or even more transparent compatibility:

```text
existing CUDA application
    ↓
CUDArena environment / loader shim
    ↓
runs unchanged
```

Potential installation complexity:

- AMD drivers
- ROCm/HSA dependencies
- compiler tools
- provider packages
- architecture-specific support

The project succeeds as a product only if most of this becomes invisible.

**Reviewer challenge:** Is a CLI/platform wrapper the right UX, or is true drop-in ABI compatibility essential?

---

# 27. Legal / IP / clean-room risk

This is a serious project risk.

Agents must not casually copy proprietary or license-incompatible implementation code.

Potential rules:

- no leaked NVIDIA source
- no proprietary SDK source redistribution
- no copying restricted implementation code
- record external repositories/documents accessed by verified agents where feasible
- enforce license compatibility for imported code
- distinguish behavior observation from implementation copying
- careful trademark wording around CUDA

Existing compatibility projects may have very different licenses and redistribution constraints.

Some may be usable only as comparison/reference providers rather than bundled components.

**Reviewer challenge:** Is the "all-in-one distribution" legally plausible if some providers have non-commercial or incompatible licensing?

---

# 28. Security risk

CUDArena Arena will execute untrusted agent-generated code.

A future distributed runner network is therefore dangerous.

Threats:

- malicious build scripts
- filesystem access
- network exfiltration
- persistence
- GPU/driver crashes
- kernel hangs
- resource exhaustion
- supply-chain dependencies

Do NOT casually instruct volunteers to install a privileged runner on their personal workstation.

Potential eventual isolation:

- disposable VM/container where GPU passthrough is practical
- strict filesystem sandbox
- network restrictions
- timeouts
- resource quotas
- reboot/recovery strategy
- signed task/result protocol

**Reviewer challenge:** Is safe volunteer GPU verification practical enough, or should CUDArena rely on centrally managed runners?

---

# 29. Result attestation and cheating

If results are submitted by community runners, how does CUDArena know they are real?

Possible approaches:

- reproducible logs + artifacts
- signed runner identity
- challenge nonce in workload
- rerun suspicious results centrally
- cross-runner consensus
- random audit
- trusted-runner tiers

Do not pretend this is solved by a JSON result file.

**Reviewer challenge:** Propose the simplest trust model that scales beyond one owner-controlled RX 9070 XT.

---

# 30. Benchmark contamination and model memorization

Because agent patches and discussions are public, future models may train on CUDArena solutions.

This creates three benchmark types:

### Engineering benchmark

Public and collaborative. Memorization does not matter much because useful fixes are the goal.

### Model evaluation benchmark

Needs hidden/sealed tasks.

### Longitudinal capability benchmark

Could rotate newly discovered failures continuously.

A dynamic benchmark may be better than a static test set.

**Reviewer challenge:** Should CUDArena even try to become a canonical model leaderboard, or focus on producing useful autonomous engineering work and treat rankings as secondary?

---

# 31. Dynamic benchmark idea

Traditional coding benchmarks freeze.

CUDArena could continuously generate new tasks from compatibility failures.

```text
new provider release / new CUDA app / new framework version
        ↓
new failure discovered
        ↓
new mission
        ↓
agent attempts
```

This makes the benchmark naturally renewable.

Potential benefits:

- harder to memorize
- measures frontier software engineering
- directly produces useful OSS improvements

Potential downside:

- task difficulty changes over time
- leaderboard comparability becomes messy

Possible solution:

- seasonal benchmark snapshots
- fixed hidden set per season
- rolling engineering arena in parallel

**Reviewer challenge:** Design a leaderboard that remains interpretable under a constantly changing task set.

---

# 32. Website information architecture

Potential pages:

## `/`

Benchmark overview, real scores only.

## `/compatibility`

Matrix by workload/provider/GPU.

## `/llm`

LLM workload benchmark.

## `/arena`

Agent leaderboard.

## `/missions`

Open reproducible failures.

## `/runs/<id>`

Full provenance.

## `/providers`

Capability matrix and versions.

## `/hardware`

GPU coverage.

## `/methodology`

Scoring, oracle, benchmark versioning.

## `/docs`

Project architecture/contribution docs.

**Reviewer challenge:** Is this too much product surface before the project has results?

---

# 33. What the homepage should NOT claim initially

Until real data exists, avoid:

```text
"39% CUDA compatible"
"1,284 verified runs"
"Best CUDA replacement"
"Runs CUDA everywhere"
```

Prefer:

```text
CUDArena
An open experiment in adaptive CUDA compatibility and autonomous repair.

Reference target:
RX 9070 XT / gfx1201

Verified benchmark results:
Not yet published
```

Then allow the data to become the marketing.

---

# 34. Existing repo direction

Current repo already contains:

- project README
- architecture notes
- benchmark specification
- agent contribution contract
- roadmap
- RX 9070 XT baseline plan
- site design contract
- static site prototype
- GitHub Pages workflow
- issues for baseline, prior-art inventory, schemas, first agent fix, real website data

Potential problem:

The repo may already be over-documented relative to actual code/results.

**Reviewer challenge:** Which documents should be deleted, merged, or postponed so the project does not become architecture fiction?

---

# 35. Kill criteria

CUDArena should have explicit failure conditions.

Possible M0 kill/pivot conditions:

- no credible open provider can execute even tiny CUDA workloads on RX 9070 XT without major unrelated bring-up
- first useful repair tasks are too large for agents to tackle
- provider architectures are too incompatible to support a meaningful common layer
- legal/licensing constraints prevent practical distribution
- native ROCm ports are consistently so much easier that compatibility provides little value
- LLM workloads overwhelmingly depend on unsupported proprietary CUDA-X pieces that make a community implementation unrealistic
- adaptive routing adds more failures than it solves

Potential pivot options:

### Pivot A
CUDArena becomes only a benchmark across existing CUDA-on-AMD projects.

### Pivot B
CUDArena becomes only an agent-driven compatibility improvement project for one open implementation.

### Pivot C
CUDArena becomes an LLM-focused CUDA compatibility test suite.

### Pivot D
CUDArena focuses on CUDA library/API shims rather than a general runtime.

**Reviewer challenge:** Define much sharper kill criteria and identify the earliest experiment that can falsify the whole premise.

---

# 36. Alternative architectures Claude MUST compare

Do not evaluate only the current favorite architecture.

Please compare at least these:

## Architecture A — monolithic clean-room CUDA reimplementation

One compiler/runtime stack, no providers.

## Architecture B — fork one existing open compatibility project

Put all effort into extending the strongest candidate.

## Architecture C — CUDArena provider platform

Loader + planner + multiple providers + gradual native replacement.

## Architecture D — workload-level launcher only

No shared runtime; inspect workload and choose one provider for the whole process.

## Architecture E — library-first compatibility

Focus on CUDA runtime + CUDA-X shims backed by ROCm; require source recompilation for kernels.

## Architecture F — LLM-only compatibility platform

Do not chase general CUDA. Target only the CUDA subset used by major LLM stacks.

## Architecture G — benchmark/arena only

Never become a runtime. Benchmark existing solutions and have agents contribute upstream.

For each, evaluate:

- feasibility for tiny team/community
- time to first real success
- long-term ceiling
- legal risk
- maintenance burden
- agent-friendliness
- usefulness to users
- performance ceiling
- likelihood of attracting contributors

---

# 37. Potential strongest strategic twist: LLM-first CUDA subset

An aggressive alternative deserves serious consideration:

> Instead of reproducing CUDA broadly, implement the subset required to run modern open LLM workloads extremely well.

This could massively reduce scope.

North star becomes:

```text
"If an open CUDA LLM stack runs on NVIDIA,
CUDArena should make it run on AMD with minimal/no source changes."
```

Then expand outward later.

Potential benefits:

- clear high-value use case
- benchmark naturally defined by real LLM stacks
- fewer obscure APIs
- strong public interest
- agents work on modern kernels/libraries

Potential costs:

- narrower than general CUDA
- still depends on nasty pieces like custom kernels and framework internals
- risk of benchmark chasing rather than principled compatibility

**Reviewer challenge:** Is LLM-first actually the most rational scope for a new project?

---

# 38. Another twist: compatibility graph instead of provider ranking

Rather than thinking of providers as complete stacks, model the ecosystem as a graph of capabilities.

```text
CUDA requirement nodes:
    runtime.memory
    runtime.streams
    ptx.shuffle
    ptx.atomics
    cublas.sgemm
    cudnn.conv
    graph.capture

Providers expose edges:
    Provider A → runtime.memory
    Provider B → ptx.shuffle
    rocBLAS adapter → cublas.sgemm
```

A workload becomes a set/graph of requirements.

CUDArena tries to find a valid execution plan covering that graph subject to interoperability constraints.

This resembles dependency solving more than simple backend ranking.

Potentially elegant.
Potentially absurdly complicated.

**Reviewer challenge:** Is capability-graph planning valuable or an overengineered trap?

---

# 39. AI-generated planner possibility

Long term, an LLM itself might inspect failure logs and propose an execution plan.

But core correctness should not depend on stochastic model judgment.

Possible hybrid:

```text
deterministic capability solver
        +
LLM diagnostics/recommendation layer
```

LLM can suggest:

- likely provider
- missing capability
- probable failure cause
- candidate fallback

But verifier determines truth.

**Reviewer challenge:** Where, if anywhere, should LLM reasoning be inside the runtime rather than only the development loop?

---

# 40. Potential benchmark tasks beyond coding fixes

CUDArena Arena could test agents on multiple competencies:

### Diagnose

Find root cause.

### Reduce

Convert giant application failure into minimal reproducer.

### Implement

Patch compiler/runtime/library.

### Optimize

Improve performance while preserving correctness.

### Route

Construct better provider plan.

### Verify

Design robust regression test.

### Port

Implement capability on new AMD architecture.

This might produce a richer benchmark than "number of issues closed."

**Reviewer challenge:** Should these be separate benchmark tracks?

---

# 41. Reproducibility bundle

A verified run could eventually be a directory like:

```text
runs/2026-xxxx-task-0042/
├── task.json
├── environment.json
├── agent.json
├── prompt.txt
├── patch.diff
├── build.log
├── run.log
├── verifier.json
├── metrics.json
└── README.md
```

This makes results durable and independently inspectable.

But storing huge logs in Git may be bad; artifacts/object storage may eventually be needed.

**Reviewer challenge:** What belongs in git versus release artifacts/object storage/database?

---

# 42. Benchmark versioning

Scores become meaningless if the task set changes invisibly.

Potential scheme:

```text
CUDArena Bench v0.1
CUDArena Bench v0.2
CUDArena LLM Suite 2026-Q4
CUDArena Agent Arena Season 1
```

Each snapshot pins:

- task set
- weighting
- reference versions
- scoring rules

Rolling engineering results remain separate.

**Reviewer challenge:** How should compatibility progress across versions be displayed without misleading users?

---

# 43. CUDA compatibility definition problem

"CUDA compatibility" itself can mean several things:

### Source compatibility

CUDA C++ source can be built.

### PTX compatibility

PTX modules execute.

### Runtime API compatibility

CUDA runtime APIs work.

### Driver API compatibility

Driver API works.

### Binary compatibility

Existing compiled CUDA applications run.

### Library compatibility

cuBLAS/cuFFT/cuDNN/etc. calls work.

### Behavioral compatibility

Corner-case semantics match.

### Application compatibility

Real programs run.

CUDArena should never conflate these.

Potential site badges:

```text
SOURCE
PTX
RUNTIME
BINARY
LIBRARY
APPLICATION
```

**Reviewer challenge:** Which compatibility dimensions matter most for the user goal?

---

# 44. What success could look like after one year

Not a promise—just a concrete target to attack.

Possible credible outcome:

- RX 9070 XT reference runner stable
- several open providers integrated/characterized
- 100+ reproducible tasks
- 10–20 real CUDA applications tested
- a useful LLM workload suite
- dozens of verified autonomous repair runs
- several accepted upstream fixes
- basic provider planner
- one or two native CUDArena adapters
- public provenance-first site

Unrealistic outcome:

- "full CUDA replacement"
- complete binary compatibility
- every CUDA-X library
- all GPUs
- perfect automatic mixing of providers

**Reviewer challenge:** What is a credible one-year target for one motivated founder plus agent assistance/community?

---

# 45. What could make CUDArena uniquely valuable even if Core fails

Even if the adaptive runtime proves too hard, the project could still create durable assets:

1. Best public CUDA-on-AMD compatibility corpus.
2. Real RX 9070 XT compatibility data.
3. Reproducible CUDA semantic tests.
4. LLM workload compatibility suite.
5. Agent software-engineering benchmark grounded in GPU/compiler bugs.
6. Cross-provider capability matrix.
7. Upstream bug fixes.
8. Tooling to reduce CUDA application failures.

This makes the project somewhat option-rich.

But option-rich projects can also become unfocused.

**Reviewer challenge:** Is this healthy optionality or lack of product discipline?

---

# 46. Questions Claude MUST answer

Please explicitly answer all of these, even briefly.

1. Is "Proton for CUDA" technically coherent or misleading?
2. Is a provider architecture the best long-term design?
3. Is workload-level provider selection enough for the first year?
4. Should CUDArena own runtime state eventually?
5. Is cross-provider shared state realistically maintainable?
6. Which existing project should be used first, if any?
7. Should CUDArena fork anything or stay orchestration-first?
8. Is BarraCUDA-like code a good agent target?
9. Is a ZLUDA-like backend a better user-facing base?
10. How should restricted-license implementations be treated?
11. Is gfx1201/RX 9070 XT a sensible first reference target?
12. What is the smallest meaningful M0?
13. What should be deliberately postponed?
14. Should Provider ABI v0 exist before real repair experiments?
15. Is provider capability discovery useful or abstraction theater?
16. What interoperability boundary is safest first?
17. What is the biggest hidden technical blocker?
18. What is the biggest hidden legal blocker?
19. What is the biggest hidden benchmark-validity blocker?
20. Is the agent benchmark scientifically meaningful?
21. How should human intervention classes be defined?
22. How should public task contamination be handled?
23. Should there be sealed tasks?
24. How should dynamic/rolling tasks be scored?
25. Should LLM workloads be central to CUDArena Bench?
26. Is "LLM as a benchmark for CUDA" a strong framing?
27. Which 3 LLM workloads would you pick first?
28. Which correctness metrics should LLM tests use?
29. What performance reference is fairest?
30. How do we avoid API-count vanity metrics?
31. How should compatibility percentages be presented?
32. Should the site be public before real data exists?
33. What should the homepage show at PRE-BASELINE stage?
34. Is the current light benchmark-first design direction appropriate?
35. What site features are unnecessary early?
36. Should agent leaderboard and compatibility leaderboard be separate?
37. How should provenance be exposed?
38. How should task difficulty be estimated?
39. Which agent metrics matter most?
40. Is cost-per-fix a useful metric?
41. Should diagnosis/reduction be benchmarked separately from repair?
42. Should agents be allowed internet access during verified runs?
43. How do we prevent agents from copying prior patches?
44. What clean-room policy is required?
45. Is distributed community GPU verification worth pursuing?
46. What security architecture would it require?
47. What is the earliest experiment that could kill the entire concept?
48. What architecture would you choose if starting from zero today?
49. What parts of the current idea are actively bad and should be deleted?
50. What is the single most important next action?

---

# 47. Reviewer roleplay prompts

Please perform the review from multiple perspectives.

## 47.1 Compiler engineer

Assume you have implemented GPU compiler/runtime systems before. Attack semantic assumptions and interoperability.

## 47.2 Benchmark researcher

Assume you care about scientific validity, contamination, reproducibility, and leaderboard gaming.

## 47.3 Open-source maintainer

Assume you will have to maintain this mess for five years with volunteer contributors.

## 47.4 Security engineer

Assume agent-generated code will be malicious eventually.

## 47.5 Licensing/IP reviewer

Assume every integration choice will be scrutinized.

## 47.6 Product engineer

Assume users just want CUDA software to run and do not care about architectural elegance.

## 47.7 Skeptical investor/funder

Assume most grand compatibility projects die. Ask what proof would change your mind.

## 47.8 Agent benchmark designer

Assume public benchmark contamination is inevitable.

For each persona, give the strongest objection and the strongest reason to continue.

---

# 48. Required adversarial output format

Please structure the response like this:

## A. Executive verdict

Choose one:

```text
BUILD
BUILD BUT NARROW
PIVOT
KILL
```

Give probability estimates if useful.

## B. Top 10 fatal assumptions

Rank by severity.

## C. Architecture review

Compare the seven alternative architectures listed above.

Provide a table:

| Architecture | Feasibility | Time-to-value | Ceiling | Complexity | Recommendation |

## D. LLM-as-CUDA-benchmark review

Attack and redesign the LLM workload strategy.

## E. Agent-benchmark review

Attack contamination, scoring, autonomy claims, and reproducibility.

## F. Website/product review

Review information architecture, visual direction, claims, and data integrity.

## G. Legal/security review

Identify blockers we are underestimating.

## H. Better design

If you have a better architecture, rewrite CUDArena from scratch.

Do not constrain yourself to our provider idea.

## I. M0 experiment

Specify exactly what should be built/run first.

Target completion should be small enough that failure is informative.

## J. Kill criteria

State what result should make us abandon or radically pivot the project.

## K. 30-day plan

Only if the project survives your review.

Keep it brutally prioritized.

---

# 49. Anti-sycophancy instruction

The reviewer should actively resist the following failure mode:

> "This is an exciting and ambitious idea with some challenges..."

That is not useful.

Instead:

- identify contradictions
- identify impossible assumptions
- identify needless complexity
- identify where existing projects already solve the problem better
- identify whether the benchmark is fake-scientific
- identify whether the agent angle adds substance or just branding
- identify whether the LLM-workload focus is rational
- identify the cheapest experiments that can disprove the thesis
- recommend deletion of ideas that are weak

Praise only components that survive adversarial scrutiny.

---

# 50. Current provisional thesis, stated as strongly as possible

CUDArena's strongest form is currently imagined as:

> **An open CUDA-compatible loader/runtime and execution-planning platform for AMD GPUs, backed by pluggable compatibility providers, native ROCm library adapters, and gradually increasing CUDArena-native functionality. CUDArena Bench measures real compatibility using reproducible CUDA applications with a strong LLM-workload track. CUDArena Arena converts observed failures into autonomous coding-agent tasks, uses real AMD hardware as the judge, and feeds verified fixes back into the compatibility stack.**

Potentially:

```text
LLM workload fails
      ↓
CUDArena Bench records failure
      ↓
minimal reproducer generated
      ↓
Arena issues mission
      ↓
agent repairs provider/core
      ↓
RX 9070 XT verifies
      ↓
LLM workload now passes
      ↓
compatibility matrix improves
```

This is the thesis Claude should try to destroy.

---

# 51. Final question to Claude

**If you had the same goal — make CUDA software, especially modern LLM CUDA workloads, run on AMD hardware while using coding agents as the engine of improvement — would you build CUDArena as described here?**

If not:

1. What would you build instead?
2. What would you preserve from CUDArena?
3. What would you delete immediately?
4. What experiment would you run this week?
5. What result would convince you this project has real legs?

Do not optimize for agreement with us. Optimize for finding the architecture most likely to work.
