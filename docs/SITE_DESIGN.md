# CUDArena site design contract

This document is the visual/UX target for the public CUDArena site.

## Product posture

CUDArena should look like a **serious public benchmark and research product**, not a gaming dashboard, crypto app, GPU marketing page, or cyberpunk landing page.

Reference qualities to preserve:

- benchmark-first and data-first
- light neutral theme by default
- restrained use of color
- excellent table readability
- clear provenance and verification state
- professional, technical, credible
- compact information density without feeling cramped

## Brand

- Wordmark: `CUDArena`
- `CUDA` may use the project teal; `rena` stays near-black.
- Primary accent: muted teal/green.
- AMD-related emphasis may use restrained warm/red accents, never large neon glows.
- PASS = muted green; FAIL = muted red; REGRESSION/PARTIAL = amber.

## Avoid

- giant GPU renders
- neon/cyberpunk hero scenes
- glowing borders everywhere
- excessive gradients
- fake sci-fi diagrams
- game-like points/trophies dominating the product
- marketing copy taking more space than benchmark data
- invented measured results presented as real

## Homepage hierarchy

1. Compact navigation.
2. One-sentence benchmark mission.
3. Community/result summary.
4. Four small KPI cards.
5. **Large benchmark/agent leaderboard as the primary surface.**
6. Recent verified runs.
7. Benchmark categories / compatibility breakdown.
8. Runner coverage and methodology links.

The leaderboard should visually dominate the page more than marketing content.

## Leaderboard table

Expected columns may include:

- rank
- model / agent
- organization
- run type / harness
- compatibility score
- passed / total
- trend
- Runtime API
- PTX/compiler
- cuBLAS
- PyTorch / real workloads
- average performance ratio

Filtering should eventually support:

- organization
- model/harness
- benchmark category
- AMD GPU architecture/model
- verified-only

## Data integrity UX

Every score shown publicly should be traceable to evidence. Real-result pages should expose:

- benchmark/task ID
- source revision
- target GPU
- environment versions
- build/run command
- agent/model/harness
- result status
- logs / artifact references
- patch/commit when relevant

Placeholder/demo values must be explicitly marked as such until the RX 9070 XT baseline exists.

## Current prototype

The static prototype is under `site/` and is intentionally dependency-free so it can deploy directly with GitHub Pages.

It is a visual/product prototype, **not yet a source of measured benchmark truth**.
