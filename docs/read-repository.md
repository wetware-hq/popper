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
- → [../systems/loop-research.md](../systems/loop-research.md)
- → [../systems/swarm-research.md](../systems/swarm-research.md)
- → [frame-design-rules.md](frame-design-rules.md)

## Route

Read `DISCLAIMER` first. The contract governs every route.

| Task | Path |
|---|---|
| Run a whole research night under a Steward's charter | contract → `systems/loop-research.md` |
| Run many lanes overnight, with transfers, merges and a commit budget | contract → `systems/loop-research.md` → `systems/swarm-research.md` |
| Generate the next diff from a hinge | contract → `skills/write-conjecture.md` |
| Choose which tulip procedure to charter, by generality | `docs/frame-writing-engine.md` |
| Propose the next hypothesis | contract → `skills/write-conjecture.md` → `skills/conjecture-hypothesis.md` |
| Test a registered hypothesis and give its verdict | contract → `skills/refute-experiment.md` |
| Check the decision rule with Lean 4 before a night | contract → `skills/prove-rule.md` |
| Check a charter against what the device can run, or report a night's margin and netlist | contract → `skills/check-netlist.md` |
| See which of Cello's design-automation primitives popper borrowed, by generality | `docs/frame-design-rules.md` |
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
    V --> W[write-conjecture]
    C --> F[frame-writing-engine]
    F --> W
    W --> H[conjecture-hypothesis]
    V --> E[refute-experiment]
    P --> E
    C --> X[prove-rule]
    X --> E
    C --> G[frame-design-rules]
    G --> N[check-netlist]
    X --> N
    N --> L
    H --> L[loop-research]
    E --> L
    L --> S[swarm-research]
```

## Licence

MIT, see `LICENSE`. Sources are cited and pinned, not vendored.
