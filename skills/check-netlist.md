# `check-netlist` — STATIC: check the night against the library before it runs

`type: skill` · role: **Checker** · subordinate to `docs/follow-research-contract.md`.

The Checker is the loop's design-rule check, assimilated from [Cello](https://github.com/CIDARLAB/cello/tree/11644045e5ab677965a8e5765db2fb8535484a53), which compiles a Verilog specification into DNA. Cello never builds a circuit it knows the host cannot express: a user constraint file (UCF) says which gates exist and what the host allows, assignment rejects a gate whose output levels do not reach the next gate's input levels, and the design is scored by its narrowest signal separation. popper borrows those three checks and nothing else. The Checker reads the library and the trace; it never runs the testbed, never judges a conjecture and never writes a trace event, so a check can change no root, no seed and no verdict (`self_verdict`).

## Specification

```json { type: notebook.cell }
{
  "id": "skills/check-netlist.md",
  "type": "skill",
  "role": "check a charter against research.json library, check that levels propagate across advances, and report the night's margin and netlist",
  "reads": ["research.json library", "research.json checker", "research.json engine", "a trace"],
  "writes": ["netlist projection", "margin and score in the note and the return"],
  "must": ["check the charter before the first base run", "every library rule cites the result that evidences it", "levels propagate: after an advance the base is the arm's own results", "score is reported, never decisive", "write no trace event"],
  "check": ["R18", "R19", "R20"]
}
```

## Graph

- ← [../docs/follow-research-contract.md](../docs/follow-research-contract.md)
- ← [../docs/frame-design-rules.md](../docs/frame-design-rules.md)
- → [prove-rule.md](prove-rule.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## The night as a netlist

A night is a chain of gates. Each advance is one gate: its input levels are the base results `b` it was tested against, its output levels are its own results `h`, and the next conjecture is wired to those outputs.

```text { type: notebook.cell }
level(run)      = round(val_bpb * 1e6)                 # 1e-6 bits per byte, exact integers, as in the Lean cell
gate(advance)   = { base, arm, b, h }                  # from the verdict the advance cites
margin(verdict) = min(level(b)) - max(level(h))        # positive when every arm run is below every base run
score(trace)    = min over advances of margin          # the night is as strong as its narrowest gate
```

`margin` is Cello's signal separation and `score` is its circuit score, in `delta_min` units rather than relative promoter units. A corroborated `improves` verdict has `margin > 0` by the rule alone, which `skills/prove-rule.md` states as `improves_margin`. A refuted verdict has `margin ≤ 0`. So `score` adds no new judgement: it reports how much room the narrowest advance in the chain had, which `delta` alone does not say, because `delta` constrains the means and the margin constrains the worst pair.

The Conjecturer may read the score, as Cello's search is guided by its predicted score. The Critic may not: its percept is fixed (`stateful`), and nothing is kept or discarded because the score is high (`self_verdict`).

## The library (`research.json` `library`)

The library is popper's UCF: what the pinned device can be asked to run, as opposed to what the Engine can write. It names the device, the measured limits and the rules, written with three of Eugene's rule operators (Cello's UCF itself caps `NOR` gates with `logic_constraints` and orders promoters with Eugene `BEFORE` and `STARTSWITH` rules).

| Rule | Form | Means |
|---|---|---|
| `NOTCONTAINS` | `NOTCONTAINS <constant> <value>` | no diff may set that value on the pinned device |
| `NOTMORETHAN` | `NOTMORETHAN <expr> <bound>` | a declared product or sum of constants stays at or below a bound |
| `WITH` | `<constant> <value> WITH <constant> <value>` | one value is admissible only beside another |

Every rule carries `evidence`: the trace, sequence number and `status` of a result that records the failure it generalises. A rule with no such citation is `invention`; the library is not a place to put a hunch. A rule is therefore always downstream of the data: the first night on a pin has an empty rule list, pays for each crash once, and the Steward writes the rule the night's P2 asks for.

At the pin the one rule is `NOTMORETHAN DEPTH * DEVICE_BATCH_SIZE 1024`. It cites three crashes: `DEPTH 8 → 10` at [`traces/nov25.jsonl`](../traces/nov25.jsonl) e34, whose P2 asks whether the warmdown gain survives at ten layers once the batch is halved; the same move on lane 2 at [`traces/nov27-swarm.jsonl`](../traces/nov27-swarm.jsonl) e16; and lane 3's `DEVICE_BATCH_SIZE 128 → 256` at `DEPTH 8`, e38. So `DEPTH 10` stays available at `DEVICE_BATCH_SIZE 64` and is closed at 128. It forbids no option the `nov25` charter offered, because `DEVICE_BATCH_SIZE` is not in `engine.lexicons`: a swarm slice reaches it (`swarm.lexicons`), and an `engine.lexicons` night reaches it only by an `amend`. Both worked nights are the rule's evidence, so it binds charters after them (`library.from`) and never re-judges a trace already written.

## Procedure

1. **Before the night.** Enumerate the options the chartered skill can offer on the base, as `skills/write-conjecture.md` does, and the stems of a swarm charter. Any option whose value violates a rule is `unrealizable`: the Steward narrows the slice, amends the lexicon or lifts the rule with new evidence before seq 1. The Checker never silently drops an option, because an option it dropped would change every later hinge size and bits, and the trace would no longer replay.
2. **While the night runs.** Nothing. The Checker does not read a pending arm.
3. **After each verdict.** Compute `margin`. The Recorder may state it in the note as the gap between the worst arm run and the best base run.
4. **At each advance.** Check that the next verdict's `b` is exactly the arm's `h` (`mismatch`). This is Cello's input-level check: a chain whose levels are refitted between gates measures nothing, and here it would mean a conjecture was tested against results its base never produced.
5. **At RETURN.** Report `score` over the probe advances, the holdout margin separately, and emit the netlist if the charter asks for one. A night with no advance has no score.

## Netlist projection (R20)

The netlist is the night as an exchange document, in the object names of [SBOL 2.3](https://sbolstandard.org/datafiles/SBOL2.3.0.pdf), which is how Cello hands a finished design to another laboratory. It is the SBOL data model expressed in JSON, not valid SBOL RDF/XML, and `research.json` `checker.netlist` pins the mapping:

| SBOL | popper |
|---|---|
| `ModuleDefinition` | the night: tag, problem, testbed and charter |
| `FunctionalComponent` | a base or arm commit, with its three levels |
| `Interaction` | a verdict, with `kind`, `delta`, `margin` and `bits` |
| `participation` with role `base` or `arm` | the results the verdict cites |
| `Model` | `research.json` `critic.test`, the function that produced every verdict |

The projection is total: every verdict in the trace appears, refuted ones included, so a netlist cannot show only what survived. Given the netlist and `research.json`, a reader recomputes every verdict, every margin and the score without reading any prose in this repository, which is the only sense in which popper claims to speak the language.

## Check cell

Run with Node.js from the repository root. It projects the worked trace, checks that levels propagate, and recomputes the margins and the score.

```js { type: notebook.cell }
const assert = require("node:assert/strict");
const { readFileSync: read } = require("node:fs");

const config = JSON.parse(read("research.json"));
const events = [null, ...read("traces/nov25.jsonl", "utf8").trimEnd().split("\n").map(JSON.parse)];
const level = v => Math.round(v * 1e6);
const margin = body => Math.min(...body.b.map(level)) - Math.max(...body.h.map(level));

const netlist = { module: {}, components: {}, interactions: [] };
for (const event of events.slice(1)) {
  const { seq, kind, body, cites } = event;
  if (kind === "charter") netlist.module = { tag: body.tag, problem: body.problem, testbed: body.testbed };
  if (kind === "run") (netlist.components[body.commit] ??= { levels: [] });
  if (kind === "result" && body.status === "ok") {
    netlist.components[events[body.run].body.commit].levels.push(level(body.val_bpb));
  }
  if (kind !== "verdict") continue;
  const conjecture = events[body.conjecture].body;
  netlist.interactions.push({
    seq, verdict: body.verdict, bits: body.bits, holdout: body.holdout === true,
    kind: conjecture.prediction.kind, delta: level(body.holdout ? config.charter.holdout_delta : conjecture.prediction.delta),
    base: conjecture.base, arm: cites.map(id => events[id]).filter(e => e.kind === "result")
      .map(e => events[e.body.run].body.commit).find(c => c !== conjecture.base) ?? null,
    margin: body.h.length ? margin(body) : null,
  });
}

// Every verdict is projected, refutations included.
assert.equal(netlist.interactions.length, events.filter(e => e && e.kind === "verdict").length);

// R18: after an advance, the next verdict's base levels are the arm's own results.
for (const event of events.slice(1)) {
  if (event.kind !== "advance") continue;
  const advanced = events[event.cites[0]].body;
  const next = events.slice(event.seq + 1).find(e => e.kind === "verdict" && !e.body.holdout);
  if (next) assert.deepEqual(next.body.b, advanced.h, "mismatch");
}

// R19: the rule's sign agrees with the margin, and the score is the narrowest advance.
const advances = netlist.interactions.filter(i => i.verdict === "corroborated" && !i.holdout);
for (const i of netlist.interactions) {
  if (i.verdict === "corroborated") assert(i.margin > 0, "corroborated with no margin");
  if (i.verdict === "refuted") assert(i.margin <= 0, "refuted with a margin");
}
const score = Math.min(...advances.map(i => i.margin));
const holdout = netlist.interactions.find(i => i.holdout);
assert.equal(score, 1400);                       // 0.9976 - 0.9962, the only probe advance
assert.equal(holdout.margin, 1500);              // 1.0009 - 0.9994, reported apart from the score
assert.equal(score / level(config.charter.delta_min), 1.4);

// R19: no rule in the library may close an option the worked night offered.
for (const rule of config.library.rules) {
  assert(rule.evidence.length > 0, "invention");
  for (const { trace, seq, status } of rule.evidence) {
    const cited = read(trace, "utf8").trimEnd().split("\n").map(JSON.parse)[seq - 1];
    assert.equal(cited.seq, seq);
    assert(cited.body.status === status || cited.body.h?.includes(status), "invention");
  }
  assert(rule.constants.some(c => !(c in config.engine.lexicons)), "unrealizable");
}

console.log(`Netlist: ${netlist.interactions.length} interactions, score ${score / level(config.charter.delta_min)} delta_min`);
```

## Refusals

`mismatch`, `unrealizable`, `invention`, `self_verdict`, `stateful`.

## Falsification

| Challenge | Failure signal |
|---|---|
| Charter a swarm slice holding `DEPTH 10` and `DEVICE_BATCH_SIZE 128` on the pinned device | `unrealizable` (step 1) |
| Re-run the base after an advance so the next test gets fresh base levels | `mismatch` (step 4) |
| Keep a refuted diff because its margin was "nearly zero" | `self_verdict` |
| Add `NOTCONTAINS DEPTH 12` because twelve layers "obviously" will not fit | `invention` |
| Let the Critic prefer the arm with the wider margin | `stateful` |
| Publish a netlist holding only the corroborated verdicts | R20: the projection is total |
| Add the probe score and the holdout margin into one number | contract §2: the two edges are never added |

## Licence

MIT, see `LICENSE`. Cello is cited and pinned, not vendored.
