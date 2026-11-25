# `pin-sources` — what the loop may run on

`type: resource.reference` · `spec: agent.reference` · subordinate to `docs/follow-agent-contract.md`, which governs on any conflict.

Every byte the loop runs, reads or scores against is pinned to a commit or a revision and a digest. This file lists the `$source`, the data, and the one harness popper adds.

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

- ← [follow-agent-contract.md](follow-agent-contract.md)
- → [../skills/refute-experiment.md](../skills/refute-experiment.md)

## The `$source`

[karpathy/autoresearch](https://github.com/karpathy/autoresearch) at commit `228791fb499afffb54b46200aca536f79142f117` (25 Mar 2026). An agent edits one file, `train.py` (a single-GPU [nanochat](https://github.com/karpathy/nanochat) GPT), trains for a fixed five minutes, reads `val_bpb` from the frozen `prepare.py` evaluator, and keeps or resets. The README ends "License: MIT", but the repository has no `LICENSE` file at the pin and GitHub reports no licence. popper therefore vendors none of it: it cites the files, restates the loop in its own words, and pins by digest. The Steward clones the source at the pin to run it.

| Path | sha256 | Role in popper |
|---|---|---|
| `program.md` | `86cf987a…acecce` | the loop popper forks (`docs/vary-cybernetics.md`) |
| `prepare.py` | `4f2ba9cb…058150` | frozen: constants, data loader, `evaluate_bpb` (the data world's voice) |
| `train.py` | `2954175f…814414` | the only file a diff may touch; seed lines 458–459 belong to the Critic |
| `pyproject.toml` | `675c150a…f1af` | frozen: no new dependency |
| `uv.lock` | `03174c5c…a0a3` | frozen |

Full digests are in `research.json` `source.files`.

## Data

[karpathy/climbmix-400b-shuffle](https://huggingface.co/datasets/karpathy/climbmix-400b-shuffle) at revision `915333b4f8b8684f39aeaafea600fea6f43fb703`, licence MIT on its dataset card. Digests are the Hugging Face LFS sha256 of each file.

| Split | File | sha256 | Used by |
|---|---|---|---|
| train | `shard_00000` … `shard_<num_shards − 1>`, `num_shards ≤ 6541` | per file, as downloaded by `prepare.py` | `train.py` |
| probe | `shard_06542.parquet` | `769fe59d…9f94` | every loop verdict (the source's pinned val shard) |
| holdout | `shard_06541.parquet` | `51da1a9b…3c73` | RETURN only, once (`peek`) |

`prepare.py` downloads shards `0 … num_shards − 1` plus the val shard, so `shard_06541` is never downloaded or trained on while `num_shards ≤ 6541`. A Steward who sets `--num-shards -1` trains on the holdout and must choose another (`unpinned` until pinned).

## The holdout harness

The frozen evaluator reads only the file named by `VAL_SHARD`. To score the holdout without editing the testbed the Critic uses, popper pins one patched copy, run in a separate cache:

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
| [tulip `a2b9560`](https://github.com/wetware-hq/tulip/tree/a2b956086f97d547f267efa47f3efabbd48995d6) | pin discipline: cite and digest an unlicensed source, never vendor |

## Falsification

| Challenge | Failure signal |
|---|---|
| Run with a `prepare.py` whose bytes are not `4f2ba9cb…` (or, at RETURN, `36b9a7c3…`) | `drift` |
| Download all shards, then use `shard_06541` as the holdout | `unpinned` |
| Score the holdout with `shard_06542` still in the cache | `drift` |
| Add a package to `pyproject.toml` | `out_of_scope` |

## Licence

MIT, see `LICENSE`.
