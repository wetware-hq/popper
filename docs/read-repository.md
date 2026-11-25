# `read-repository` — map the research loop

`type: resource.router` · non-normative; `docs/follow-research-contract.md` governs on any conflict.

popper is a cybernetic auto-research specbase. A Conjecturer proposes, a Critic tries to refute, and only the data decides. The contract defines the loop; `research.json` pins the testbed.

## Specification

```json { type: notebook.cell }
{
  "id": "docs/read-repository.md",
  "type": "resource.router",
  "role": "map each research operation to its governing files",
  "reads": ["DISCLAIMER"],
  "writes": [],
  "must": ["read DISCLAIMER first", "apply the contract before any skill"],
  "check": ["each route resolves inside the repository"]
}
```

## Graph

- → [follow-research-contract.md](follow-research-contract.md)
- → [vary-cybernetics.md](vary-cybernetics.md)
- → [frame-writing-engine.md](frame-writing-engine.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## Route

Read `DISCLAIMER` first. The contract governs every route.

| Task | Path |
|---|---|
| Run a whole research night under a Steward's charter | contract → `systems/loop-research.md` |
| Propose the next hypothesis | contract → `skills/conjecture-hypothesis.md` |
| Turn a pick into the exact diff | contract → `skills/substitute-value.md` |
| See how tulip's writing engine generates conjectures | `docs/frame-writing-engine.md` |
| Test a registered hypothesis and give its verdict | contract → `skills/refute-experiment.md` |
| Understand the feedback loops and what changes from autoresearch | `docs/vary-cybernetics.md` |
| Check whether a source, shard or harness may be used | `docs/pin-sources.md` |
| Audit or replay a finished trace | contract §7 → `systems/loop-research.md` §Replay |
| Traverse without reading prose | `ooc/popper.og` |
| Anything else, or a conflict | `docs/follow-research-contract.md` |

## Reading order

```mermaid { type: notebook.cell }
flowchart LR
    D[DISCLAIMER] --> R[read-repository]
    R --> C[follow-research-contract]
    C --> P[pin-sources]
    C --> V[vary-cybernetics]
    V --> H[conjecture-hypothesis]
    C --> W[frame-writing-engine]
    W --> S[substitute-value]
    H --> S
    V --> E[refute-experiment]
    P --> E
    S --> L[loop-research]
    H --> L
    E --> L
```

## Licence

MIT, see `LICENSE`. Sources are cited and pinned, not vendored.
