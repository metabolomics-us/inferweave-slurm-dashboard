# inferweave-dashboard

A Next.js dashboard for the InferWeave gateway — and a **measurable benchmark**
for evaluating agent-produced code.

## What it shows

Against a live gateway (`/v1/stats`, `/v1/models`, and per-worker `/slots`):

- **Placement** — node → GPU → resident model(s). A GPU can host several models,
  so a GPU is not a leaf.
- **Context accounting** — per slot: who holds it, tokens used against the
  window, and busy state derived from `is_processing`.
- **Capacity health** — a model with desired capacity and **zero live workers**
  is an alarm, not a table row.
- **Throughput** — prefill and decode tok/s, as a labelled rolling window.

## Two rules that come from real failures

**Busy is `is_processing`, never `id_task`.** `id_task` retains the last
*completed* task's id indefinitely, so counting it reports every slot busy
forever. That mistake produced a **79% utilisation reading on an entirely idle
fleet**.

**Unknown is not zero.** An unreachable worker returns no data; an idle worker
returns zero. Rendering them the same way is how **7% fleet utilisation went
unnoticed for hours**. See [docs/EVALUATION.md](docs/EVALUATION.md).

## Why this repo exists twice over

It is a tool the gateway needs, and a benchmark with a fixed, real data source —
which makes fabrication detectable and quality comparable between
implementations. The scoring rubric is in
[docs/EVALUATION.md](docs/EVALUATION.md), written so that every criterion is
checkable by a command or a screenshot rather than by opinion.

## Data source

The gateway is at `NEXT_PUBLIC_GATEWAY_URL`. It is **read-only**: the dashboard
observes and must never offer a control that spends GPUs.

## Status

Scaffold and evaluation criteria first; implementation to follow.
