# `refute-experiment` — NEGATIVE: try to break a registered conjecture

`type: skill` · role: **Critic** · subordinate to `docs/follow-research-contract.md`.

The Critic is the loop's negative feedback. It runs a registered conjecture against the base with seeds derived from the trace and applies a fixed rule. It is Type I: a fixed mapping from percept to verdict, with no memory of earlier conjectures and no access to the rationale.

## Specification

```json { type: notebook.cell }
{
  "id": "skills/refute-experiment.md",
  "type": "skill",
  "role": "run the base and each arm with derived seeds, apply research.json critic.test, emit verdict and maybe advance",
  "reads": ["research.json testbed", "research.json critic", "conjecture prediction and diff", "base results"],
  "writes": ["run", "result", "verdict", "advance"],
  "must": ["check pins before running", "derive seeds from the root", "stop at the first refuting replicate", "never read the rationale", "never rerun a refuted arm", "never touch the holdout before RETURN"],
  "check": ["R1", "R5", "R6", "R7", "R8", "R10"]
}
```

## Graph

- ← [conjecture-hypothesis.md](conjecture-hypothesis.md)
- ← [../docs/pin-sources.md](../docs/pin-sources.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## Percept

```text { type: notebook.cell }
percept = { diff, kind, delta, b: [Number; 3], h: [Number] }   # h grows one replicate at a time
```

Nothing else. Not the claim, not the notebook, not how many conjectures came before (`stateful`).

## Running one replicate

1. Check the frozen file and probe digests (`drift`), and use only the staged probe cache described in `docs/pin-sources.md` (`peek` if it contains the holdout).
2. Commit the arm diff once on `popper/<tag>`. In a scratch copy of that commit, set the two seed calls in `train.py` (`torch.manual_seed` and `torch.cuda.manual_seed`, lines 458–459 at the pin; find by pattern) to `seed(r, i)` from contract §3. For the initial base, use a scratch copy of the source commit. The registration root `r` is the root after the conjecture or, for the initial base, the charter. The seed edit is not committed and the `run` records the unseeded commit.
3. Append a `run` event, then run `uv run --locked train.py > run.log 2>&1` in the scratch copy with the staged probe cache and a `timeout_s` kill.
4. Read `val_bpb` and `peak_vram_mb` from `run.log`. Append a `result` event: `status: "ok"`, or `"crash"` if there is no `val_bpb:` line or its value is not a finite number (`nan`, `inf`), or `"timeout"`. `memory_gb = peak_vram_mb / 1024`, rounded to one decimal, as in the source.

## The base

At the charter and after every `advance` the Critic needs three base results `b`. After an `advance`, `b` is the corroborated arm's `h`: the code has not changed, and the seeds were already the Critic's. At the charter it runs the base three times. If an initial base replicate crashes or times out, the Recorder returns with no tests and no holdout, citing the failed result; the loop cannot start on a base it cannot run.

## The rule

`research.json` `critic.test`, applied as replicates arrive:

```text { type: notebook.cell }
improves:
  for i in 1..3:
    run h_i
    if h_i is crash or timeout         → inconclusive, stop
    if h_i >= min(b)                   → refuted, stop          # one counter-instance (P3)
  if mean(h) - mean(b) > -delta        → refuted
  else                                 → corroborated           # chance under no effect: 1/20

simplifies:
  if added(diff) >= deleted(diff)      → refuted, no run        # it is not simpler
  run h_1..h_3                         # any crash or timeout → inconclusive
  if mean(h) - mean(b) > delta         → refuted
  else                                 → corroborated
```

For `improves`, ties at `min(b)` refute. For `simplifies`, the mean-loss limit is the registered `delta`, equal to the charter's `delta_min` at registration. An `inconclusive` verdict carries 0 bits; `refuted` and `corroborated` carry 1 (contract §2).

## Emitting

1. Append a `verdict` event citing the conjecture, all three base results and every arm result used. Cite the initial base's results until the first advance; thereafter cite the arm results of the corroborated verdict that set the current base.
2. If `corroborated`, append an `advance` event from the base to the arm commit, citing the verdict. The arm becomes the base and `h` becomes `b`.
3. If `refuted` or `inconclusive`, the base does not move. Reset the branch to the base; the diff stays in the conjecture event.
4. Append one row per arm run to `results.tsv`: `commit`, `val_bpb` (`0.000000` on a crash), `memory_gb`, `status` from `research.json` `loop.status_map` (only the last replicate of a corroborated arm is `keep`; earlier replicates of every arm are `discard`), and the claim as `description`.

## The holdout, at RETURN

If the first and last base are the same commit, do not open the holdout. Otherwise use the harness in `docs/pin-sources.md` for one comparison: run the first base three times with indices 1–3, then test the last base with indices 4–6 using the `improves` rule and `delta = charter.holdout_delta`. Use the root and sequence number of the `stop` event if it triggered RETURN, otherwise the last probe verdict. Mark the runs and verdict with `body.holdout: true`. The verdict cites the conjecture that set the final base, its advance, and the holdout results. A failed replicate of the first base makes the comparison inconclusive; the holdout never causes an advance.

## Refusals

`unpinned`, `drift`, `unregistered`, `stateful`, `self_verdict`, `closed_loop`, `peek`.

## Falsification

| Challenge | Failure signal |
|---|---|
| Run replicate 3 after replicate 1 refuted "just to be sure" | `self_verdict` |
| Use seeds 1, 2, 3 | R6 fails |
| Read the claim and give a promising conjecture a fourth replicate | `stateful` |
| Advance on a mean gain with one replicate above `min(b)` | `self_verdict` |

## Licence

MIT, see `LICENSE`.
