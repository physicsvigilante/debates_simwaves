# Opus — Response to the independent assessment

**Responding to:** [Independent assessment of sim_waves, `686f2f6`](https://github.com/physicsvigilante/debates_simwaves/blob/686f2f6c4119441c142f415ab31d415c8075487a/debates/independent-assessment-sim-waves.md).
**Baseline:** sim_waves [`7244bb6fa984c441d9fbcdcbf2954e99c44b5efb`](https://github.com/physicsvigilante/sim_waves/commit/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb) (still `main`).
**Purpose:** input to the owner's planned project review. **This authorises nothing,** and no implementation has begun.

**Evidence:**
- **[Source]** read at the baseline;
- **[Trace]** derived by following the source, not by running it;
- **[Static]** a text scan of the baseline's test YAML files, with no project code run;
- **[Math]** by hand.

No sim_waves files were modified, no project code or tests were run, and no reserved or blind-evaluation material was used.

---

## 1. Verdict

**I agree with the assessment's main conclusions and its ordering:** make verification trustworthy first, then a bounded dimensional-semantics milestone.
- **The new grading defect is real.** It is more consequential than the assessment states: I measured its reach (section 2).
- **Several criticisms of my earlier reasoning are correct,** and I concede them (section 4).

**My disagreements are about scope, not direction:**
1. The proposed milestone cannot deliver *independent* dimensional grading while result dimensions still come from the walker it does not repair. It should be honest about this common-mode limit (section 5.1).
2. The charter's fresh-author blind stage does not test a grading milestone. Its transfer evidence needs adapting, with the owner's approval, as m15a's was (section 5.4).
3. Known wrong-answer paths should get explicit fence-or-fix decisions **at the project review**, not "separately" without an owner or a date (section 6).

I also add one source-traced state hazard that supports the assessment's transactional recommendation (section 3).

---

## 2. The symbolic grading defect: confirmed, and measured

### 2.1 The mechanism

**[Source]**
- `harness.check` ends with `return equivalent(got, want), f"want {want}"` ([harness.py L64](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L64)).
- `scene.equivalent` returns a tuple, `(truth, basis)`, such as `(False, "counterexample")`, `(None, "undecided")` or `(True, "sampled")` ([scene.py L2798–L2822](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2798-L2822)).
- `_run_file_local` unpacks `ok, detail = check(...)` and tests `if not ok` ([harness.py L125–L132](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L125-L132)).

**A nonempty tuple is truthy,** so whenever `got` or `want` is not a plain number, **the grade is PASS whatever the comparison found.**
- The tuple branch inherits the problem: it uses `all(check(...)[0] for ...)`, so a symbolic list entry also passes.
- **[Math]** Witness: with A > 0, the result A against the reference 2A. `equivalent` finds a counterexample at any sampled A > 0 and returns `(False, "counterexample")`, which is graded PASS.

### 2.2 Reach: larger than "a single interface defect" suggests

**[Static]** I scanned every `expect` block in the baseline's regression, negative and example suites: 394 expected values.
- **42 expected values contain free identifiers.** About 6 of those are false positives: an `e` in `5e-9`, a unit word, a prose note.
- That leaves **about 36 expected values in about 22 scenes that reach the symbolic branch.** Examples:
  - `r58` hanging-string tension `mu0*g*s**2/2` and front `g*t**2/8`;
  - `r88` and `r89` energy expressions in `A`;
  - `r102`, `r109` and `r111` bound expressions;
  - examples `13_03_bmc_004` (`P/(4*pi*r**2)`, `Pl/(2*pi*r)`), `06_dmt_004`, `08_02_asc_005`, `23_01_bmc_005` and `23_01_bmc_007`.

**What this means:**
- **Roughly 9% of all expected values have never been checked by the harness.** Every PASS for them, in every run including m14 and m15a, was vacuous for those entries.
- The static count is a **lower bound.** A numeric expected value whose engine result is symbolic also reaches `equivalent`, and that cannot be found without running the scenes.
- **[Hypothesis]** Most of those results are probably correct: they were derived by hand and the scenes were reviewed. But "probably" is exactly what the harness exists to remove.
- **Correcting the RESULT reports.** The suite totals quoted in recent RESULT.md files (for example "regression 336/336" in m15a) are overstated for these entries. The correction should be recorded when the defect is fixed, with the grade change of every affected scene.

### 2.3 How it interacts with existing findings

- **D4-6:** `equivalent()` proves a length equal to a time. The harness's symbolic path uses the same function, so even after the tuple is fixed, symbolic grading is dimension-blind until D4-6 is closed.
- **D4-11 and D5-1:** **[Source]** the numeric path's `abs(g - w) <= tol * max(1.0, abs(w))` ([harness.py L62–L63](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L62-L63)) puts an absolute floor of `tol` in display units on every tolerance.
  - **[Math]** Under a 1% tolerance, 10⁻¹² against 10⁻¹⁰ passes, because the difference 9.9×10⁻¹¹ is under 0.01·max(1, 10⁻¹⁰) = 0.01.
  - The assessment's oracle is right.
- **Negative tests:** **[Source]** an `expect: {error: …}` scene passes if *any* observable fails with the expected kind ([harness.py L82–L85](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L82-L85)), not the intended one. The assessment's "expected-error matching" item is supported.

---

## 3. An additional state hazard, supporting "transactional observations"

**[Source]**
- `Resonator._check` records every symbolic harmonic or overtone number in `sc.harmonic_terms` while an expression is *evaluated* ([scene.py L2136–L2151](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2136-L2151)).
- Dimension checking runs after evaluation (`translate`, [expr.py L50–L67](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L50-L67)).
- `_evaluate_inner` calls `harmonic_problem(sc, sol, final=True)` **after** the observation loop, and raises a **whole-scene** `InconsistentError` from whatever is recorded ([run.py L660–L664](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L660-L664); [scene.py L2174–L2188](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2174-L2188)).

**[Trace]** So an *observation* that uses a symbolic harmonic (for example `frequency(p, harmonic=k)` on a closed-open pipe, with k a free integer symbol) can make the **whole scene** inconsistent. That happens even if the observation is itself refused later, for a dimension error.
- This contradicts FORMAT's rule that "each observable stands on its own" ([FORMAT L772–L778](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L772-L778)).
- It also makes the outcome depend on bookkeeping left by other observations.
- **Status:** a source trace only. It needs reproduction on the pinned environment before it counts as a finding.
- **Consequence:** it supports the assessment's §2.4. Observation checking should be transactional, and the harmonic-role record, which I proposed using as evidence in Round 9, is not even a stable scene property.

---

## 4. Concessions

1. **"The 37 findings reduce to two causes" overreached** (my Round 1). The assessment's classification (T 11, B 5, S 9, P 6, N 1, V 5) is more accurate.
   - **[Source]** I checked every assignment against the issue records, and they hold.
   - Metadata and search account for 25 of the 37. Physical applicability (P), numerical invariants (N) and verification (V) are separate contracts that a typed record would not fix.
2. **The S packets treated "the walker returned a type" as evidence of a well-typed expression,** and later treated a mutable register as dimensional authority. Both were wrong; Astra's Rounds 8 and 10 and section 3 above show why.
3. **My recent statement that "a local patch cannot make it sound" was too strong.** A closed, recursively checked local language could be sound. The packets failed because they depended on the legacy walker's permissive operations, not because local soundness is impossible. **Shelving S was a prioritization decision, not an impossibility result.** The assessment is right.
4. **"The physics is sound" was too optimistic as a project-wide claim.** D3-2 and D3-6 are genuine physical and numerical errors. **[Math]** I re-derived D3-2's oracle: d = 360 m, c = 340 m/s, w = 20 m/s gives t→ = 360/360 = 1 s and t← = 360/320 = 9/8 s, not 18/17 s both ways.
5. **"Scope by an invariant and its dependency closure, not by a list of files to leave untouched" is the right lesson.** The S packets' file-based stop rule forced the design around the very component that needed changing.

---

## 5. Challenges and amendments to the proposed milestone

### 5.1 Independence has a common-mode limit the plan must state

The milestone requires that "dimensioned comparisons require a trustworthy recorded dimension or an explicitly declared result contract". **[Source]** But today a result's dimension comes from the same `_Dim` walker that lets unknowns through (the S rounds; [expr.py L516–L583](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L516-L583)). And the reference parser uses the production `sc.expr` ([harness.py L20–L25](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L20-L25)).

A grading milestone that precedes the dimensional milestone therefore **cannot certify result dimensions independently.** I propose making the limit explicit:
- **References carry their own declared unit** when they are dimensional, for example `{value: 50, unit: cm}`. Comparison is in SI after converting the *reference*, using the shared unit **table** (data), but **not** the production evaluator, `_Dim` or `equivalent`.
- **A three-valued grade:** pass | fail | **uncheckable**. A reference unit whose dimension the engine's *reported* result dimension cannot confirm is **uncheckable**, not pass. The report says that the result dimension is engine-reported, so it is common-mode evidence.
- **Symbolic comparison:** a structured decision (established / disproved / unresolved), from a comparator of the bounded class the assessment proposes (polynomial and rational identities, with retained domain conditions). Production `equivalent` is not reused, so D4-6 does not leak into grading. Sampled agreement is graded **unresolved**.
- **Independent oracles:** the mutation tests the assessment lists (wrong sign, unit, scale, offset, element, coefficient), with expected verdicts computed outside the comparator.

### 5.2 Measure first, then fix

The milestone's first deliverable should be a **grade-change inventory**: the corrected comparator run against every existing scene, before any expectation is edited. Every scene whose grade changes is listed as one of:
- (a) the expectation was right, and the engine agrees: now genuinely verified;
- (b) the engine disagrees: **a new finding**, triaged under the review policy;
- (c) the expectation was wrong: corrected, with an independent derivation;
- (d) uncheckable under the new contract: listed as reduced verified coverage.

That turns section 2.2's lower bound into a measured fact, and keeps the "pass count drops" honest.

### 5.3 Migrate existing tolerances explicitly

**[Source]** Existing `tol:` values mean "relative, with a floor of `tol`" (`_tol` plus the `max(1, |w|)`). The new rule |g − w| ≤ a_tol + r_tol·|w|, with a_tol carrying the quantity's dimension, changes their meaning.
- Each existing `tol` needs a recorded reinterpretation, mechanical where the reference is well above 1 in its display unit, and reviewed otherwise.
- That keeps old PASSes from silently changing meaning.

### 5.4 The blind stage must test what the milestone changes

The assessment says to follow the charter, including the **fresh-author** blind stage. For a grading milestone, a fresh author writing scenes from book problems tests authoring, which this milestone does not change. That is the same mismatch m15a had.
- **I propose the m15a precedent:** a separate agent writes **reserved grading cases**: reference/result pairs, including mutated and correct references across units, tolerances, symbolic forms, lists and expected errors. Each has an independently computed verdict.
- They are unread by the builder before the freeze, and run against the frozen comparator.
- **This adaptation needs the owner's explicit approval,** as m15a's did.

### 5.5 Negative tests name their observable

Expected errors should match **the intended observable** and category, for example `expect: {flux: {error: rejected, message: "..."}}`, as well as whole-scene errors. An unrelated observable's refusal must not make a negative test pass.

---

## 6. Known wrong-answer paths: decide at the review, not "separately"

The assessment says accepted wrong-answer paths "deserve separately bounded correction or refusal decisions" and need not wait. I agree, but "separately" needs an owner and a date, or it becomes indefinite.

**[History]** From the issue records, the confirmed **accepted wrong values** (severity 1) include:
- D3-6 (circular-listener arrival, 1 s instead of 0.25 s);
- D3-2 (wind ignored in sound transit);
- D4-1 to D4-6 (units and dimensions);
- D2-1 and D2-2 (search completeness);
- D1-2 and D1-3;
- H17-1 to H17-3;
- and now the grading defect.

**I recommend that the project review produce, for each of these, one of the following:**
- **(a) a fence:** a sound, checkable refusal of the affected class, added now in a small PR with valid-neighbour tests, as the review policy allows;
- **(b) fix inside a named milestone**, with its order;
- **(c) accept the risk, with the owner's recorded reason.**

**[Hypothesis]**
- D3-6 and D3-2 are isolated, high-severity and on common JEE paths (Doppler and sound travel). They are good candidates for (a) or for an early small fix.
- The D4 cluster belongs to the dimensional milestone (b).

---

## 7. The architecture sections: agreement, with one caution

- **A checked front end over SymPy,** with expression nodes, a checked environment, and results with subject-qualified claims. This matches where the debate converged, and I agree.
- **Dimensions as linear constraints** (D(ab) = D(a) + D(b); D(a + b): D(a) = D(b)), so that D(u/u) = 0. I agree; this is the right design for the dimensional milestone. **Caution:** a constraint system must distinguish *determined* from *merely consistent*. A dimension variable left free by the constraints must stay **unresolved** and fail a requirement. That is exactly D4-1's failure ("lack of contradiction becomes unit inference"). Requirements should pass only when the system **uniquely determines** the dimension in question.
- **Implicit SI as an explicit numeric-profile context** (X = x/1 m, τ = t/1 s, y = (1 m) F(X, τ)). I agree. **[Math]** It gives the right derivative scaling: ∂y/∂x = F_X (a dimensionless slope), and ∂y/∂t = (1 m/s) F_τ (a velocity). This should be tested against existing `given:` waves that use `vy` and `slope`.
- **Three-valued decisions,** separate from computation outcomes, and **subject-qualified claims** (`unique_value(query)` ≠ `unique_state(model)`). I agree. Query-specific determinacy is a later, deliberate contract change.
- **Waveguides:** I agree with the assessment's position. Pipe pressure and particle velocity from a selected mode are the most valuable first physical step. Certified transcendental search is not a prerequisite. The first-arrival versus steady-state distinction stays.

---

## 8. Input for the project review

**The roadmap I would propose to the owner,** which is the assessment's with my amendments:

| Order | Milestone | Amendments |
|---|---|---|
| 0 | **Wrong-answer triage** (a review decision, not a milestone) | fence / fix-in-milestone / accept, for each severity-1 path (section 6) |
| 1 | **Reference Contract Integrity** | the grade-change inventory first (5.2); declared-unit references, a three-valued grade, and an independent bounded symbolic comparator (5.1); the tolerance migration table (5.3); per-observable negative tests (5.5); reserved *grading* cases instead of fresh-author scenes, with owner approval (5.4) |
| 2 | **Dimensional semantics and checked binding** | linear dimension constraints with "uniquely determined" as the pass criterion; the implicit-SI context; transactional observations (section 3); this absorbs the D4 cluster, H17-3, D2-4, D2-5, D5-4 and D4-2. Scoped by invariant and dependency closure |
| 3 | **Solution-set and search contracts** | H17-1/2, D1-x, D2-1/2/3 |
| 4 | **Observable applicability** | D3-x not already fenced; H17-5 |
| 5 | **Physical composition** | pipe fields first; then the qualified waveguide assembly |
| 6 | **A broad stratified evaluation** | as already agreed |

**Decisions the owner needs to make at the review:**
1. Accept or amend this order.
2. The wrong-answer triage list.
3. Whether to adapt the blind stage for milestone 1.
4. How to correct the past RESULT pass counts for the vacuous symbolic grades.
5. Whether the review policy should add the assessment's three process changes: review invariants before examples; systematic transformation and composition tests; separate ledgers for defects, scope and evidence.

*No sim_waves files were modified, no project code or tests were run, and no reserved or blind-evaluation material was used. This is input to the owner's project review and authorises nothing.*
