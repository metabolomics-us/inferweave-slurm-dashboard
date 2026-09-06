# Evaluation criteria

This project has two purposes. It is a working dashboard for the InferWeave
gateway, and it is a **benchmark for judging agent-produced code** on something
harder than "the tests pass".

Every criterion below is checkable by a person or a script. Nothing here is a
matter of taste, because a rubric that rests on taste cannot compare two
implementations.

## Why a dashboard makes a good benchmark

A gateway dashboard is unusually well suited to this:

- it has a **real data source** with a fixed shape, so fabrication is detectable
- it has **states that are easy to get wrong** and that most implementations get
  wrong in the same way
- it is **visually inspectable**, so a reviewer can see quality without reading
  every line
- it is small enough to implement in one session and large enough to have
  architecture

## The states that separate good from adequate

This is the discriminating test, and it comes from a real failure.

A gateway worker that cannot be reached returns **no data**. A worker that is
reachable and doing nothing returns **zero**. These are different facts:

| state | meaning | correct rendering |
|---|---|---|
| unreachable | we do not know | `unknown`, visually distinct |
| reachable, idle | we know: nothing running | `0` |
| stale | last report N minutes ago | value plus its age |

Rendering the first two identically is how **7% fleet utilisation went unnoticed
for hours** on the real system: an instantaneous sample returned `0`, and `0`
looked like "idle" rather than "no data".

Almost every implementation renders loading and success correctly. Far fewer
distinguish unknown from zero. That single distinction is the strongest quality
signal in this rubric.

## Scored dimensions

### 1. Correctness against the real API (weight: high)

- types match the gateway's actual `/v1/stats` and `/v1/models` payloads
- no invented fields; no placeholder data shipped in a non-demo path
- `unknown` / `0` / stale rendered distinctly, per the table above
- numbers are formatted, not raw floats: `p50_ms: 11522.715` renders as `11.5 s`

**Measurable:** point it at a live gateway and at a gateway returning partial or
malformed payloads. It must not crash, and must not display a wrong number
confidently.

### 2. Architecture (weight: high)

- data fetching separated from presentation; components do not fetch
- one typed API layer; the payload shape is defined in exactly one place
- no business logic in JSX
- state that belongs to the server is not duplicated into client state

**Measurable:** count the files that import `fetch` or the API client. More than
the API layer means the boundary leaked. Count the places the response type is
declared. More than one means the contract is duplicated.

### 3. Visual quality (weight: medium)

Judged from screenshots at fixed viewports (1440x900 and 390x844):

- no horizontal overflow at either width
- text contrast meets WCAG AA
- the zero-capacity alarm is **unmissable** — this is the dashboard's most
  important state and must not be a row in a table
- dense numeric data is aligned and scannable

### 4. Build discipline (weight: medium)

- `next build` succeeds with no warnings
- no console errors or warnings at runtime
- no unused dependencies; no dependency added for something the platform provides
- typecheck and lint clean

**Measurable:** all four are commands with exit codes.

### 5. Honest failure (weight: high)

- a request that fails renders an error state naming what failed
- a partial payload renders what it has and marks the rest unknown
- nothing is ever fabricated to fill a gap

**Measurable:** run it against a gateway that returns 500, a truncated JSON body,
and an empty worker list. Three distinct, correct renderings.

## What is deliberately NOT scored

- lines of code, file count, or comment density
- framework or styling choices, provided the criteria above are met
- test count — **coverage of the states above matters, quantity does not**

Measured on this project's sibling work, review length and citation count varied
**2.2x between identical runs of the same model on the same input**. Any metric
that noisy cannot distinguish implementations, and scoring on it rewards
verbosity over correctness.
