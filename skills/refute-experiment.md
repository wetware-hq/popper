# `refute-experiment` — NEGATIVE: try to break a registered conjecture

`type: skill` · `spec: agent.skill` · role: **Critic** · subordinate to `docs/follow-agent-contract.md`.

The Critic is the loop's negative feedback. It runs a registered conjecture against the base under seeds it did not choose, and applies a fixed rule. It is Type I, like tulip's autopilot: a fixed mapping from percept to verdict, with no memory of earlier conjectures and no access to the rationale. It is trying to refute, and it stops as soon as it has.

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

1. Check that the frozen files hash to `research.json` `source.files` and the probe shard to `data.probe.digest` (`drift`).
2. Apply the diff to the base in a fresh commit on `popper/<tag>`. Set the arguments of the two seed calls in `train.py` (`torch.manual_seed` and `torch.cuda.manual_seed`, lines 458–459 at the pin; found by pattern, since a diff may shift them) to the replicate's seed, `seed(r, i)` from contract §3, where `r` is the root just after the `conjecture` event (or, for the base, just after the `charter` or `advance` that set it). This seed edit is the Critic's and is not part of the conjecture.
3. Append a `run` event, then run `uv run train.py > run.log 2>&1` with a `timeout_s` kill.
4. Read `val_bpb` and `peak_vram_mb` from `run.log`. Append a `result` event: `status: "ok"`, or `"crash"` if there is no `val_bpb:` line, or `"timeout"`. `memory_gb = peak_vram_mb / 1024`, rounded to one decimal, as in the source.

## The base

At the charter and after every `advance` the Critic needs three base results `b`. After an `advance`, `b` is the corroborated arm's `h`: the code has not changed, and the seeds were already the Critic's. At the charter it runs the base three times. If any base replicate crashes, it emits `return` with the crash; the loop cannot start on a base it cannot run.

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
  if mean(h) - mean(b) > delta_min     → refuted
  else                                 → corroborated
```

Ties (`h_i = min(b)`) refute. Means are over the replicates run. An `inconclusive` verdict carries 0 bits; `refuted` and `corroborated` carry 1 (contract §2).

## Emitting

1. Append a `verdict` event citing the conjecture and every result read.
2. If `corroborated`, append an `advance` event from the base to the arm commit, citing the verdict. The arm becomes the base and `h` becomes `b`.
3. If `refuted` or `inconclusive`, the base does not move. Reset the branch to the base; the diff stays in the conjecture event.
4. Append one row per arm run to `results.tsv`: `commit`, `val_bpb` (`0.000000` on a crash), `memory_gb`, `status` from `research.json` `loop.status_map` (only the last replicate of a corroborated arm is `keep`; earlier replicates of every arm are `discard`), and the claim as `description`.

## The holdout, at RETURN

Called once by `systems/loop-research.md`. Using the harness in `docs/pin-sources.md`, run the first base and the last base three times each, seeds `seed(r, i)` and `seed(r, 3 + i)` for `i = 1..3`, with `r` the root just after the `stop` or last `verdict`. Apply the `improves` rule with `delta = charter.holdout_delta`, last base as the arm. Emit one `verdict` with `body.holdout: true`. If the first and last base are the same commit, there is nothing to test and the Critic does not open the holdout.

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
