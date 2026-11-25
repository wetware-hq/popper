# popper

**popper** is the cybernetic auto-research engine of Wetware. It forks the loop of Andrej Karpathy's [autoresearch](https://github.com/karpathy/autoresearch), where an agent edits a small language-model training script overnight and keeps whatever lowers the validation loss. popper changes the rule for what is kept. The agent must say in advance what result would prove its idea wrong. A separate critic then tries to prove it wrong. Only ideas the data fails to refute are kept.

```text
VARY<CYBERNETICS>
LOOP(data) {
  POSITIVE hypothesis;
  NEGATIVE experiments;
} RETURN wetware-hq/popper
```

## How a night runs

1. **You write the charter.** You set the problem, the smallest gain worth keeping, and how many ideas to test. Then you can go to sleep.
2. **The Conjecturer proposes.** It reads everything tried so far, including every failure. It proposes one change to `train.py` and states the prediction: lower by at least this much, on every run.
3. **The Critic tries to refute it.** It trains the change three times, with seeds it derives from the record so nobody picks them. It stops at the first run that misses the prediction.
4. **The data decides.** If all three runs beat all three runs of the current best, by enough, the change becomes the new best. Otherwise it is kept in the record as something ruled out.
5. **The holdout is read once.** At the end, the first and the last best are each trained again and scored on a data shard the loop never saw. That result is the night's answer, and it gives you the next problem.

## Words

| Word | Means |
|---|---|
| Steward | You. You set the rules by which the loop can be wrong. You never conjecture or judge. |
| Conjecturer | The agent's positive loop: it builds on what has survived. |
| Critic | The agent's negative loop: it tries to remove what should not survive. |
| Prediction | What a change forbids, fixed before any run. |
| Verdict | One bit: corroborated or refuted. A crash is inconclusive. |
| Corroborated | Survived the test so far. Never "true". |
| Probe shard | The source's validation shard. Every verdict reads it. |
| Holdout | A second shard, sealed until the end. |
| Trace | The append-only record of everything above. |

## Start here

An agent starts at [`docs/read-repository.md`](docs/read-repository.md). A person who wants the reasoning reads [`docs/vary-cybernetics.md`](docs/vary-cybernetics.md). The contract is [`docs/follow-agent-contract.md`](docs/follow-agent-contract.md), and every other file defers to it.

## What is in the repository

| Path | Holds |
|---|---|
| `DISCLAIMER` | one-line research-use notice |
| `docs/` | router, contract, source and data pins, the variation from tulip and autoresearch |
| `skills/` | the Conjecturer and the Critic |
| `systems/loop-research.md` | the whole night, with a worked trace |
| `research.json` | pins, testbed, data split, decision rule, charter defaults |
| `ooc/popper.og` | the object graph |

popper is a specbase: documents an agent reads at run time. There is no app and no code here. To run it you need the pinned autoresearch checkout, one NVIDIA GPU and `uv`.

## Licence

MIT, see [`LICENSE`](LICENSE). autoresearch, its data and the Popper texts are cited and pinned, not vendored.
