# `loop-research` — LOOP(data) { POSITIVE; NEGATIVE } RETURN

`type: system` · roles: **Conjecturer**, **Critic**, **Recorder** · subordinate to `docs/follow-research-contract.md`.

This is one research night, from the Steward's charter to the holdout. It composes the two skills and the Recorder's notebook into a single append-only trace.

## Specification

```json { type: notebook.cell }
{
  "id": "systems/loop-research.md",
  "type": "system",
  "role": "run a charter to RETURN: base, then conjecture and refutation until the budget, then the holdout once",
  "reads": ["research.json", "charter", "skills/conjecture-hypothesis.md", "skills/refute-experiment.md"],
  "writes": ["trace", "results.tsv", "notebook"],
  "must": ["start from a Steward charter", "replicate the base first", "one conjecture under test at a time", "a note after every probe verdict", "stop at max_tested or Steward stop", "compare on the holdout only at RETURN if the base changed", "return P2"],
  "check": ["R1-R13"]
}
```

## Graph

- ← [../docs/read-repository.md](../docs/read-repository.md)
- ← [../skills/conjecture-hypothesis.md](../skills/conjecture-hypothesis.md)
- ← [../skills/refute-experiment.md](../skills/refute-experiment.md)

## Procedure

```text { type: notebook.cell }
charter := Steward                                   # P1
setup:   prepare the pinned checkout and cache (docs/pin-sources.md)
         create popper/<tag> at source.commit; append the charter as seq 1
base:    Critic runs the base three times → b      # seeds from the root after the charter
LOOP(data) {
  POSITIVE: Conjecturer emits one conjecture on the base      # TT
  NEGATIVE: Critic runs it until refuted, inconclusive, or corroborated   # EE
            corroborated → advance; b := h
  Recorder emits one note citing the conjecture, results, verdict, advance
} until verdicts ≥ charter.max_tested or Steward emits stop
RETURN:  if the base changed, Critic compares first base vs last base on the holdout
         Recorder emits return { tested, refuted, corroborated, inconclusive, edge, holdout, base, P2 }
```

- **Unattended between charter and RETURN.** Like the source, the loop does not stop to ask whether to continue. Unlike the source, it does stop, at the charter's budget, and the Steward may `stop` or `amend` at any time. An `amend` applies only from the next conjecture on, never to one already registered (`ad_hoc`).
- **One arm at a time.** Arms are tested in trace order, so every verdict reads the base that was current when the conjecture was registered.
- **The holdout runs cite what ended the probe loop.** Their `arm` is the sequence number of the Steward's `stop` or the last probe verdict. Their seeds are derived from the root just after that event.
- **P2.** The return names one new problem the night opened, as a question. It does not say what is true.

## Worked trace

[`traces/nov25.jsonl`](../traces/nov25.jsonl) is an **illustrative** night of three tests. Its scores and arm commit IDs are synthetic, not measured results. The trace replays from the charter to the root pinned in `research.json`. To check the example diffs against the pinned testbed, use `git apply --check --unidiff-zero` on the corresponding base content.

| seq | world | kind | What happens |
|---|---|---|---|
| 1 | human | charter | `delta_min = 0.001`, `max_tested = 3`, `holdout_delta = 0.001`, tag `nov25` |
| 2–7 | data | run, result | base `228791f` × 3, seeds 618655339, 1770127804, 4035022987: `b = [0.9981, 0.9976, 0.9984]`, `min(b) = 0.9976` |
| 8 | theory | conjecture | A: `MATRIX_LR` 0.04 → 0.05, `improves`, `delta = 0.002` |
| 9–10 | data | run, result | seed 1878499161: 0.9979 ≥ 0.9976 |
| 11 | data | verdict | **refuted** after one run, 1 bit; the base stays |
| 12 | frame | note | a learning rate of 0.05 is ruled out |
| 13 | theory | conjecture | B: `WARMDOWN_RATIO` 0.5 → 0.6, `improves`, `delta = 0.001`, cites 11 |
| 14–19 | data | run, result | `h = [0.9958, 0.9962, 0.9955]`, all below 0.9976; mean gain 0.0022 ≥ 0.001 |
| 20 | data | verdict | **corroborated**, 1 bit |
| 21 | data | advance | base := arm B; `b := h`, `min(b) = 0.9955` |
| 22 | frame | note | B survived one test; it may still fall |
| 23 | theory | conjecture | C: `DEPTH` 8 → 10, `improves`, `delta = 0.003` |
| 24–25 | data | run, result | crash: no `val_bpb` line |
| 26 | data | verdict | **inconclusive**, 0 bits; three tests used |
| 27 | frame | note | nothing is ruled in or out about depth |
| 28–39 | data | run, result | holdout: base `228791f` → `[1.0012, 1.0017, 1.0009]`; base B → `[0.9991, 0.9994, 0.9988]` |
| 40 | data | verdict | holdout **corroborated**: all three below 1.0009, mean gain 0.0022 ≥ 0.001 |
| 41 | frame | return | tested 3, refuted 1, corroborated 1, inconclusive 1, edge 2 bits, P2 below |

The note at seq 22, as written by the Recorder:

> You see the second conjecture lengthen the warmdown to 0.6 and risk refutation by any run at or above 0.9976 [e13]. All three runs come in below it, at 0.9958, 0.9962 and 0.9955 [e15] [e17] [e19], a mean gain of 0.0022 against the 0.001 it predicts. It is corroborated [e20] and becomes the base [e21]. It survives this test; it may still fall.

P2: *Does the warmdown gain survive at ten layers once the batch is halved to fit memory?*

### `results.tsv` projection (R13)

```text { type: notebook.cell }
commit	val_bpb	memory_gb	status	description
1c55ce2	0.997900	44.0	discard	A higher Muon learning rate trains faster within five minutes.
a3460f8	0.995800	44.0	discard	A longer warmdown lets the model settle lower before the budget ends.
a3460f8	0.996200	44.0	discard	A longer warmdown lets the model settle lower before the budget ends.
a3460f8	0.995500	44.0	keep	A longer warmdown lets the model settle lower before the budget ends.
e849727	0.000000	0.0	crash	Ten layers fit more capacity into the same five minutes.
```

The base and holdout runs are in the trace, not in the TSV.

## Replay

Run this cell with Node.js from the repository root. It checks the object graph checksum, trace digest and root, and replays the seeds and the four `improves` verdicts in the example. It does not train a model.

```js { type: notebook.cell }
const assert = require("node:assert/strict");
const { readFileSync: read } = require("node:fs");
const { createHash } = require("node:crypto");
const sha = value => createHash("sha256").update(value).digest("hex");
const jcs = value => {
  if (Array.isArray(value)) return "[" + value.map(jcs).join(",") + "]";
  if (value !== null && typeof value === "object") {
    return "{" + Object.keys(value).sort()
      .map(key => JSON.stringify(key) + ":" + jcs(value[key])).join(",") + "}";
  }
  return JSON.stringify(value);
};

const graph = read("ooc/popper.og", "utf8");
const checksum = /^checksum: sha256:([a-f0-9]{64})\n/m;
assert.equal(sha(graph.replace(checksum, "")), graph.match(checksum)[1]);

const config = JSON.parse(read("research.json"));
const path = "traces/nov25.jsonl";
const pin = config.traces[path], bytes = read(path);
assert.equal("sha256:" + sha(bytes), pin.digest);
const events = [null, ...bytes.toString("utf8").trimEnd().split("\n").map(JSON.parse)];
assert.equal(events.length - 1, pin.events);
const roots = [sha("popper/1")];
assert.equal(roots[0], config.loop.root_empty);

const mean = values => values.reduce((sum, n) => sum + n, 0) / values.length;
const charter = events[1].body;
const scores = rows => rows.filter(r => r.body.status === "ok").map(r => r.body.val_bpb);
let baseArm = 1;
for (const [i, event] of events.entries()) {
  if (i === 0) continue;
  const { seq, kind, body, cites } = event;
  assert.equal(seq, i);
  assert(cites.every(id => id > 0 && id < seq));
  roots.push(sha(roots[i - 1] + sha(jcs(event))));
  if (kind === "run") {
    assert(cites.includes(body.arm));
    assert.equal(body.seed, parseInt(sha(roots[body.arm] + "seed/" + body.i).slice(0, 8), 16));
  }
  if (kind === "advance") baseArm = events[cites[0]].body.conjecture;
  if (kind !== "verdict") continue;
  const results = cites.map(id => events[id]).filter(e => e.kind === "result");
  const runOf = r => events[r.body.run].body;
  const inBase = r => body.holdout ? runOf(r).i <= 3 : runOf(r).arm === baseArm;
  const base = results.filter(inBase);
  assert.deepEqual(base.map(r => runOf(r).i), [1, 2, 3]);
  const b = scores(base);
  assert.equal(b.length, 3);
  const arm = results.filter(r => !inBase(r));
  if (!body.holdout) assert(arm.every(r => runOf(r).arm === body.conjecture));
  const h = scores(arm);
  assert.deepEqual(body.b, b);
  assert.deepEqual(body.h, h);
  const delta = body.holdout ? charter.holdout_delta : events[body.conjecture].body.prediction.delta;
  const firstMiss = arm.findIndex(r => r.body.status !== "ok" || r.body.val_bpb >= Math.min(...b));
  assert(firstMiss === -1 ? arm.length === 3 : firstMiss === arm.length - 1);
  const verdict = arm.some(r => r.body.status !== "ok") ? "inconclusive"
    : firstMiss !== -1 || mean(h) - mean(b) > -delta ? "refuted" : "corroborated";
  assert.equal(body.verdict, verdict);
  assert.equal(body.bits, verdict === "inconclusive" ? 0 : 1);
}
assert.equal(roots.at(-1), pin.root);
console.log(`Replayed ${pin.events} events: ${pin.root}`);
```

## Falsification

| Challenge | Failure signal |
|---|---|
| Swap seq 8 and seq 9 | `post_hoc`, and every later root changes |
| Run a fourth replicate of A | `self_verdict` |
| Count the crash at seq 25 as a refutation | `self_verdict` (R7) |
| Read the holdout after seq 21 | `peek` |
| Return "the warmdown schedule is better" | `verification` |

## Licence

MIT, see `LICENSE`.
