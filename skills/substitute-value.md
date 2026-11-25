# `substitute-value` — the Engine: S+n on `train.py`

`type: skill` · role: **Engine** · subordinate to `docs/follow-research-contract.md`.

The Engine writes every diff the Conjecturer does not declare as its own. It is tulip's `substitute-lexeme` (Lescure's S+7) moved from nouns to constants. The ladder is the lexicon, the knob is the word, and `n` is the distance to move. The Conjecturer picks the knob and `n` at a hinge, and the Engine writes the bytes. The Engine never chooses.

## Specification

```json { type: notebook.cell }
{
  "id": "skills/substitute-value.md",
  "type": "skill",
  "role": "turn one pick (knob, n) at the base into the exact diff bytes of one conjecture",
  "reads": ["research.json engine", "train.py at base", "every refuted diff"],
  "writes": ["conjecture.diff", "conjecture.key"],
  "must": ["one knob per diff", "ladder order, no clamp", "keep the comment column", "offer every move but refuted bytes", "bits = log2(offered)"],
  "check": ["R14", "R15"]
}
```

## Graph

- ← [../docs/frame-writing-engine.md](../docs/frame-writing-engine.md)
- ← [conjecture-hypothesis.md](conjecture-hypothesis.md)
- → [refute-experiment.md](refute-experiment.md)

## Lexicon

`research.json` `engine.knobs` holds, for each editable constant in the hyperparameter block of `train.py` at the pin, its line number, its exact line, and its **ladder**: an ascending list of value tokens that contains the pinned value. The Steward sets the ladders with the charter. Nobody else may change them (`self_charter`). `ADAM_BETAS` (a tuple), the seed calls, and every constant of `prepare.py` are not knobs.

| Knob | Ladder | Pin |
|---|---|---|
| `ASPECT_RATIO` | 32, 48, 64, 96, 128 | 64 |
| `HEAD_DIM` | 64, 128 | 128 |
| `WINDOW_PATTERN` | "L", "SL", "SSL", "SSSL" | "SSSL" |
| `TOTAL_BATCH_SIZE` | 2\*\*17, 2\*\*18, 2\*\*19, 2\*\*20 | 2\*\*19 |
| `EMBEDDING_LR` | 0.2, 0.4, 0.6, 0.8, 1.0 | 0.6 |
| `UNEMBEDDING_LR` | 0.002, 0.004, 0.008 | 0.004 |
| `MATRIX_LR` | 0.02, 0.03, 0.04, 0.05, 0.06 | 0.04 |
| `SCALAR_LR` | 0.25, 0.5, 1.0 | 0.5 |
| `WEIGHT_DECAY` | 0.0, 0.1, 0.2, 0.3 | 0.2 |
| `WARMUP_RATIO` | 0.0, 0.05, 0.1 | 0.0 |
| `WARMDOWN_RATIO` | 0.3, 0.4, 0.5, 0.6, 0.7 | 0.5 |
| `FINAL_LR_FRAC` | 0.0, 0.05, 0.1 | 0.0 |
| `DEPTH` | 6, 8, 10, 12 | 8 |
| `DEVICE_BATCH_SIZE` | 32, 64, 128 | 128 |

## Procedure

```text { type: notebook.cell }
at(k)       := line k of train.py at the base (after every advance, the advanced line)
j(k)        := index of at(k)'s value in ladder(k)
rewrite(k,v):= "<k> = <v>", then spaces up to the column of "#" in at(k) (at least one), then the comment
move(k, n)  := "--- a/train.py\n+++ b/train.py\n@@ -L +L @@\n-" at(k) "\n+" rewrite(k, ladder(k)[j(k)+n]) "\n"
offered     := [move(k, n) for k in engine.knobs order, for n ascending,
                n ≠ 0, 0 ≤ j(k)+n < |ladder(k)|, move(k, n) not the bytes of a refuted diff]
```

1. **Offer.** Build `offered` at the current base. A refuted diff is not offered again (`ad_hoc`); the same move on a later base has the same bytes while its line is unchanged, so it stays excluded.
2. **Take the pick.** With `seed: "conjecturer"` the Conjecturer names `(knob, n)`; it must be in `offered` (`bad_key`, `ad_hoc`). With `seed: "root"` the Engine takes `offered[int(hex(sha256(r || "engine"))[0:8], 16) mod |offered|]`, where `r` is the trace root just before the conjecture.
3. **Write.** The diff is `move(knob, n)`, byte for byte. Any other bytes under this procedure are `pure_chance`.
4. **Count.** `key.offered = |offered|`. `key.bits = log2(|offered|)` rounded to three decimals for `seed: "conjecturer"`, and `0` for `seed: "root"`.

The source clamps S+7 at the end of the lexicon. The Engine refuses a move off the ladder instead (`bad_key`), because a clamped move would be a no-op diff.

## `verbatim`

An idea that no ladder expresses (a new schedule shape, a deletion for `simplifies`) uses `key = { procedure: "verbatim", seed: "conjecturer", bits: null }`. The Conjecturer writes the diff itself. It is a declared swerve whose selection the loop cannot count; the Recorder says so. It is still bound by contract §4.5 and §4.6.

## Worked example

At the pin, with nothing refuted, there are 39 moves (`4+1+3+3+4+2+4+2+3+2+4+2+3+2`). The Conjecturer picks `("MATRIX_LR", +1)`:

```text { type: notebook.cell }
key  = { procedure: "substitute-value", knob: "MATRIX_LR", n: 1, seed: "conjecturer", offered: 39, bits: 5.285 }
diff = --- a/train.py
       +++ b/train.py
       @@ -441 +441 @@
       -MATRIX_LR = 0.04        # learning rate for matrix parameters (Muon)
       +MATRIX_LR = 0.05        # learning rate for matrix parameters (Muon)
```

Once that diff is refuted, 38 moves remain. `("DEPTH", +1)` gives `DEPTH = 10              # number of transformer layers`: one digit longer, one space shorter.

## Citations

| Pin | Supports |
|---|---|
| [tulip `skills/substitute-lexeme.md` at `a2b9560`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/skills/substitute-lexeme.md) | S+n, lexicon order, offset bits |
| [autoresearch `train.py` at `228791f`](https://github.com/karpathy/autoresearch/blob/228791fb499afffb54b46200aca536f79142f117/train.py) | the hyperparameter block, lines 433–451 |

## Falsification

| Challenge | Failure signal |
|---|---|
| Pick `("DEPTH", +3)` at `DEPTH = 8` | `bad_key` |
| Write `MATRIX_LR = 0.045` under `substitute-value` | `pure_chance` |
| Pick `("MATRIX_LR", +1)` again after it was refuted | `ad_hoc` |
| Report 5.285 bits for a `seed: "root"` move | R15 |
| Add 0.045 to the `MATRIX_LR` ladder in the middle of a night | `self_charter` |

## Licence

MIT, see `LICENSE`.
