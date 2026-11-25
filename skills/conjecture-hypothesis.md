# `conjecture-hypothesis` — POSITIVE: propose a refutable change

`type: skill` · `spec: agent.skill` · role: **Conjecturer** · subordinate to `docs/follow-agent-contract.md`.

The Conjecturer is the loop's positive feedback. It reads everything the loop has learned, including every refutation, and proposes one bold change to the current base, with a prediction that the data could break. It is Type II: it has memory and is meant to use it. It never runs anything and never judges.

## Specification

```json { type: notebook.cell }
{
  "id": "skills/conjecture-hypothesis.md",
  "type": "skill",
  "role": "emit one conjecture event: a diff to train.py on the base, and a falsifiable prediction",
  "reads": ["charter", "base commit", "every verdict", "every refuted diff", "train.py at base", "prepare.py constants"],
  "writes": ["conjecture event"],
  "must": ["one change per conjecture", "kind and delta before any run", "delta >= charter.delta_min", "name what it forbids", "cite the refutations it learns from", "touch only train.py"],
  "check": ["R3", "R4", "R5", "R9"]
}
```

## Graph

- ← [../docs/vary-cybernetics.md](../docs/vary-cybernetics.md)
- → [refute-experiment.md](refute-experiment.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## Input and output

```text { type: notebook.cell }
input  = { charter, base: commit, trace }          # the whole trace: Type II
output = Conjecture = {
  claim:      String,                              # one sentence, in the theory's own terms
  diff:       Patch,                               # unified diff of train.py against base
  base:       commit,
  prediction: { kind: "improves" | "simplifies", delta: Number, forbids: String },
  rescues?:   seq                                  # a refuted conjecture this one revises, with a different diff
}
```

## Procedure

1. **Read the problem.** Read the charter and the current base. Read every `verdict` in the trace, and every refuted diff. A refutation says what not to try again; corroborations say where the base came from.
2. **Choose boldly.** Prefer a conjecture that forbids more: a larger predicted `delta`, or a deletion that predicts no loss. A bold conjecture that is refuted teaches more than a timid one that survives. The choice of *what* to try is the machine's own and needs no justification beyond the claim.
3. **Make one change.** Write a diff of `train.py` against the base that tests one idea. Two ideas in one diff cannot be told apart by a verdict.
4. **Stay in the testbed.** The diff must not touch `prepare.py`, `pyproject.toml`, `uv.lock`, the two seed calls in `train.py` (`torch.manual_seed`, `torch.cuda.manual_seed`; lines 458–459 at the pin), `TIME_BUDGET`, `MAX_SEQ_LEN`, `EVAL_TOKENS`, or the `evaluate_bpb` call (`out_of_scope`, `confound`). It must run within the source's memory and time limits; a crash is `inconclusive`, not a refutation, but it spends one of the charter's tests.
5. **Predict.** Set the kind and the threshold before any run.

    | `kind` | Prediction | `forbids` |
    |---|---|---|
    | `improves` | every arm replicate is below every base replicate, and the arm mean is at least `delta` below the base mean | any arm replicate `≥ min(base)`, or a mean gain below `delta` |
    | `simplifies` | the diff deletes more lines than it adds, and the arm mean is at most `charter.delta_min` above the base mean | a mean loss above `delta_min` |

    `delta ≥ charter.delta_min` (`unfalsifiable`). For `simplifies`, `delta = charter.delta_min`.
6. **Do not rescue.** A diff whose bytes match a refuted diff is refused (`ad_hoc`). A revision of a refuted idea (say, a smaller learning rate after a larger one failed) is a new conjecture with a different diff, and it cites the refutation in `rescues`.
7. **Emit.** Append one `conjecture` event with `world: "theory"`, citing the base's last verdict and any refutation it learns from. It is now registered: it cannot change (`post_hoc`).

## Refusals

`out_of_scope`, `confound`, `unfalsifiable`, `ad_hoc`, `post_hoc`, `peek` (the Conjecturer may never read the holdout).

## Falsification

| Challenge | Failure signal |
|---|---|
| A diff that also sets `torch.manual_seed(0)` | `confound` |
| "Should help a bit" with no `delta` | `unfalsifiable` |
| Resubmit a refuted diff with `delta` halved | `ad_hoc` |
| Change `forbids` after the first result | `post_hoc` |

## Licence

MIT, see `LICENSE`.
