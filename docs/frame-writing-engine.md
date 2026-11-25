# `frame-writing-engine` — tulip's writing engine as popper's generator

`type: resource.reference` · subordinate to `docs/follow-research-contract.md`, which governs on any conflict.

popper generates conjectures with the writing engine of [tulip](https://github.com/wetware-hq/tulip/tree/a2b956086f97d547f267efa47f3efabbd48995d6). In tulip, a Reader picks at a hinge, an Engine writes bytes by a fixed rule, and a Narrator says who chose what, in bits. In popper, the Conjecturer picks a move, the Engine writes the diff, the data returns a verdict, and the Recorder counts both directions: the bits of selection the theory put in and the bits of verdict the data gave back.

tulip is cited and pinned in `research.json` `framework`, like the autoresearch source. Its rules are restated here and in `skills/substitute-value.md`, so popper does not need tulip at run time.

## Specification

```json { type: notebook.cell }
{
  "id": "docs/frame-writing-engine.md",
  "type": "resource.reference",
  "role": "map tulip's writing engine onto the positive loop, and rank the residual features it patches",
  "reads": ["research.json framework", "research.json engine"],
  "writes": [],
  "must": ["the Engine never chooses", "every diff byte traces to a pick, a rule or a declared swerve", "count selection bits", "name every swerve"],
  "check": ["R11", "R14", "R15"]
}
```

## Graph

- ← [follow-research-contract.md](follow-research-contract.md)
- → [../skills/substitute-value.md](../skills/substitute-value.md)
- → [../skills/conjecture-hypothesis.md](../skills/conjecture-hypothesis.md)

## The mapping

| tulip | popper | Where |
|---|---|---|
| Reader picks at a hinge (`choice`) | Conjecturer picks `(knob, n)` from `offered` | `conjecture.key` |
| Engine: `text = engine(constraint, key)` | Engine: `diff = move(knob, n)` at the base | `skills/substitute-value.md` |
| `substitute-lexeme` (S+n over a lexicon) | `substitute-value` (S+n over a ladder of values) | `research.json` `engine.knobs` |
| pinned corpus | `train.py` at the base, pinned through the source | `research.json` `source` |
| `bits(choice) = log2(offered)` | `key.bits = log2(\|offered\|)` | contract §2 |
| `seed: "machine"`, a declared clinamen | `seed: "root"` (derived) or `verbatim` (unmeasured) | contract §2 |
| `pure_chance`: selection no rule explains | `pure_chance`: diff bytes the key does not produce | contract §5 |
| Narrator names the swerve | Recorder names the pick as a swerve the Steward did not make | contract §6 |
| Reader: the only human | Steward: the only human; writes the ladders with the charter | contract §0 |

The Conjecturer stays Type II: it reads the whole trace and chooses what to try. What changes is that its choice is a pick from a finite, recorded set, so the trace can say how much selection it made.

## Residuals patched

The features of the writing engine that popper lacked, ranked by expected utility gain for a reader of a finished night.

| Rank | tulip feature | Gap in popper before | Patch | Gain |
|---|---|---|---|---|
| 1 | The Engine never chooses; replay regenerates bytes | Diffs were freehand; only the diff format was checked | Every `substitute-value` diff is regenerated from its key on replay (R14, `pure_chance`) | high: a diff can no longer carry an unrecorded second change |
| 2 | Choice bits on the edge | Only verdict bits were counted | `key.bits`, and `return.selection` next to `edge` (R15) | high: the night shows the theory spent 15.781 bits and the data returned 2 |
| 3 | Mechanical `ad_hoc` via offered set | `ad_hoc` relied on comparing bytes by hand | A refuted diff is dropped from `offered` | medium |
| 4 | Declared clinamen | No way to let the trace choose | `seed: "root"` move, 0 bits | medium |
| 5 | Narrator names who chose | Notes named verdicts, not picks | Notes state the pick, its bits, and that it was a swerve (§6) | medium |
| — | Calvino frame and title sentence | — | Not patched: notes already alternate with the conjecture and verdict they cite | low |
| — | Type I autopilot under a heading | — | Not patched: the Critic is already Type I and the charter already acts as the heading | low |

## Citations

| Pin | Supports |
|---|---|
| [tulip contract at `a2b9560`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/docs/follow-agent-contract.md) | the Engine law, choice bits, clinamen, `pure_chance` |
| [tulip `substitute-lexeme` at `a2b9560`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/skills/substitute-lexeme.md) | S+n |
| [tulip `compose-frame` at `a2b9560`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/systems/compose-frame.md) | pick, key, text, chapter |
| [Clinamen](https://en.wikipedia.org/wiki/Clinamen) | a declared swerve |
| [Variety (cybernetics)](https://en.wikipedia.org/wiki/Variety_(cybernetics)) | arms vary no more than picks and rules allow |

## Falsification

| Challenge | Failure signal |
|---|---|
| A `substitute-value` conjecture whose diff also edits `WARMUP_RATIO` | `pure_chance` |
| Note says "the loop chose a higher learning rate" with no bits | `invention` (R11) |
| Return reports `selection` without the `verbatim` picks marked unmeasured | `invention` |

## Licence

MIT, see `LICENSE`.
