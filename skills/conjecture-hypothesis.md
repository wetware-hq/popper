# `conjecture-hypothesis` — POSITIVE: propose a refutable change

`type: skill` · role: **Conjecturer** · subordinate to `docs/follow-research-contract.md`.

The Conjecturer is the loop's positive feedback. It reads everything the loop has learned, including every refutation, and proposes one bold change to the current base, with a prediction that the data could break. It is Type II: it has memory and is meant to use it. It never runs anything and never judges.

## Specification

```json { type: notebook.cell }
{
  "id": "skills/conjecture-hypothesis.md",
  "type": "skill",
  "role": "pick one option at the Engine's hinge, then emit one conjecture event: the diff the Engine wrote, a claim and a falsifiable prediction",
  "reads": ["charter", "base commit", "every verdict", "every refuted diff", "train.py at base", "prepare.py constants", "the hinge"],
  "writes": ["choice event", "conjecture event"],
  "must": ["one change per conjecture", "kind and delta before any run", "delta >= charter.delta_min", "name what it forbids", "cite the refutations it learns from", "touch only train.py"],
  "check": ["R3", "R4", "R5", "R9"]
}
```

## Graph

- ← [../docs/vary-cybernetics.md](../docs/vary-cybernetics.md)
- ← [write-conjecture.md](write-conjecture.md)
- → [refute-experiment.md](refute-experiment.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## Input and output

```text { type: notebook.cell }
input  = { charter, base: commit, trace }          # the whole trace: Type II
output = Conjecture = {
  claim:      String,                              # one sentence, in the theory's own terms
  diff:       Patch,                               # engine(constraint, key), skills/write-conjecture.md
  base:       commit,
  prediction: { kind: "improves" | "simplifies", delta: Number, forbids: String },
  constraint: { skill, lexicon, key: seq },        # the key event the Engine derived from the pick
  rescues?:   seq                                  # a refuted conjecture this one revises, with a different diff
}
```

## Procedure

1. **Read the problem.** Read the charter and the current base. Read every `verdict` in the trace, and every refuted diff. A refutation says what not to try again; corroborations say where the base came from.
2. **Choose boldly.** Prefer a conjecture that forbids more, such as a larger predicted gain. The Conjecturer chooses what to try and records the idea in the claim.
3. **Pick, do not write.** The Engine offers a hinge of diffs on the base (`skills/write-conjecture.md`). Pick one option. The Engine derives the key and writes the diff. The Conjecturer never writes diff bytes (`not_offered`, `pure_chance`). A `substitute-value` option changes one constant, so it tests one idea; a `combine-lines` key that joins corroborated arms tests one idea, that their gains add up.
4. **Stay in the testbed.** The diff must not touch `prepare.py`, `pyproject.toml`, `uv.lock`, the two seed calls in `train.py` (`torch.manual_seed`, `torch.cuda.manual_seed`; lines 458–459 at the pin), `TIME_BUDGET`, `MAX_SEQ_LEN`, `EVAL_TOKENS`, or the `evaluate_bpb` call (`out_of_scope`, `confound`). It must run within the source's memory and time limits; a crash is `inconclusive`, not a refutation, but it spends one of the charter's tests.
5. **Predict.** Set the kind and the threshold before any run.

    | `kind` | Prediction | `forbids` |
    |---|---|---|
    | `improves` | every arm replicate is below every base replicate, and the arm mean is at least `delta` below the base mean | any arm replicate `≥ min(base)`, or a mean gain below `delta` |
    | `simplifies` | the diff deletes more lines than it adds, and the arm mean is at most `delta` above the base mean | no net deletion, or a mean loss above `delta` |

    `delta ≥ charter.delta_min` (`unfalsifiable`). For `simplifies`, `delta = charter.delta_min`.
6. **Do not rescue.** A diff whose bytes match a refuted diff is refused (`ad_hoc`). A revision of a refuted idea (say, a smaller learning rate after a larger one failed) is a new conjecture with a different diff, and it cites the refutation in `rescues`.
7. **Emit.** Append one `conjecture` event with `world: "theory"`, citing the current base's verdict or its charter and initial results, and any refutation it learns from. It is now registered: it cannot change (`post_hoc`).

## Refusals

`not_offered`, `pure_chance`, `out_of_scope`, `confound`, `unfalsifiable`, `ad_hoc`, `post_hoc`, `peek` (the Conjecturer may never read the holdout).

## Falsification

| Challenge | Failure signal |
|---|---|
| A diff that also sets `torch.manual_seed(0)` | `confound` |
| "Should help a bit" with no `delta` | `unfalsifiable` |
| Resubmit a refuted diff with `delta` halved | `ad_hoc` |
| Change `forbids` after the first result | `post_hoc` |

## Licence

MIT, see `LICENSE`.
