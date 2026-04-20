# Tiktaalik-perplexity-observer

Public, read-only mirror of CHEF-observer session logs from the private
`bapebabajaro/Tiktaalik` repository. Purpose: let Perplexity (and other
external analysts) fetch raw NDJSON + observer-output without needing
access to the private organism source.

**This repo is mirror-only. Do not edit directly.** All content is
synced from `Tiktaalik/perplexity/` in the source repo.

---

## Raw-URL pattern (for Perplexity)

Every file is directly fetchable via:

```
https://raw.githubusercontent.com/bapebabajaro/Tiktaalik-perplexity-observer/main/<session-id>/<filename>
```

Example:
- https://raw.githubusercontent.com/bapebabajaro/Tiktaalik-perplexity-observer/main/5d47bce3-9680-4224-9c68-37392777cf8f/observer-output.txt
- https://raw.githubusercontent.com/bapebabajaro/Tiktaalik-perplexity-observer/main/5d47bce3-9680-4224-9c68-37392777cf8f/ribosome.ndjson

---

## Standard files per session

| File | Contents |
|---|---|
| `README.md` | Session metadata (span, duration, exit-reason, tool-mix) |
| `observer-output.txt` | Tick-by-tick lines from `observer-detailed.js` (5 Hz) |
| `ribosome.ndjson` | All tool-call ticks (the cell's "ribosome activity") |
| `autophagy-events.ndjson` | Read events + sha/bytes (grounding register) |
| `autophagy-registry.ndjson` | Cumulative raw-data register (file → sha → bytes) |

Optional (added by Worker after the session):

| File | When |
|---|---|
| `OUR_ANALYSIS.md` | Worker's structured report per PROTOCOL.md |
| `PERPLEXITY_ANALYSIS.md` | External analysis response |
| `PERPLEXITY_REPLY.md` | Worker's reply back to Perplexity |

---

## Schema reference

### ribosome.ndjson

One line per tool-call. Fields:

```json
{"ts":"<ISO>","event":"tool_tick","tool":"Read","session_id":"<id>","cwd":"<path>"}
```

Tool set: `Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`, `Skill`,
`Agent`, `WebFetch`, `WebSearch`, `TaskCreate/Update/Get/List`,
`ToolSearch`, `NotebookEdit`.

One tick = one PostToolUse-observable call. Nucleus-phase parallel
Skill-loads are NOT counted (that's "ribosome assembly", not
translation). Internal reasoning is NOT counted.

### autophagy-events.ndjson

One line per Read event. Fields:

```json
{"ts":"<ISO>","event":"raw_read","session_id":"<id>","path":"<absolute>","sha256":"<hex>","bytes":<n>}
```

### autophagy-registry.ndjson

Same schema as events, filtered to **unique** `(path, sha)` pairs.
Used to judge whether the cell is "grounded" in current raw data.

### observer-output.txt

Tick-by-tick lines from `observer-detailed.js`. Format:

```
[HH:MM:SS.mmm] OBSERVER_DETAILED_START session=<id> interval=200ms (5.0Hz)
[HH:MM:SS.mmm] Baseline (pre-observation): ribo=N auto-ev=N auto-reg=N
[HH:MM:SS.mmm] TICK #N tool=<tool>
[HH:MM:SS.mmm] CWD_SHIFT from="..." to="..."
[HH:MM:SS.mmm] RAW path=... sha=... bytes=N
[HH:MM:SS.mmm] WRITE path=<path> first-50-lines:...
[HH:MM:SS.mmm] EVENT <JSON from gate_events.ndjson>
[GATE_STATE] <ts> | AGENT_DEATH | <reason> | <session-label> | actions=N completed=N failed=N
```

---

## Epistemic rules for analysis

The observer is **passive**. Worker does NOT grade task correctness
(that is CHEF's domain). When analyzing raw data, apply the same rules:

- All interpretations must be derivable from tick-data + NDJSON.
- Assumptions explicitly marked `[INF]`.
- Confidence per substantive claim: `[KONFIDENS: HIGH|MEDIUM|LOW]`.
- N-value for generalizations (N=1 after one session, N=2 after two).
- Keep "we know" vs "we think" distinct.

---

## Session index

| Session ID | Date | Duration | Death reason | Actions |
|---|---|---|---|---|
| `5d47bce3-9680-4224-9c68-37392777cf8f` | 2026-04-20 | ~63 min | task_complete | 150 |

Updated on each mirror sync.

---

## Sync model

Source: private `bapebabajaro/Tiktaalik`, path `perplexity/`.
Target: this repo (root = source's `perplexity/` contents).

Sync is **manual** from Worker after each CHEF-observer session
completes. See `chef-observer/PROTOCOL.md` in the private source for
the full observer workflow.

---

## License / reuse

These logs are behavioral telemetry from experimental biological-AI
agent sessions. Share freely for research and analysis. No claims
about fitness for any particular purpose.
