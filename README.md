# popper 🍿

**popper** is a standalone cybernetic auto-research specification. In Andrej Karpathy's [autoresearch](https://github.com/karpathy/autoresearch), a change to a small language-model training script stays if it lowers the validation loss. In popper, the Conjecturer says in advance what result would count against the change. The Critic applies that test. Only changes the data fails to refute are kept.

```text
VARY<CYBERNETICS>
LOOP(data) {
  POSITIVE hypothesis;
  NEGATIVE experiments;
} RETURN P2
```

## How a night runs

1. **You write the charter.** You set the problem, the smallest gain worth keeping, and how many ideas to test. The Prover has Lean 4 check the decision rule before any run. Then you can go to sleep.
2. **The Engine offers, the Conjecturer picks.** The Engine uses tulip's writing engine to generate a set of possible changes to `train.py`: each one shifts a hyperparameter up or down its pinned list of values. The Conjecturer reads everything tried so far, including every refutation, and picks one. The Engine writes the diff. The Conjecturer then states what result would refute it.
3. **The Critic tries to refute it.** It trains the change up to three times, using seeds derived from the trace. For an improvement, it stops when a run fails to beat the best base run.
4. **The data decides.** An improvement becomes the new base only if all three runs beat all three base runs and the mean gain meets the prediction. A simplification must remove more lines than it adds and stay within the allowed mean loss. Every verdict stays in the trace.
5. **The holdout is read at RETURN.** If the base changed, the Critic compares the first and last base on a data shard the loop never saw. The Recorder returns the result and the next problem, P2.

## How a swarm night runs

A [swarm night](systems/swarm-research.md) runs many of these loops at once, one per GPU, each on its own branch `popper/<tag>/lane-<k>`. A Planner gives each lane a starting base and a slice of hyperparameters, so no diff is tested twice on the same base. Corroborated diffs are passed to other lanes. A lane head merges into `popper/<tag>/research` only after an Auditor replays its trace and the head beats research by the same rule. Each verdict is one commit, so a night produces 10^3–10^4 commits. The night is scored by the area under its utility curve, which is depth × reach summed over corroborated diffs.

## Words

| Word | Means |
|---|---|
| Steward | You. You set the rules by which the loop can be wrong. You never conjecture or judge. |
| Engine | tulip's writing engine. It offers diffs, then writes the one that was picked. It never chooses. |
| Conjecturer | The positive loop: it picks what to try, building on what has survived. |
| Critic | The negative loop: it tries to remove what should not survive. |
| Recorder | The role that turns the trace into a notebook for the Steward. |
| Prover | The formal check: Lean 4 proves properties of the rule, such as the 1-in-20 bound. It never proves a conjecture. |
| Prediction | What a change forbids, fixed before any run. |
| Verdict | One bit: corroborated or refuted. A crash is inconclusive. |
| Corroborated | Survived the test so far. Never "true". |
| Probe shard | The source's validation shard. Every verdict reads it. |
| Holdout | A second shard, sealed until the end. |
| Trace | The append-only record of everything above. |

## Start here

Start with [`DISCLAIMER`](DISCLAIMER) and the [repository map](docs/read-repository.md). The [research contract](docs/follow-research-contract.md) defines the roles, events and rules. To run a night, set a charter, [prepare the pinned testbed](docs/pin-sources.md), and follow the [research loop](systems/loop-research.md). To inspect a complete example without a GPU, use the [worked trace and replay](systems/loop-research.md#replay).

## What is in the repository

| Path | Holds |
|---|---|
| `DISCLAIMER` | one-line research-use notice |
| `docs/` | reading order, research contract, source and data pins, principles |
| `skills/` | the Engine, the Conjecturer, the Critic and the Prover |
| `systems/loop-research.md` | the whole night, with a worked trace |
| `systems/swarm-research.md` | many lanes in parallel, one ledger, with a worked ledger |
| `research.json` | pins, testbed, data split, decision rule, charter defaults |
| `ooc/popper.og` | the object graph |
| `traces/` | a replayable example, not measured results |

popper is a specbase, not an executable. All rules and formats are defined here. Running new experiments requires the pinned autoresearch checkout and data, one NVIDIA GPU, and `uv`. The specbase follows the form of [tulip](https://github.com/wetware-hq/tulip) and uses tulip's writing engine to generate conjectures. The engine's rules are restated in [`skills/write-conjecture.md`](skills/write-conjecture.md), and its three procedures are indexed by generality in [`docs/frame-writing-engine.md`](docs/frame-writing-engine.md), so no other specification is needed to run or replay the loop.

## Licence

MIT, see [`LICENSE`](LICENSE). autoresearch and its data are cited and pinned, not vendored. The Popper texts are cited.
