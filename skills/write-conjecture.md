# `write-conjecture` — the writing engine that generates diffs

`type: skill` · role: **Engine** · subordinate to `docs/follow-research-contract.md`.

This skill uses [tulip](https://github.com/wetware-hq/tulip/tree/a2b956086f97d547f267efa47f3efabbd48995d6)'s writing engine to generate conjectures. In tulip, a Reader picks one option at a hinge, and a deterministic Engine writes the text from a pinned corpus. Here the Conjecturer takes the Reader's place, and the Engine writes the diff to `train.py`. Each diff is then fully explained by the Conjecturer's choice plus the Engine's rule. The Conjecturer still owns the claim and the prediction. The Engine owns every byte of the diff.

This file restates all the rules popper uses, so a reader does not need tulip to run or replay them. The tulip pin is cited only to show where they come from.

## Specification

```json { type: notebook.cell }
{
  "id": "skills/write-conjecture.md",
  "type": "skill",
  "role": "offer a hinge of generated diffs on the base, take the Conjecturer's pick, derive a key, write the diff",
  "reads": ["train.py at base", "research.json engine", "every refuted diff"],
  "writes": ["hinge event", "key event"],
  "must": ["offer every distinct non-refuted diff the skill can write", "count bits as log2 of distinct diffs", "write the diff from the key alone", "never choose"],
  "check": ["R14", "R15", "R16"]
}
```

## Graph

- ← [../docs/vary-cybernetics.md](../docs/vary-cybernetics.md)
- → [conjecture-hypothesis.md](conjecture-hypothesis.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## Roles, mapped from tulip

| tulip | popper | Does | Never |
|---|---|---|---|
| Reader | Conjecturer | picks one offered option at each hinge | writes diff bytes (`not_offered`) |
| Engine | Engine | offers the hinge, derives the key, writes the diff | chooses, samples, edits a lexicon (`pure_chance`, `drift`) |
| Narrator | Recorder | states in the note what was offered, what was picked and how many bits it carried | invents an event |
| incipit, which breaks off at a hinge | the base, which waits at a hinge | — | — |
| corpus | `research.json` `engine.lexicons` and `train.py` at the base | — | — |
| clinamen (`seed: "machine"`) | a declared machine pick | — | — |

## The three procedures

The Engine reads only the hyperparameter block of `train.py`: the `NAME = value  # comment` assignments, lines 433–451 at the pin. A constant can be offered only if `research.json` `engine.lexicons` gives it a lexicon, which is a sorted list of values that includes the value at the pin.

### `substitute-value` (tulip `substitute-lexeme`, Lescure's S+n)

```text { type: notebook.cell }
key   = { constant, n }                              # n in research.json engine.offsets
L     = engine.lexicons[constant]                    # ascending; the base value must be in L, else bad_key
j     = index of the base value in L
v'    = L[clamp(j + n, 0, |L| − 1)]                  # clamp at the ends, as tulip does; recorded as clamped
line' = NAME + " = " + str(v') + padding + comment   # padding keeps the comment column, at least one space
diff  = "--- a/train.py\n+++ b/train.py\n@@ -<ln> +<ln> @@\n-<line>\n+<line'>\n"
```

The hinge offers every `(constant, n)` whose diff is not empty and does not match the bytes of a refuted diff. Two options that write the same diff count as one, for example two different offsets that both clamp to the same end. `bits = log2(distinct diffs)`.

### `combine-lines` (tulip `combine-lines`, Queneau's line matrix)

```text { type: notebook.cell }
rows  = [base block] + [the block of each corroborated arm in the trace], in trace order
cols  = the constants in engine.lexicons, in file order
key   = k in {0..r−1}^c
block'= for each column i: the value of column i in row k[i]
diff  = the lines where block' differs from the base, as one unified diff
```

The rows are the night's surviving changes, and a key recombines them. A key whose values are all the base's own writes an empty diff and is refused (`bad_key`). A row that lacks a constant is refused (`ragged`). A diff that combines lines from two or more corroborated arms is one conjecture, `improves`, and it claims that the gains add up.

### `traverse-tale` (tulip `traverse-tale`, Queneau's branching tale)

The night itself is the tale. Each node is a base, and the night starts at the pinned commit. The out-edges of a node are the options offered at its hinge. A corroborated verdict moves to a new node. A refuted or inconclusive verdict stays at the same node, and its edge is closed, because a refuted diff is no longer offered. The terminal is RETURN. At RETURN the Recorder prints the path of bases next to the edges that were never taken. Choosing between two options that write the same diff is illusory and carries 0 bits, as in tulip.

## Events

```text { type: notebook.cell }
Hinge  = { skill, offered: [Option] | Domain }          # world: machine; cites the base's charter or verdict
Choice = { picked: String, bits: Number }               # world: theory; cites the hinge
Key    = { skill, ...key, clamped?: Boolean, seed: "conjecturer" | "machine" }  # world: machine; cites the choice
```

Next comes the `conjecture` event. It cites the key and carries `constraint: { skill, lexicon, key: seq }`, and its `diff` must equal `engine(constraint, key)` byte for byte (`pure_chance`).

- **Declared clinamen.** When the charter sets `engine.seed = "machine"`, the Engine itself picks option `seed(r, "key") mod size`. Here `r` is the root just after the hinge. The key carries `seed: "machine"` and 0 Conjecturer bits, and the Recorder calls it a swerve the Conjecturer did not make.
- **Requisite variety.** The diffs a night can test are bounded by its choices plus the rule (Ashby). A diff that no offered option explains reveals a hidden source of choice (`pure_chance`).

## Worked example

The hinge at seq 8 of [`traces/nov25.jsonl`](../traces/nov25.jsonl) is on the pinned base and uses the lexicons in `research.json`:

| Option | Diff | |
|---|---|---|
| `MATRIX_LR:-1` | 0.04 → 0.03 | |
| `MATRIX_LR:+1` | 0.04 → 0.05 | picked at seq 9 |
| `WARMDOWN_RATIO:-1` | 0.5 → 0.4 | |
| `WARMDOWN_RATIO:+1` | 0.5 → 0.6 | |
| `DEPTH:-1` | 8 → 6 | |
| `DEPTH:+1` | 8 → 10 | |

Six distinct diffs, so the pick carries `log2 6 ≈ 2.585` bits. After the refutation at seq 14, the hinge at seq 16 offers five options, because `MATRIX_LR:+1` would write the refuted bytes again (`ad_hoc`). It also offers five at seq 29, on the new base: that diff's bytes are unchanged, because the line it edits is unchanged.

## Refusals

`bad_key`, `ragged`, `not_offered`, `pure_chance`, `ad_hoc`, `drift`.

## Falsification

| Challenge | Failure signal |
|---|---|
| The Conjecturer pastes in its own diff for `MATRIX_LR = 0.045` | `not_offered` |
| A diff that differs from `engine(constraint, key)` by one space | `pure_chance` |
| Offer `MATRIX_LR:+1` again after it was refuted | `ad_hoc` |
| Report 1 bit for a hinge of six distinct diffs | R15 |
| A lexicon that does not contain the base value | `bad_key` |
| `combine-lines` with a row that lacks `DEPTH` | `ragged` |

## Licence

MIT, see `LICENSE`.
