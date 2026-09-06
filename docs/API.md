# Gateway API contract

Captured from a **live gateway**, not from documentation. Any implementation
must match these shapes; a field not present here does not exist.

## `GET /v1/stats`

```json
{
  "models": [
    {
      "model": "Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf",
      "requests": 2,
      "errors": 0,
      "served_503": 0,
      "inflight": 0,
      "p50_ms": "11522.715"
    }
  ],
  "uptime_seconds": 18.191291306,
  "total_requests": 2,
  "total_errors": 0,
  "total_served_503": 0,
  "dropped": 0,
  "events": [
    { "kind": "worker_registered",
      "model": "Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf",
      "at": "2026-09-06T12:30:17.628797389-07:00" }
  ]
}
```

Note `p50_ms` is a **string**, not a number, and carries sub-millisecond
precision that must not be displayed raw. `dropped` is the telemetry drop
counter: a non-zero value means metrics were lost and the other numbers
understate reality — it must be surfaced, not hidden.

## `GET /v1/models`

```json
{ "object": "list",
  "data": [ { "id": "...", "object": "model", "owned_by": "",
              "created": 1788723017,
              "x_context_window": 65536,
              "x_state": "warm" } ] }
```

`x_state` is `hot` | `warm` | `cold`. `cold` means the site knows how to bring
the model up but is not serving it — so the catalog is deliberately wider than
the set of live workers.

## Worker `GET /slots` (llama.cpp, direct)

```json
[ { "id": 0, "id_task": 66429, "is_processing": false,
    "n_ctx": 65536, "n_prompt_tokens": 0, ... } ]
```

**Busy is `is_processing`.** `id_task` persists after completion and is not a
liveness signal.

## Absent, and must be treated as such

There is no per-GPU VRAM endpoint and no session/username field yet. An
implementation must render these as **unknown**, never as zero or a placeholder,
until the gateway provides them (tracked in inferweave-gateway#34 and #22).
