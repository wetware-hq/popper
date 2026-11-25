# `loop-research` — LOOP(data) { POSITIVE; NEGATIVE } RETURN

`type: system` · `spec: agent.system` · roles: **Conjecturer**, **Critic**, **Recorder** · subordinate to `docs/follow-agent-contract.md`.

This is one research night, from the Steward's charter to the holdout. It composes the two skills and the Recorder's notebook into a single append-only trace.

## Specification

```json { type: notebook.cell }
{
  "id": "systems/loop-research.md",
  "type": "system",
  "role": "run a charter to RETURN: base, then conjecture and refutation until the budget, then the holdout once",
  "reads": ["research.json", "charter", "skills/conjecture-hypothesis.md", "skills/refute-experiment.md"],
  "writes": ["trace", "results.tsv", "notebook"],
  "must": ["start from a Steward charter", "replicate the base first", "one conjecture under test at a time", "a note after every verdict", "stop at max_tested or Steward stop", "read the holdout once", "return P2"],
  "check": ["R1-R13"]
}
```

## Graph

- ← [../docs/read-repository.md](../docs/read-repository.md)
- ← [../skills/conjecture-hypothesis.md](../skills/conjecture-hypothesis.md)
- ← [../skills/refute-experiment.md](../skills/refute-experiment.md)

## Procedure

```text { type: notebook.cell }
charter := Steward                                   # P1; an agent may not write it (self_charter)
setup:   clone autoresearch at the pin; check every digest (R1); git checkout -b popper/<tag>
base:    Critic runs the base three times → b      # seeds from the root after the charter
LOOP(data) {
  POSITIVE: Conjecturer emits one conjecture on the base      # TT
  NEGATIVE: Critic runs it until refuted, inconclusive, or corroborated   # EE
            corroborated → advance; b := h
  Recorder emits one note citing the conjecture, results, verdict, advance
} until verdicts = charter.max_tested or Steward emits stop
RETURN:  Critic reads the holdout once, first base vs last base
         Recorder emits return { tested, refuted, corroborated, inconclusive, edge, holdout, base, P2 }
```

- **Unattended between charter and RETURN.** Like the source, the loop does not stop to ask whether to continue. Unlike the source, it does stop, at the charter's budget, and the Steward may `stop` or `amend` at any time. An `amend` applies only from the next conjecture on, never to one already registered (`ad_hoc`).
- **One arm at a time.** Arms are tested in trace order, so every verdict reads the base that was current when the conjecture was registered.
- **The holdout runs cite the last verdict.** They test no new conjecture; they re-test the night's survivors, so their `arm` is the last verdict's `seq`, and their seeds are `seed(r, 1..6)` from the root just after it.
- **P2.** The return names one new problem the night opened, as a question. It does not say what is true.

## Worked trace

[`traces/nov25.jsonl`](../traces/nov25.jsonl) is an **illustrative** night of three tests. Its structure, diffs (against the pinned `train.py`), seeds and roots are exact and replay; its `val_bpb` values and arm commits were made up to show the rule and were not measured. Root after 41 events: `7f1e170d13b44aac653b929151bfa4952b3fd610194c04e93b0a3097247b368d`.

| seq | world | kind | What happens |
|---|---|---|---|
| 1 | human | charter | `delta_min = 0.001`, `max_tested = 3`, `holdout_delta = 0.001`, tag `nov25` |
| 2–7 | data | run, result | base `228791f` × 3, seeds 618655339, 1770127804, 4035022987: `b = [0.9981, 0.9976, 0.9984]`, `min(b) = 0.9976` |
| 8 | theory | conjecture | A: `MATRIX_LR` 0.04 → 0.05, `improves`, `delta = 0.002` |
| 9–10 | data | run, result | seed 307932280: 0.9979 ≥ 0.9976 |
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

> The second conjecture lengthened the warmdown to 0.6 and risked refutation by any run at or above 0.9976 [e13]. All three runs came in below it, at 0.9958, 0.9962 and 0.9955 [e15, e17, e19], a mean gain of 0.0022 against the 0.001 it predicted, so it is corroborated [e20] and becomes the base [e21]. It has survived one test; it may still fall.

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

The source's own tools (`analysis.ipynb`) read this file unchanged. The base and holdout runs are in the trace, not in the TSV.

## Replay

```python { type: notebook.cell }
import hashlib, json
h = lambda s: hashlib.sha256(s.encode()).hexdigest()
jcs = lambda e: json.dumps(e, sort_keys=True, separators=(",", ":"), ensure_ascii=False)
root = h("popper/1")
for line in open("traces/nov25.jsonl"):
    root = h(root + h(jcs(json.loads(line))))
assert root == json.load(open("research.json"))["traces"]["traces/nov25.jsonl"]["root"]
```

`json.dumps` with sorted keys equals RFC 8785 JCS for the strings, integers and short decimals used here. A full audit also recomputes every seed from §3 and every verdict from the rule (R6, R7).

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
