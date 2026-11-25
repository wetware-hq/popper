# `frame-writing-engine` — tulip's metaheuristics, indexed by generality

`type: resource.reference` · non-normative; `docs/follow-research-contract.md` and `skills/write-conjecture.md` govern on any conflict.

popper generates every diff with [tulip](https://github.com/wetware-hq/tulip/tree/a2b956086f97d547f267efa47f3efabbd48995d6)'s writing engine. Each tulip skill is a metaheuristic: a fixed rule that turns a pick at a hinge into text, and never chooses. This file indexes them by **generality**, meaning how much of the search space a single hinge can reach, so a Steward can see what each procedure can and cannot propose.

## Specification

```json { type: notebook.cell }
{
  "id": "docs/frame-writing-engine.md",
  "type": "resource.reference",
  "role": "index tulip's writing-engine metaheuristics by generality and map each onto popper's search",
  "reads": ["research.json framework", "research.json engine", "skills/write-conjecture.md"],
  "writes": [],
  "must": ["the Engine never chooses", "a more general procedure contains every move of a less general one", "name what each procedure cannot reach"],
  "check": ["each row cites a pinned tulip file", "each popper procedure is defined in skills/write-conjecture.md"]
}
```

## Graph

- ← [follow-research-contract.md](follow-research-contract.md)
- → [../skills/write-conjecture.md](../skills/write-conjecture.md)
- → [vary-cybernetics.md](vary-cybernetics.md)

## Generality index

Rank 1 is the most general. Rank 1 contains the moves of ranks 2 and 3; ranks 2 and 3 overlap only where a substitution has been corroborated.

| Rank | tulip metaheuristic | popper procedure | One hinge reaches | Search analogue | Cannot reach |
|---|---|---|---|---|---|
| 1 | `traverse-tale` (Queneau, branching tale) | `traverse-tale` | the untested out-edges of the current base, inside a graph that spans the whole night | tree search over bases, with refuted edges closed | a base that no corroborated verdict produced |
| 2 | `combine-lines` (Queneau, line matrix) | `combine-lines` | any block whose column values come from the base or from corroborated arms | recombination (crossover) of survivors | a value that no survivor holds |
| 3 | `substitute-lexeme` (Lescure, S+n) | `substitute-value` | one constant moved `n` rungs along its pinned lexicon | local search, one coordinate at a time | two constants at once; a value off the lexicon |
| — | `fly-autopilot` (Type I reflex under a heading) | not used | — | — | Popper needs the pick from the Type II Conjecturer; the Type I role is the Critic |

How the ranks contain one another:

- `combine-lines` reaches diffs that change several constants at once, which `substitute-value` cannot write. It only reaches values that a survivor already holds, so an untried substitution is beyond it. Once a substitution is corroborated, its block becomes a row, and both procedures can then write that diff.
- `traverse-tale` ⊇ both. Its out-edges at a node are the options of that node's hinge, so on the current base it offers what the other two offer. It also keeps the whole night as a graph, and at RETURN shows the path next to the edges never taken.
- A more general hinge offers more distinct diffs. Its choice therefore carries more bits (contract §2), so the Conjecturer is responsible for more of the diff.

## Which procedure to charter

| Night | Procedure | Why |
|---|---|---|
| First night on a pin, nothing corroborated yet | `substitute-value` | the matrix has one row, and the tale has one node |
| Several corroborated arms that may add up | `combine-lines` | tests whether the gains compose; a refutation rules out the sum, not the parts |
| A long night with a backlog of untried edges | `traverse-tale` | keeps the whole night as one graph and shows the untaken edges at RETURN |

## Framework pin

`research.json` `framework` pins the tulip commit and the SHA-256 digests of the files this index cites. tulip is cited, not vendored. Its rules are restated in `skills/write-conjecture.md`, so popper does not read tulip at run time.

## Citations

| Pin | Supports |
|---|---|
| [tulip contract at `a2b9560`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/docs/follow-agent-contract.md) | Reader, Engine, hinge, key, choice bits, clinamen |
| [tulip `substitute-lexeme`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/skills/substitute-lexeme.md) | S+n |
| [tulip `combine-lines`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/skills/combine-lines.md) | line matrix |
| [tulip `traverse-tale`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/skills/traverse-tale.md) | branching tale |
| [tulip `fly-autopilot`](https://github.com/wetware-hq/tulip/blob/a2b956086f97d547f267efa47f3efabbd48995d6/systems/fly-autopilot.md) | Type I reflex, not used |
| [Metaheuristic](https://en.wikipedia.org/wiki/Metaheuristic) | local search, recombination, tree search |

## Falsification

| Challenge | Failure signal |
|---|---|
| A `substitute-value` hinge that offers a two-constant diff | `pure_chance` |
| A `combine-lines` key that takes a value no row holds | `bad_key` |
| Rank `substitute-value` above `traverse-tale` | the tale offers every `substitute-value` option on the current base, inside a graph of the whole night |

## Licence

MIT, see `LICENSE`. tulip is cited and pinned, not vendored.
