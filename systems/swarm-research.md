# `swarm-research` — many LOOP(data) in parallel, one ledger

`type: system` · roles: **Steward**, **Planner**, **Lane**, **Auditor**, **Recorder** · subordinate to `docs/follow-research-contract.md` and `systems/loop-research.md`.

A swarm night runs many popper loops at once, one per GPU, and outputs between 10^3 and 10^4 commits. Each lane is an ordinary `loop-research` night on its own branch. A Planner assigns lanes and passes corroborated diffs between them, an Auditor replays a lane before anything merges, and a ledger records it all. The night is scored by the area under its utility curve. The swarm adds width and routing to the loop. It does not add a new way to judge.

## Specification

```json { type: notebook.cell }
{
  "id": "systems/swarm-research.md",
  "type": "system",
  "role": "run one charter across many lanes to RETURN and maximise the area under the utility curve",
  "reads": ["research.json swarm", "charter", "systems/loop-research.md", "skills/write-conjecture.md", "skills/refute-experiment.md", "skills/check-netlist.md"],
  "writes": ["swarm ledger", "lane traces", "lane and research branches"],
  "must": ["every lane is a loop-research night", "one handoff is one commit", "no diff is tested twice on the same base bytes", "transfer only corroborated diffs", "merge only audited heads that pass the rule", "open the holdout once, for the research head", "never pad the commit count"],
  "check": ["R1-R20 per lane", "S1-S10"]
}
```

## Graph

- ← [../docs/read-repository.md](../docs/read-repository.md)
- ← [loop-research.md](loop-research.md)
- ← [../skills/write-conjecture.md](../skills/write-conjecture.md)

## Tree

The swarm follows the planner → worker → verifier tree of Cursor's [`orchestrate`](https://github.com/cursor/plugins/tree/main/orchestrate/skills/orchestrate) skill, the research → review → publication branches of [solved-conjectures](https://github.com/1337isnot1337/solved-conjectures), and the one-GPU loop of [autoresearch](https://github.com/karpathy/autoresearch). It is runtime-neutral: a lane can be a Devin child session, a Cursor cloud agent, or a plain script on a GPU host.

| Node | orchestrate | popper role | Writes | Never |
|---|---|---|---|---|
| Steward | user | Steward | the charter, `stop` | conjectures, judges |
| Planner | root planner, subplanners | theory world | `assign`, `transfer`, `checkpoint` | writes a diff or a verdict |
| Lane | worker | Engine, Conjecturer, Critic, Recorder | a lane trace, one `handoff` per commit | reads another lane except through a transfer |
| Auditor | verifier | Critic (replay only) | `audit` | runs the testbed |
| Recorder | handoff log | Recorder | `merge`, `return`, the notebook | ranks lanes by feel |

Subplanners split the Planner by stem when there are more than about 24 lanes. They write the same events.

## Branches

| Ref | Holds | Mirrors |
|---|---|---|
| `popper/<tag>/lane-<k>` | lane `k`'s trace and its corroborated diffs | autoresearch's `autoresearch/<tag>` |
| `popper/<tag>/research` | the promoted head and the swarm ledger | solved-conjectures' `research` |

A swarm night creates no `popper/<tag>` ref, because git cannot hold both `popper/<tag>` and `popper/<tag>/…`. Every lane branch starts at `source.commit` plus its stem. Publishing a research head to another branch is a Steward act after RETURN.

## Changes to a lane

A lane follows `systems/loop-research.md` and R1–R17 with three changes:

1. **Lane charter.** Its charter is seq 1 of a fresh lane trace (`root_empty` as in `loop.root_empty`). It copies `delta_min` and the testbed from the swarm charter and cites the `assign` event's ledger root. `max_tested` is unbounded. The lane stops when the Planner ends the night.
2. **Transferred rows.** The `combine-lines` rows may include a block taken from another lane's corroborated verdict. The lane trace records the ledger `transfer` seq and the origin's lane root. The Engine still writes the diff, and the Conjecturer still picks it from the hinge.
3. **Deferred holdout.** A lane never opens the holdout. Only the swarm RETURN does (§Procedure).

The lane hinge offers only the constants in its slice, plus any transfers addressed to it. Engine lexicons for the swarm are `research.json swarm.lexicons`: 11 numeric constants of `train.py`, a superset of `engine.lexicons`.

## Ledger

The ledger is `traces/<tag>/swarm.jsonl` on the research branch. It is one more append-only trace, with the contract §3 algebra and its own empty root: `swarm.root_empty = sha256("popper-swarm/1")`. Every event carries `body.t`, the minutes since the charter.

```text { type: notebook.cell }
charter    = { tag, problem, testbed, delta_min, holdout_delta, lanes, stems, end_min, commits: { min, max } }  # human
assign     = { lane, stem, slice: [constant], branch }                                                    # theory; cites the charter
handoff    = { lane, n, base: true, b, runs, commit, lane_root }                                           # data; the lane's base
           | { lane, n, constant, from, to, key: { n } | { transfer }, delta, b, h, verdict, bits, runs, commit, lane_root }  # data
transfer   = { from: handoff seq, lane, constant, to }                                                     # theory; cites a corroborated handoff
audit      = { handoff, lane_root, replay: "pass" }                                                        # data; cites the handoff
merge      = { handoff, branch, commit }                                                                   # machine; cites the audit
checkpoint = { commits, rbar, U, auc, projected }                                                          # theory; every checkpoint_min
return     = { commits: { handoffs, merges, total }, short, tested, refuted, corroborated, inconclusive, verdict_bits, U, auc, research, holdout }  # frame
```

A handoff copies the lane verdict and its results. The ledger does not judge it again, but replay recomputes it.

## Stems and frontier

At the pin, the Engine offers at most 22 distinct `substitute-value` diffs (11 constants × 2 offsets). A lane is idle when it has nothing new to test. So the charter declares `stems`, which are starting bases written by the Engine under a declared clinamen:

```text { type: notebook.cell }
stem 0 = the pin
stem s = the pin with each constant K moved by (seed(r1, "stem/" + s + "/" + K) mod 3) − 1 rungs, clamped to its lexicon
r1     = the ledger root just after the charter
```

A stem is a chartered base, not an advance. Its lane replicates it three times before any conjecture. Lanes on the same stem get disjoint slices of whole constants, so at most 11 lanes share a stem. Choose `stems ≥ ⌈lanes / 11⌉`. Every corroborated advance opens up to 22 new diffs on its own branch, so the frontier widens as the night goes on.

## Commits

One handoff is one commit on the lane branch. That commit appends the lane's new trace events and, if the verdict was corroborated, the diff to `train.py`. One merge is one commit on the research branch. Nothing else counts: ledger and checkpoint commits are bookkeeping.

```text { type: notebook.cell }
commits ≈ lanes × (1 + (60·H / run_min − 3) / r̄) + merges       # r̄ = mean runs per verdict
e3:  24 lanes, 3 stems, H = 8, run_min = 5.5, r̄ = 1.6  →  ≈ 1,290     meets min 10^3 while r̄ ≤ 2.0
e4: 192 lanes, 18 stems, H = 8                         →  ≈ 10,300    the Planner stops assigning at max 10^4
```

Refutations take one run each and corroborations take three, so r̄ is low on refutation-heavy nights. A night with many corroborations makes fewer commits. It reports `short: true` and does not pad (`farming`). The count is an output of the night, not its goal.

## Utility

The goal is the area under the utility curve. A corroborated diff `c` gains utility from its **depth**, which is how much it lowered the probe metric, and from its **reach**, which is how many branches independently corroborated or promoted it.

```text { type: notebook.cell }
depth(c)   = round((mean(b) − mean(h)) / delta_min, 3)     at c's own corroborated verdict
reach_t(c) = 1 + corroborated transfers of c by t + promotions by t whose head carries c
U(t)       = Σ_c depth(c) · reach_t(c)                     in delta_min·branches
AUC        = ∫_0^end_min U(t) dt / 60                      in delta_min·branch·hours
```

U is a step function that changes only at ledger events, so AUC is exact. Nights that corroborate early and move good diffs fast to other lanes score highest. A refutation adds 0 utility but costs only one run. A lane that only farms commits adds nothing. Utility is a measure on the probe shard, not on the holdout: it never decides an advance, and the holdout never decides utility.

## Procedure

```text { type: notebook.cell }
charter := Steward                                   # lanes, stems, end_min, commits {min, max}
setup:   pin the checkout (docs/pin-sources.md); create popper/<tag>/research at source.commit
plan:    Planner assigns each lane a stem and a slice; disjoint slices on a shared stem; lane 1 is on stem 0
LANES(data) in parallel {                            # each one is LOOP(data) from loop-research
  base:     Critic runs the lane's base three times → handoff (base)
  LOOP:     hinge ∩ slice ∪ transfers → pick → diff → conjecture → refute → handoff
}
PLANNER, on each handoff {
  corroborated → transfer the block to lanes whose head lacks that constant value
                 (a lane's hinge offers it only once it is not a duplicate on that lane's base)
  head beats research by the rule → Auditor replays the lane trace → audit → merge
  idle lane → reassign it to a fresh stem, or let it end
  every checkpoint_min → checkpoint { commits, r̄, U, AUC, projected }
}
RETURN at end_min or Steward stop or commits = max:
         if the research head moved, Critic compares the pin with the research head on the holdout once
         Recorder emits return { commits, short, tally, U, AUC, research, holdout, P2 }
```

**Promotion rule.** A merge needs the `improves` rule from `skills/refute-experiment.md`, with `delta = delta_min`. It is applied to the lane head's recorded results `h` against the research head's `b`, or, while research is still at the pin, against the first stem-0 base handoff in the ledger. Lane 1 always starts on stem 0, so that base exists, and no merge happens before it. A head that beats only its own weaker stem therefore cannot replace the pin. No new runs are made, so the rule is a pure function of the ledger and is replayable.

**Audit.** The Auditor runs the checks of the `loop-research` replay cell on the candidate lane's own trace, not on `nov25`. It replaces the `research.json traces` pin with the lane trace and the handoff's `lane_root`. It adds this system's three lane changes: the charter cites the `assign`, transferred rows cite their `transfer`, and there is no holdout. The audit passes only if the lane trace replays to `lane_root` and its last verdict equals the handoff's. Before the first audit of a night, the Auditor reruns the Prover check (`skills/prove-rule.md`); if it fails, nothing merges (`unproved`). It also runs the Checker over the lane's slice: a slice that can reach a configuration `research.json` `library` forbids on the device is `unrealizable` and the Planner reslices before the lane starts, and a handoff whose base levels are not the arm results of the lane's last corroborated handoff is `mismatch` (R18); after a refuted or inconclusive handoff the base does not move, so neither do its levels.

**Holdout.** The holdout harness runs exactly as in `refute-experiment` §RETURN, between the pin and the research head, with `delta = holdout_delta`. The holdout never causes a merge or an advance.

## Refusal codes

All contract §5 codes apply in every lane. The swarm adds:

| Code | Refused |
|---|---|
| `duplicate` | a diff already tested on the same base bytes by any lane, or overlapping slices on one stem |
| `cross_talk` | a lane acting on another lane's results other than through a transfer |
| `unaudited` | a merge without a passing audit of that handoff |
| `stale_head` | a merge of a handoff that is not the lane's current head |
| `farming` | a counted commit with no handoff or merge, or a handoff with no lane verdict |
| `over_budget` | a commit after `commits.max` |

A Planner that writes a diff or a verdict is refused as `invention` or `self_verdict` (contract §5).

## Predicates

| # | Holds when |
|---|---|
| S1 | the ledger root and digest match `research.json traces`; every cite points backwards; `t` never decreases |
| S2 | each stem matches the formula from `r1`; each lane's first handoff is its base, with three runs |
| S3 | every own-key handoff is `substitute-value` on the lane's current value, inside its slice |
| S4 | every transfer comes from a corroborated own-key handoff on another lane and is picked up only by its addressee |
| S5 | no two verdict handoffs test the same constant value on the same base bytes |
| S6 | every handoff verdict equals the `improves` rule on its `b` and `h`; `b` is the lane's current base |
| S7 | every merge cites a passing audit of the lane's current head and passes the promotion rule |
| S8 | each lane clock advances by at least `runs × run_min` per handoff |
| S9 | checkpoint and return totals, U and AUC equal their recomputation; `commits ≤ max`; `short` is `commits < min` |
| S10 | the holdout verdict is the `improves` rule with `holdout_delta`, or `unopened` if research never moved |

## Worked ledger

[`traces/nov27-swarm.jsonl`](../traces/nov27-swarm.jsonl) is illustrative, like `nov25`. It has 3 lanes, 2 stems and a 90-minute night with commit bounds of 24 to 48. Results and commit IDs are synthetic, and lane traces are represented by synthetic roots. Stem 1 seeds `DEPTH 6, WARMDOWN 0.4, EMBEDDING_LR 0.4, UNEMBEDDING_LR 0.003, MATRIX_LR 0.03, SCALAR_LR 0.3`.

| Seq | t | Event | Detail |
|---|---|---|---|
| 1 | 0 | charter | lanes 3, stems 2, `delta_min = 0.001`, commits 24–48 |
| 2–4 | 0 | assign | lanes 1 and 2 on the pin with disjoint slices; lane 3 on stem 1 with all 11 constants |
| 5–7 | 16.5 | handoff × 3 | bases: lane 1 `b̄ = 0.99803`, lane 2 `0.99803`, lane 3 `1.00187` |
| 9 | 33 | handoff | lane 2 `WARMDOWN_RATIO 0.5 → 0.6` **corroborated**, depth 2.2 |
| 10 | 33 | handoff | lane 3 `DEPTH 6 → 8` **corroborated**, depth 1.767 |
| 11–12 | 33 | transfer | WARMDOWN 0.6 → lanes 1 and 3 |
| 13–14 | 33 | audit, merge | research := lane 2 head; WARMDOWN reach 2 |
| 15 | 38.5 | handoff | lane 1 `EMBEDDING_LR 0.6 → 0.8` **corroborated**, depth 1.167 |
| 19 | 49.5 | handoff | lane 3 picks the transfer; corroborated; WARMDOWN reach 3 |
| 21 | 60 | checkpoint | 13 commits, r̄ = 1.889, U = 9.534, projected 22 < 24 |
| 22 | 60.5 | handoff | lane 1 picks the transfer; corroborated; WARMDOWN reach 4 |
| 23 | 60.5 | handoff | lane 2 `FINAL_LR_FRAC` beats `min(b)` three times but gains only 0.00083: **refuted** |
| 24–25 | 60.5 | audit, merge | research := lane 1 head (EMBEDDING + WARMDOWN), beating lane 2's head by the rule |
| 38 | 88 | handoff | lane 3 `DEVICE_BATCH_SIZE 128 → 256` crash, **inconclusive** |
| 39 | 90 | return | 27 handoffs + 2 merges = 29 commits, not short; U = 15.101; AUC = 11.082; holdout corroborated |

The checkpoint at seq 21 projects a short night. The Planner records the projection and does nothing else. Later refutations cost one run each, so the night ends with 29 commits. Two of the five corroborations are transfers. Together they add 4.4 to U, which is more than any single new diff adds (at most 2.2).

P2: *Does EMBEDDING_LR 0.8 corroborate on stem 1, where the base learning rates are a rung lower?*

## Replay

Run this cell with Node.js from the repository root. It checks the ledger digest and root, stems, keys, slices, the duplicate gate, every verdict, transfers, audits, merges, lane clocks, commit totals, utility, AUC, and the holdout verdict (S1–S10). Lane traces are not replayed here; an Auditor replays each one as described under **Audit** before it signs an audit.

```js { type: notebook.cell }
const { createHash } = require("node:crypto");
const fs = require("node:fs");
const assert = require("node:assert/strict");
const sha = value => createHash("sha256").update(value).digest("hex");
const jcs = v => Array.isArray(v) ? "[" + v.map(jcs).join(",") + "]"
  : v !== null && typeof v === "object" ? "{" + Object.keys(v).sort().map(k => JSON.stringify(k) + ":" + jcs(v[k])).join(",") + "}"
  : JSON.stringify(v);
const seed = (r, i) => parseInt(sha(r + "seed/" + i).slice(0, 8), 16);
const mean = a => a.reduce((x, y) => x + y, 0) / a.length;
const r3 = x => Math.round(x * 1000) / 1000;
const improves = (b, h, delta) => h.some(x => typeof x !== "number") ? "inconclusive"
  : h.some(x => x >= Math.min(...b)) || h.length < 3 || mean(h) - mean(b) > -delta ? "refuted" : "corroborated";

const research = JSON.parse(fs.readFileSync("research.json", "utf8"));
const S = research.swarm, L = S.lexicons, pin = research.traces["traces/nov27-swarm.jsonl"];
const bytes = fs.readFileSync("traces/nov27-swarm.jsonl");
assert.equal("sha256:" + sha(bytes), pin.digest);
const events = bytes.toString("utf8").trim().split("\n").map(line => JSON.parse(line));
assert.equal(events.length, pin.events);
assert.equal(S.root_empty, sha("popper-swarm/1"));
for (const K of Object.keys(research.engine.lexicons)) assert.deepEqual(research.engine.lexicons[K], L[K]);   // superset
for (const K of Object.keys(L)) assert.ok(L[K].includes(S.pin[K]), "bad_key " + K);

const roots = [S.root_empty];
for (const e of events) roots.push(sha(roots.at(-1) + sha(jcs(e))));
assert.equal(roots.at(-1), pin.root);

const ch = events[0].body;
assert.equal(events[0].kind, "charter");
assert.ok(ch.delta_min > 0 && ch.commits.min <= ch.commits.max && ch.commits.max <= S.commits.max);
const stem = s => Object.fromEntries(Object.keys(L).map(K => {
  const j = L[K].indexOf(S.pin[K]), d = s === 0 ? 0 : seed(roots[1], "stem/" + s + "/" + K) % 3 - 1;
  return [K, L[K][Math.min(L[K].length - 1, Math.max(0, j + d))]];
}));

const lane = {}, seen = new Set(), depth = {}, reach = {}, head = { b: null, set: new Set(), commit: null };
let pinB = null, U = 0, auc = 0, last = 0, commits = 0, merges = 0, runs = 0, tested = 0, bits = 0;
const tally = { refuted: 0, corroborated: 0, inconclusive: 0 };
const at = seq => events[seq - 1];
for (const e of events.slice(1)) {
  const B = e.body;
  assert.ok(e.cites.every(c => c < e.seq), "cites forward at " + e.seq);
  assert.ok(B.t >= last && B.t <= ch.end_min, "clock at " + e.seq);
  auc += U * (B.t - last) / 60; last = B.t;
  if (e.kind === "assign") {
    assert.ok(B.lane >= 1 && B.lane <= ch.lanes && !lane[B.lane] && B.stem < ch.stems);
    if (B.lane === 1) assert.equal(B.stem, 0, "no pin base");
    assert.equal(B.branch, S.branches.lane.replace("<tag>", ch.tag).replace("<k>", B.lane));
    assert.ok(B.slice.every(K => K in L), "out_of_scope");
    for (const o of Object.values(lane)) if (o.stem === B.stem) assert.ok(B.slice.every(K => !o.slice.includes(K)), "duplicate slice");
    lane[B.lane] = { stem: B.stem, slice: B.slice, cfg: stem(B.stem), b: null, t: B.t, set: new Set(), head: null };
  }
  if (e.kind === "handoff") {
    const l = lane[B.lane];
    commits++;
    assert.ok(B.t >= +(l.t + B.runs * S.run_min).toFixed(1), "lane clock at " + e.seq);
    l.t = B.t;
    if (B.base) {
      assert.ok(l.b === null && B.runs === 3 && B.b.length === 3);
      l.b = B.b; l.head = B.commit;
      if (l.stem === 0 && !pinB) pinB = B.b;
    } else {
      assert.ok(l.b !== null, "no base");
      assert.deepEqual(B.b, l.b);
      assert.equal(B.from, l.cfg[B.constant]);
      assert.notEqual(B.from, B.to, "bad_key");
      if (B.key.transfer) {
        const T = at(B.key.transfer);
        assert.ok(T.kind === "transfer" && T.body.lane === B.lane && e.cites.includes(T.seq), "cross_talk");
        assert.equal(B.constant, T.body.constant); assert.equal(B.to, T.body.to);
      } else {
        assert.ok(l.slice.includes(B.constant), "out_of_scope at " + e.seq);
        assert.ok(research.engine.offsets.includes(B.key.n));
        const j = L[B.constant].indexOf(B.from);
        assert.equal(B.to, L[B.constant][Math.min(L[B.constant].length - 1, Math.max(0, j + B.key.n))], "pure_chance");
      }
      const sig = jcs(l.cfg) + "|" + B.constant + "=" + B.to;
      assert.ok(!seen.has(sig), "duplicate at " + e.seq); seen.add(sig);
      assert.ok(B.delta >= ch.delta_min && B.runs === B.h.length);
      assert.equal(B.verdict, improves(B.b, B.h, B.delta), "self_verdict at " + e.seq);
      assert.equal(B.bits, B.verdict === "inconclusive" ? 0 : 1);
      runs += B.runs; tested++; tally[B.verdict]++; bits += B.bits;
      if (B.verdict === "corroborated") {
        const c = B.key.transfer ? at(B.key.transfer).body.from : e.seq;
        if (!B.key.transfer) { depth[c] = r3((mean(B.b) - mean(B.h)) / ch.delta_min); reach[c] = 0; }
        reach[c]++; l.set.add(c);
        l.cfg = { ...l.cfg, [B.constant]: B.to }; l.b = B.h; l.head = B.commit;
      }
    }
  }
  if (e.kind === "transfer") {
    const o = at(B.from).body;
    assert.ok(at(B.from).kind === "handoff" && o.verdict === "corroborated" && !o.key.transfer && o.lane !== B.lane);
    assert.equal(B.constant, o.constant); assert.equal(B.to, o.to);
  }
  if (e.kind === "audit") {
    const h = at(B.handoff);
    assert.ok(e.cites.includes(B.handoff) && h.body.verdict === "corroborated" && B.lane_root === h.body.lane_root && B.replay === "pass");
  }
  if (e.kind === "merge") {
    const A = at(e.cites[0]), h = at(B.handoff).body;
    assert.ok(A.kind === "audit" && A.body.handoff === B.handoff, "unaudited at " + e.seq);
    assert.equal(h.commit, lane[h.lane].head, "merge of a stale head");
    assert.ok(pinB, "no pin base");
    assert.equal(improves(head.b ?? pinB, h.h, ch.delta_min), "corroborated", "self_verdict at " + e.seq);
    assert.equal(B.branch, S.branches.research.replace("<tag>", ch.tag));
    commits++; merges++;
    head.b = h.h; head.commit = h.commit; head.set = new Set(lane[h.lane].set);
    for (const c of head.set) reach[c]++;
  }
  U = Object.keys(depth).reduce((s, c) => s + depth[c] * reach[c], 0);
  if (e.kind === "checkpoint") {
    const rbar = r3(runs / tested);
    assert.deepEqual({ commits: B.commits, rbar: B.rbar, U: B.U, auc: B.auc, projected: B.projected },
      { commits, rbar, U: r3(U), auc: r3(auc), projected: Math.round(commits + ch.lanes * (ch.end_min - B.t) / (S.run_min * rbar)) });
  }
  if (e.kind === "return") {
    assert.ok(e.seq === events.length && B.t === ch.end_min);
    assert.ok(commits <= ch.commits.max, "over_budget");
    assert.deepEqual(B.commits, { handoffs: commits - merges, merges, total: commits });
    assert.equal(B.short, commits < ch.commits.min);
    assert.deepEqual([B.tested, B.refuted, B.corroborated, B.inconclusive, B.verdict_bits], [tested, tally.refuted, tally.corroborated, tally.inconclusive, bits]);
    assert.deepEqual([B.U, B.auc, B.research], [r3(U), r3(auc), head.commit]);
    assert.equal(B.holdout.verdict, head.commit ? improves(B.holdout.b, B.holdout.h, ch.holdout_delta) : "unopened");
  }
}
console.log("ok:", events.length, "events,", commits, "commits, U =", r3(U), ", AUC =", r3(auc), "delta_min-branch-hours");
```

## Falsification

| Challenge | Failure signal |
|---|---|
| Lane 3 also tests `WARMDOWN_RATIO 0.5 → 0.6` on the pin | `duplicate` (S5) |
| Merge lane 2's head at seq 23 without an audit | `unaudited` (S7) |
| Merge lane 1's head at seq 22 against lane 2's head when lane 1's max is not below lane 2's min | `self_verdict` (S7) |
| Add a no-op commit to reach `commits.min` | `farming` (S9) |
| Lane 1 tries `WARMDOWN_RATIO` before the transfer at seq 11 | `out_of_scope` or `cross_talk` (S3, S4) |
| Count `U` from holdout results | `peek` |
| Call the research head "the best hyperparameters" | `verification` |

## Licence

MIT, see `LICENSE`.
