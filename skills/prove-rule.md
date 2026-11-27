# `prove-rule` — FORMAL: check the rule with Lean 4

`type: skill` · role: **Prover** · subordinate to `docs/follow-research-contract.md`.

The Prover checks the loop's rules, not its conjectures. It states the Critic's decision rule, the Engine's clamp and the single-comparison bound as [Lean 4](https://lean-lang.org/doc) definitions and theorems, and has the pinned Lean kernel check them before a night starts. A conjecture about `train.py` is empirical and is never proved (`verification`). A property of the rule is analytic: it holds or fails for every possible result, so a proof is the right test for it, and a counterexample refutes it.

## Specification

```json { type: notebook.cell }
{
  "id": "skills/prove-rule.md",
  "type": "skill",
  "role": "state the decision rule and the engine clamp in Lean 4 and have the pinned kernel check every theorem before the base runs",
  "reads": ["research.json prover", "research.json critic", "research.json engine", "contract §3"],
  "writes": [],
  "must": ["check before the first base run", "use only the pinned toolchain and Lean core", "no sorry, admit, axiom, native_decide, implemented_by or extern", "only propext, Quot.sound and Classical.choice in #print axioms", "never state a theorem about a conjecture or a measured result"],
  "check": ["R17"]
}
```

## Graph

- ← [../docs/follow-research-contract.md](../docs/follow-research-contract.md)
- → [refute-experiment.md](refute-experiment.md)
- → [../systems/loop-research.md](../systems/loop-research.md)

## What is proved

The cell models `val_bpb` in units of 10⁻⁶ bits per byte, the precision of `results.tsv`, so each rule is exact integer arithmetic. With three base and three arm replicates, `mean(h) − mean(b) ≤ −delta` is `3·delta ≤ Σb − Σh`.

| Theorem | Statement | Backs |
|---|---|---|
| `scan_stop`, `improves_stop` | once a replicate reaches `min(b)` or crashes, no later replicate can change the verdict | stop at the first refuting replicate; never rerun (R7, R9) |
| `improves_corroborated` | a corroborated `improves` arm ran every replicate, each below `min(b)`, with mean gain at least `delta` | `critic.test.improves` (R7) |
| `improves_tightens` | corroborated at `delta'` implies corroborated at every `delta ≤ delta'` | "the gain threshold can only tighten" (§3) |
| `improves_crash` | a crash or timeout gives `inconclusive`, with 0 bits | §2, `research.json` `critic.crash` |
| `simplifies_engine` | a diff with `added = deleted` is refuted under `simplifies` before any run, whatever the data | Engine diffs are `improves` (§4.8, `unfalsifiable`) |
| `clamp_in` | the `substitute-value` index stays inside every non-empty lexicon | `skills/write-conjecture.md` clamp, `bad_key` |
| `one_in_twenty` | of the 20 three-element rank sets in six, exactly one lies below all the others | the `1 / C(6,3)` bound (§3) |

The four `example`s at the end replay the verdicts of [`traces/nov25.jsonl`](../traces/nov25.jsonl) at seq 14, 26, 35 and 49 through the same definitions.

`one_in_twenty` is a count, not a calibrated error rate. Together with `improves_corroborated` it says that, under exchangeability and no ties, one arm with no effect is corroborated with chance at most 1/20. It says nothing about the adaptive loop over many arms (`docs/vary-cybernetics.md`).

## Procedure

1. Install the toolchain pinned in `research.json` `prover` (`elan toolchain install` with that version) and check that `lean +leanprover/lean4:v4.36.0 --version` prints the pinned commit (`unpinned`, `drift`). Every `lean` call names the pin with `+`, so another active or default toolchain is never used.
2. Extract the cell from the repository root: `awk '/^```lean( |$)/{f=1;next}/^```/{f=0}f' skills/prove-rule.md > Popper.lean`. The file must be non-empty and declare every theorem in the table above; a missing or renamed fence is `unproved`, not a pass.
3. Check that the extracted file contains none of `sorry`, `admit`, `axiom `, `native_decide`, `implemented_by`, `extern`.
4. Run `lean +leanprover/lean4:v4.36.0 Popper.lean`. It must exit 0 with no `error` or `warning` line, and print one `#print axioms` line for each of the eight theorems in the table. Each such line must list only `propext`, `Quot.sound` or `Classical.choice`, or no axioms.
5. If any step fails, the night does not start: no charter base run is made, and the Steward reads the code `unproved` with the failing step. The Prover writes no trace event, so a check never changes a root or a seed.

The Prover runs once per charter, before the Critic's first base run, and again whenever `research.json` `critic` or `engine` changes. In a swarm, the Auditor reruns it before the first audit. A passing check is reported as **checked by Lean**, never as "the loop is verified": it says the rule has these properties, not that any run followed the rule (that is R7, by replay).

## Cell

```lean { type: notebook.cell }
namespace Popper

/-! Values are `val_bpb` in units of 1e-6 bits per byte, so every rule below is exact integer arithmetic. -/

inductive Verdict where
  | corroborated | refuted | inconclusive
  deriving DecidableEq, Repr

inductive Outcome where
  | ok (v : Int) | crash | timeout
  deriving DecidableEq, Repr

def bits : Verdict → Nat
  | .inconclusive => 0
  | _ => 1

def sumL (l : List Int) : Int := l.foldr (· + ·) 0

def minL : List Int → Int
  | [] => 0
  | x :: xs => xs.foldl min x

def oks : List Outcome → List Int
  | [] => []
  | .ok v :: rest => v :: oks rest
  | _ :: rest => oks rest

/-- Replicates in arrival order. `some v` is an early stop; `none` means every replicate ran and none reached `m`. -/
def scan (m : Int) : List Outcome → Option Verdict
  | [] => none
  | .ok v :: rest => if m ≤ v then some .refuted else scan m rest
  | _ :: _ => some .inconclusive

/-- `research.json` `critic.test.improves`, with three base and three arm replicates:
`mean(h) - mean(b) ≤ -delta` is `3 * delta ≤ sum b - sum h`. -/
def improves (delta : Int) (b : List Int) (h : List Outcome) : Verdict :=
  match scan (minL b) h with
  | some v => v
  | none => if 3 * delta ≤ sumL b - sumL (oks h) then .corroborated else .refuted

/-- `research.json` `critic.test.simplifies`; the line count is judged before any run. -/
def simplifies (delta : Int) (added deleted : Nat) (b : List Int) (h : List Outcome) : Verdict :=
  if deleted ≤ added then .refuted
  else if h.any (fun o => match o with | .ok _ => false | _ => true) then .inconclusive
  else if sumL (oks h) - sumL b ≤ 3 * delta then .corroborated else .refuted

/-- P1. Stop at the first refuting replicate: no later replicate can change an early stop. -/
theorem scan_stop (m : Int) (l t : List Outcome) (v : Verdict) :
    scan m l = some v → scan m (l ++ t) = some v := by
  induction l with
  | nil => simp [scan]
  | cons o rest ih =>
    cases o with
    | ok x =>
      simp only [List.cons_append, scan]
      split
      · exact id
      · exact ih
    | crash => simp [scan]
    | timeout => simp [scan]

theorem improves_stop (delta : Int) (b : List Int) (l t : List Outcome) (v : Verdict) :
    scan (minL b) l = some v → improves delta b (l ++ t) = v := by
  intro hs
  simp [improves, scan_stop _ _ _ _ hs]

theorem scan_none (m : Int) (h : List Outcome) :
    scan m h = none → ∀ o ∈ h, ∃ v, o = .ok v ∧ v < m := by
  induction h with
  | nil => simp
  | cons o rest ih =>
    cases o with
    | ok x =>
      simp only [scan]
      split
      · intro hc; cases hc
      · rename_i hlt
        intro hs o' ho'
        cases ho' with
        | head => exact ⟨x, rfl, by omega⟩
        | tail _ hm => exact ih hs o' hm
    | crash => simp [scan]
    | timeout => simp [scan]

theorem scan_not_corroborated (m : Int) (h : List Outcome) : scan m h ≠ some .corroborated := by
  induction h with
  | nil => simp [scan]
  | cons o rest ih =>
    cases o with
    | ok x =>
      simp only [scan]
      split
      · simp
      · exact ih
    | crash => simp [scan]
    | timeout => simp [scan]

/-- P2. A corroborated `improves` arm ran every replicate, each below `min(b)`, with the predicted mean gain. -/
theorem improves_corroborated (delta : Int) (b : List Int) (h : List Outcome) :
    improves delta b h = .corroborated →
      (∀ o ∈ h, ∃ v, o = .ok v ∧ v < minL b) ∧ 3 * delta ≤ sumL b - sumL (oks h) := by
  unfold improves
  split
  · rename_i v hs
    intro hv; subst hv
    exact absurd hs (scan_not_corroborated _ _)
  · rename_i hs
    split
    · intro _; exact ⟨scan_none _ _ hs, by omega⟩
    · intro hc; cases hc

/-- P3. Raising `delta` can only tighten the test (contract §3). -/
theorem improves_tightens (d d' : Int) (b : List Int) (h : List Outcome) :
    d ≤ d' → improves d' b h = .corroborated → improves d b h = .corroborated := by
  intro hd
  unfold improves
  split
  · exact id
  · split
    · intro _; simp only [ite_eq_left_iff]; omega
    · intro hc; cases hc

/-- P4. A crash or timeout ends the arm with 0 bits; it is never a refutation. -/
theorem improves_crash (delta : Int) (b : List Int) (o : Outcome) (t : List Outcome) :
    (o = .crash ∨ o = .timeout) → improves delta b (o :: t) = .inconclusive ∧ bits .inconclusive = 0 := by
  intro ho
  rcases ho with rfl | rfl <;> simp [improves, scan, bits]

/-- P5. The Engine rewrites lines in place (`added = deleted`), so a `simplifies` prediction on its diff
is refuted with no run whatever the data: it forbids every outcome (`unfalsifiable`). -/
theorem simplifies_engine (delta : Int) (n : Nat) (b : List Int) (h : List Outcome) :
    simplifies delta n n b h = .refuted := by
  simp [simplifies]

/-- P6. `substitute-value` clamps at the ends, so its index stays inside a non-empty lexicon. -/
def clampIdx (j : Nat) (n : Int) (len : Nat) : Nat := min (len - 1) (Int.toNat (j + n))

theorem clamp_in (j : Nat) (n : Int) (len : Nat) : 0 < len → clampIdx j n len < len := by
  intro hl
  unfold clampIdx
  omega

/-- The `k`-element subsets of a list, in order. -/
def subsets {α : Type} : Nat → List α → List (List α)
  | 0, _ => [[]]
  | _ + 1, [] => []
  | k + 1, x :: xs => (subsets k xs).map (x :: ·) ++ subsets (k + 1) xs

/-- The arm's ranks `s` all lie below every base rank. -/
def dominates (s : List Nat) : Bool :=
  s.all fun a => (List.range 6).all fun x => s.contains x || decide (a < x)

/-- P7. Single-comparison bound (contract §3). With no ties and exchangeable runs, the arm's three ranks
among six are one of 20 equally likely sets, and exactly one of them lies below all base ranks: 1/20. -/
theorem one_in_twenty :
    (subsets 3 (List.range 6)).length = 20 ∧
    ((subsets 3 (List.range 6)).filter dominates).length = 1 := by
  decide

/-! The worked trace `traces/nov25.jsonl`: the rule gives each recorded verdict. -/
def b0 : List Int := [998100, 997600, 998400]

example : improves 2000 b0 [.ok 997900] = .refuted := by decide
example : improves 1000 b0 [.ok 995800, .ok 996200, .ok 995500] = .corroborated := by decide
example : improves 3000 [995800, 996200, 995500] [.crash] = .inconclusive := by decide
example : improves 1000 [1001200, 1001700, 1000900] [.ok 999100, .ok 999400, .ok 998800] = .corroborated := by decide

#print axioms Popper.scan_stop
#print axioms Popper.improves_stop
#print axioms Popper.improves_corroborated
#print axioms Popper.improves_tightens
#print axioms Popper.improves_crash
#print axioms Popper.simplifies_engine
#print axioms Popper.clamp_in
#print axioms Popper.one_in_twenty

end Popper
```

## Refusals

`unpinned`, `drift`, `unproved`, `verification`.

## Falsification

| Challenge | Failure signal |
|---|---|
| Replace a proof with `sorry` "until after the night" | `unproved` (step 3) |
| Let `scan` run past a replicate at `min(b)` so the mean decides | `scan_none` and `improves_corroborated` no longer check: `unproved` |
| Claim `one_in_twenty` with `length = 2` | `decide` proves the proposition false: `unproved` |
| Add `axiom rule_sound : ...` to discharge a theorem | `unproved` (steps 3 and 4) |
| A note says the Prover "verified that depth 10 is better" | `verification` |
| Check the cell with an unpinned Lean | `unpinned` |

## Licence

MIT, see `LICENSE`.
