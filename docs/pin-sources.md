# `pin-sources` — what the loop may run on

`type: resource.reference` · subordinate to `docs/follow-research-contract.md`, which governs on any conflict.

The testbed files, input shards and holdout harness are pinned by commit or revision and SHA-256. The tokenizer and run outputs are derived locally.

## Specification

```json { type: notebook.cell }
{
  "id": "docs/pin-sources.md",
  "type": "resource.reference",
  "role": "pin the testbed, the data split and the holdout harness to bytes",
  "reads": ["research.json source", "research.json data"],
  "writes": [],
  "must": ["pin to a commit", "digest the bytes", "cite, do not vendor", "keep the holdout sealed"],
  "check": ["R1", "R10"]
}
```

## Graph

- ← [follow-research-contract.md](follow-research-contract.md)
- → [../skills/refute-experiment.md](../skills/refute-experiment.md)

## Source

[karpathy/autoresearch](https://github.com/karpathy/autoresearch) at commit `228791fb499afffb54b46200aca536f79142f117` (25 Mar 2026). The editable file is `train.py`, a single-GPU [nanochat](https://github.com/karpathy/nanochat) GPT. The evaluator in frozen `prepare.py` reports `val_bpb` after a five-minute training budget. The README says MIT, but the repository has no `LICENSE` file at the pin. The source is cited and pinned, not vendored.

| Path | sha256 | Role in popper |
|---|---|---|
| `program.md` | `86cf987a…acecce` | the loop popper forks (`docs/vary-cybernetics.md`) |
| `prepare.py` | `4f2ba9cb…058150` | frozen: constants, data loader, `evaluate_bpb` (the data world's voice) |
| `train.py` | `2954175f…814414` | the only file a diff may touch; its two seed calls (lines 458–459 at the pin) belong to the Critic |
| `pyproject.toml` | `675c150a…f1af` | frozen: no new dependency |
| `uv.lock` | `03174c5c…a0a3` | frozen |
| `.python-version` | `7a41a413…e707` | frozen |

Full digests, including the Python version and dependency lock, are in `research.json` `source.files`.

## Data

[karpathy/climbmix-400b-shuffle](https://huggingface.co/datasets/karpathy/climbmix-400b-shuffle) at revision `915333b4f8b8684f39aeaafea600fea6f43fb703`, licence MIT on its dataset card. Digests are the Hugging Face LFS sha256 of each file.

| Split | File | sha256 | Used by |
|---|---|---|---|
| train | shard index 0 through `num_shards − 1`, `1 ≤ num_shards ≤ 6541` | Git LFS oid for each file at the pinned revision | `train.py` |
| probe | `shard_06542.parquet` | `769fe59d…9f94` | every loop verdict (the source's pinned val shard) |
| holdout | `shard_06541.parquet` | `51da1a9b…3c73` | RETURN only, once (`peek`) |

## Prepare the probe testbed

1. Clone the upstream repository into a separate checkout, check out `source.commit`, and compare each file against `source.files` before running it. From that checkout run `uv sync --locked`.
2. Choose `num_shards` (default 10) and an empty run home. Under that home's `.cache/autoresearch/data`, stage exactly the chosen train shards and the probe, never the holdout. File names are `shard_<five-digit index>.parquet`.
3. Fetch from https://huggingface.co/datasets/karpathy/climbmix-400b-shuffle/resolve/ followed by `data.revision` and the file name. Use the same path with `raw` instead of `resolve` to read the Git LFS pointer; compare the file's SHA-256 to its `oid sha256:` and the probe to `data.probe.digest`. Do not allow a missing staged file to fall back to upstream's mutable `main`.
4. With `HOME` set to that run home and `UV_CACHE_DIR` set to the cache used for sync, run `uv run --locked prepare.py --num-shards <num_shards>`. The pinned preparer skips existing shard files and fits the tokenizer on the train split only. Keep that home and its tokenizer fixed for probe runs. Reject unexpected shards: the loader reads every parquet file there except the active validation file.

## The holdout harness

The frozen evaluator reads only the file named by `VAL_SHARD`. At RETURN, if the base changed, the Critic stages the holdout from the pinned revision and checks its digest. Both bases use a copy of the pinned checkout with one patched `prepare.py` and a separate cache:

| Item | Value |
|---|---|
| Patched `prepare.py` | line 43 becomes `VAL_SHARD = MAX_SHARD - 1  # popper holdout shard (shard_06541)`; sha256 `36b9a7c3…9c88` |
| Cache | `HOME=<holdout-home>`; `<holdout-home>/.cache/autoresearch` has the same tokenizer and train shards, `shard_06541`, and **not** `shard_06542` (otherwise the loader would train on the probe shard) |
| uv | `UV_CACHE_DIR` set to the Steward's usual uv cache, so `HOME` does not trigger a fresh install |

The tokenizer is reused, not retrained: it was fitted on the train shards only, which excludes both the probe and the holdout.

## Citations

| Pin | Supports |
|---|---|
| [autoresearch `228791f`](https://github.com/karpathy/autoresearch/tree/228791fb499afffb54b46200aca536f79142f117) | testbed, loop, evaluator |
| [climbmix-400b-shuffle `915333b`](https://huggingface.co/datasets/karpathy/climbmix-400b-shuffle/tree/915333b4f8b8684f39aeaafea600fea6f43fb703) | probe and holdout shards |

## Falsification

| Challenge | Failure signal |
|---|---|
| Run with a `prepare.py` whose bytes are not `4f2ba9cb…` (or, at RETURN, `36b9a7c3…`) | `drift` |
| Stage `shard_06541` in the probe cache | `peek` |
| Score the holdout with `shard_06542` still in the cache | `drift` |
| Add a package to `pyproject.toml` | `out_of_scope` |

## Licence

MIT, see `LICENSE`.
