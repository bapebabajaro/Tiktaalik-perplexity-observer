# perplexity/ — CHEF-observer rådata för extern analys

**Syfte:** Denna mapp samlar rådata från alla `chef-observer`-sessioner i ett
publikt, Perplexity-läsbart format. Perplexity (eller annan extern analysator)
kan hämta varje sessionsmapp via GitHub raw-URL utan att behöva navigera
`chef-observer/sessions/` eller ladda hem klonade repos.

**Skiljelinje mot `chef-observer/sessions/<id>/`:**
- `chef-observer/sessions/<id>/` = Workers arbetsmapp (OUR_ANALYSIS.md,
  PERPLEXITY_ANALYSIS.md, PERPLEXITY_REPLY.md + rådata-snapshots).
- `perplexity/<id>/` = rena rådata-kopior + session-README, optimerad för
  Perplexity-access via `raw.githubusercontent.com`.

Se `chef-observer/PROTOCOL.md` för observer-workflowet i sin helhet.

---

## Raw-URL-mönster (för extern hämtning)

Alla filer under `perplexity/<session-id>/` nås direkt via:

```
https://raw.githubusercontent.com/bapebabajaro/Tiktaalik/main/perplexity/<session-id>/<filename>
```

Standard-filer per session:

| Fil | Innehåll |
|---|---|
| `README.md` | Session-metadata (span, duration, exit-reason, länkar till andra filer) |
| `observer-output.txt` | Råa tick-för-tick-rader från `observer-detailed.js` (5 Hz) |
| `ribosome.ndjson` | Alla tool-call-ticks (cellens "ribosom-aktivitet") |
| `autophagy-events.ndjson` | Read-events + sha-checks (grounding-register) |
| `autophagy-registry.ndjson` | Kumulativt raw-data-register (file → sha → bytes) |

Optionella filer (skapas av Worker efter sessionen):

| Fil | När |
|---|---|
| `OUR_ANALYSIS.md` | Worker skriver strukturerad rapport enligt PROTOCOL.md §granskning-metodologi |
| `PERPLEXITY_ANALYSIS.md` | Extern analys-respons från Perplexity |
| `PERPLEXITY_REPLY.md` | Workers svar till Perplexity (iteration fortsätter) |

---

## Format-referens

### ribosome.ndjson

En rad per tool-call. Fält:

```json
{"ts":"<ISO>","event":"tool_tick","tool":"Read","session_id":"<id>","cwd":"<path>"}
```

Tool-set: `Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`, `Skill`, `Agent`,
`WebFetch`, `WebSearch`, `TaskCreate/Update/Get/List`, `ToolSearch`,
`NotebookEdit`.

Ett tick = ett PostToolUse-observabelt anrop. Nucleus-fasens parallella
Skill-loads räknas INTE (de är "ribosome-assembly", ej translation). Intern
resonering räknas INTE.

### autophagy-events.ndjson

En rad per Read-event. Fält:

```json
{"ts":"<ISO>","event":"raw_read","session_id":"<id>","path":"<absolute>","sha256":"<hex>","bytes":<n>}
```

### autophagy-registry.ndjson

Samma schema som events, men filtrerat till **unika** `(path, sha)`-par.
Används för att bedöma om cellen är "grounded" i aktuell rådata.

### observer-output.txt

Tick-för-tick loggrader från `observer-detailed.js`. Format:

```
[HH:MM:SS.mmm] OBSERVER_DETAILED_START session=<id> interval=200ms (5.0Hz)
[HH:MM:SS.mmm] Baseline (pre-observation): ribo=N auto-ev=N auto-reg=N
[HH:MM:SS.mmm] TICK #N tool=<tool>
[HH:MM:SS.mmm] CWD_SHIFT from="..." to="..."
[HH:MM:SS.mmm] RAW path=... sha=... bytes=N
[HH:MM:SS.mmm] WRITE path=<path> first-50-lines:...
[HH:MM:SS.mmm] EVENT <JSON från gate_events.ndjson>
[GATE_STATE] <ts> | AGENT_DEATH | <reason> | <session-label> | actions=N completed=N failed=N
```

---

## Epistemiska regler för Perplexity-analys

Observer är **passiv**. Worker noterar men värderar inte task-korrekthet (det
är CHEF:s domän). När Perplexity analyserar rådata gäller samma regler:

- Alla tolkningar ska vara avläsbara från tick-data + NDJSON.
- Antaganden markeras explicit `[INF]`.
- Konfidens per substantiellt påstående: `[KONFIDENS: HIGH|MEDIUM|LOW]`.
- N-värde för generaliseringar ska anges (N=1 efter en session, N=2 efter två).
- Skillnad "vi vet" vs "vi tror" ska hållas rent.

---

## Sessions-index

| Session-ID | Datum | Duration | Death-reason | Actions |
|---|---|---|---|---|
| `5d47bce3-9680-4224-9c68-37392777cf8f` | 2026-04-20 | ~62 min | task_complete | 150 |

Uppdateras manuellt när nya CHEF-observer-sessioner arkiveras.
