# `frame-design-rules` — Cello's design automation, indexed by generality

`type: resource.reference` · non-normative; `docs/follow-research-contract.md` and `skills/check-netlist.md` govern on any conflict.

[Cello](https://github.com/CIDARLAB/cello/tree/11644045e5ab677965a8e5765db2fb8535484a53) compiles a Verilog specification into a DNA sequence for a living cell. It is the nearest thing to popper in another field: a loop that proposes a design, checks it against what the host can physically do, and scores it by how far its signals are from being indistinguishable. This file indexes Cello's primitives by **generality**, meaning how much of popper's loop each one constrains, so a Steward can see which of them popper assimilated and which it declined.

popper takes three: the user constraint file, the input-level check and the circuit score. It declines the synthesis search, because popper already has one.

## Specification

```json { type: notebook.cell }
{
  "id": "docs/frame-design-rules.md",
  "type": "resource.reference",
  "role": "index Cello's design-automation primitives by generality and map each onto popper's verification layer",
  "reads": ["research.json design", "research.json library", "research.json checker", "skills/check-netlist.md"],
  "writes": [],
  "must": ["name what popper declines and why", "every borrowed primitive maps onto an existing popper object", "no borrowed primitive may decide a verdict"],
  "check": ["each row cites a pinned Cello file", "each popper counterpart is defined in the contract or a skill"]
}
```

## Graph

- ← [follow-research-contract.md](follow-research-contract.md)
- → [../skills/check-netlist.md](../skills/check-netlist.md)
- → [../skills/prove-rule.md](../skills/prove-rule.md)

## Generality index

Rank 1 is the most general: it constrains every conjecture a night can make. Rank 4 constrains only the report.

| Rank | Cello primitive | popper counterpart | Constrains | Why it generalises |
|---|---|---|---|---|
| 1 | UCF: `logic_constraints` (at most 12 `NOR` gates) and `eugene_part_rules` (`STARTSWITH`, `BEFORE`, `NOTMORETHAN`) | `research.json` `library`, checked at step 1 of `skills/check-netlist.md` | which options a hinge may offer at all | it closes options before any diff exists, so it bounds the whole night |
| 2 | Input-level check in gate assignment: a gate's input thresholds must contain its predecessor's output levels | R18, `mismatch` | which chains of advances are readable as one experiment | it binds each advance to the one before it |
| 3 | Circuit score: the narrowest separation between the on and off levels of the output | `margin` per verdict, `score` over the night | nothing; it reports and may guide the Conjecturer | it reads results the rule already produced |
| 4 | SBOL export and Eugene part lists: the design as a document another laboratory can build from | the netlist projection, R20 | nothing; it restates the trace | it is a change of notation, not of content |
| — | Logic synthesis: truth table → AND-Inverter graph → NOR-Inverter graph → subcircuit substitution | the Engine's hinge (`docs/frame-writing-engine.md`) | — | declined: popper already generates candidates, from a lexicon, and its options are already collapsed by distinct diff (§2) |
| — | Assignment search: breadth-first, hill climbing, simulated annealing | the Conjecturer's pick | — | declined: a search that keeps whatever scores best is the hill climber popper exists to replace (`self_verdict`) |

How the ranks contain one another:

- A rank-1 rule removes options, so it changes the size of every later hinge and therefore every `choice.bits`. This is why the Checker runs before seq 1 and never mid-night: a rule applied in the middle would make the night's bits incoherent and the trace unreplayable.
- Rank 2 is rank 1 applied along the chain rather than across the options: the levels a conjecture is tested against must be the levels its base actually produced.
- Rank 3 is downstream of both. `improves_margin` in `skills/prove-rule.md` shows the margin of a corroborated advance is positive by the rule alone, so the score can never contradict a verdict; it can only say how narrowly each one held.
- Rank 4 is a projection: it adds nothing and may drop nothing (R20).

## What Cello scores and what popper scores

Cello's score is a ratio of fluorescence levels in relative promoter units, maximised over a search; popper's is a difference of validation losses in 10⁻⁶ bits per byte, reported after the fact. The shared idea is only this: **a chain is as sound as its narrowest margin**, and a design whose margin is near zero is one the next measurement may invert. Cello uses that to pick an assignment. popper uses it to say, at RETURN, how much room the night actually had — never to keep or discard anything, which is the rule's work alone.

| | Cello | popper |
|---|---|---|
| Unit | RPU, a ratio | 10⁻⁶ bits per byte, a difference |
| Where the limits come from | measured response functions in the UCF | crashes and timeouts recorded in earlier traces |
| What the score does | drives the assignment search | reported at RETURN; may guide the Conjecturer's pick, never a verdict |
| What a near-zero margin means | the gate may not switch | the advance may not survive the next measurement |

## Why the host limits are not hyperparameters

A lexicon value is a conjecture the night may test; a library rule is a statement about the pinned device, that `DEPTH 10` at `DEVICE_BATCH_SIZE 128` exhausts its memory, as three recorded crashes show. The first is refutable by this testbed. The second is not: a crash is `inconclusive` and carries 0 bits, so the device never refutes anything, it only declines to run. Keeping them apart is what stops the library becoming a place to put a preference: each rule names the result that evidences it, or it is `invention`.

## Design pin

`research.json` `design` pins the Cello commit and the SHA-256 digests of the files this index cites. Cello is cited, not vendored: popper reads no UCF at run time, and the `library` block restates what it borrowed, in popper's own units.

## Citations

| Pin | Supports |
|---|---|
| [Cello at `1164404`](https://github.com/CIDARLAB/cello/tree/11644045e5ab677965a8e5765db2fb8535484a53) | the compiler, its stages and its search |
| [`resources/UCF/Eco1C1G1T1.UCF.json`](https://github.com/CIDARLAB/cello/blob/11644045e5ab677965a8e5765db2fb8535484a53/resources/UCF/Eco1C1G1T1.UCF.json) | `logic_constraints`, `response_functions`, `measurement_std` (RPU), `eugene_part_rules` |
| [`resources/eugene/majority_1_default.eug`](https://github.com/CIDARLAB/cello/blob/11644045e5ab677965a8e5765db2fb8535484a53/resources/eugene/majority_1_default.eug) | `PartType`, `Device`, `Rule`, `CONTAINS`, `BEFORE`, `ALL_FORWARD`, `EXACTLY` |
| [Nielsen et al., *Genetic circuit design automation*, Science 352:aac7341 (2016)](https://doi.org/10.1126/science.aac7341) | the circuit score and the input-level check |
| [SBOL 2.3](https://sbolstandard.org/datafiles/SBOL2.3.0.pdf) | `ModuleDefinition`, `FunctionalComponent`, `Interaction`, `participation`, `Model` |

## Falsification

| Challenge | Failure signal |
|---|---|
| Add a library rule from a hunch about memory | `invention` (no cited result) |
| Apply a new library rule at seq 20 of a running night | every later hinge size changes: the trace no longer replays (`drift`) |
| Pick the arm with the widest margin and call it corroborated | `self_verdict` |
| Charter Cello's annealing search over the lexicons | the search keeps what scores best with no prediction: `unfalsifiable` |
| Rank the netlist above the library | the netlist closes no option; the library closes every hinge's options |

## Licence

MIT, see `LICENSE`. Cello (BSD-2-Clause) and SBOL are cited and pinned, not vendored.
