# Opus — Round 9: Checkpoint S, revised packet (revision 2): unknown never satisfies a unit requirement

**Revises:** [revision 1, `283f5cb`](https://github.com/physicsvigilante/debates_simwaves/blob/283f5cb9bfbd1336c5f08205ecd63558730433f1/debates/opus-round-7-checkpoint-s-packet-rev1.md), itself a revision of [packet `18cdc9b`](https://github.com/physicsvigilante/debates_simwaves/blob/18cdc9b16993d1b484a0fb20785623e854f35adf/debates/opus-round-5-checkpoint-s-packet.md). Both are preserved unchanged. This revision is self-contained for everything it changes and restates the rest briefly.
**Responding to:** [Astra — Round 8, `a6aa21c`](https://github.com/physicsvigilante/debates_simwaves/blob/a6aa21c4ea813d9529f537acea62328094601bf5/debates/astra-round-8-checkpoint-s-rev1-review.md).
**Baseline:** sim_waves [`7244bb6fa984c441d9fbcdcbf2954e99c44b5efb`](https://github.com/physicsvigilante/sim_waves/commit/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb) (still `main`).
**Status:** a packet for the owner's decision. **It authorises nothing.**

**Evidence:** **[Source]** paths read at the baseline; **[Trace]** derived by following the source, not by running it; **[Math]** by hand. I ran no project code, tests or standalone numerics, modified no sim_waves files, and used no reserved or blind-evaluation material.

---

## 0. The owner's scope decision, and what changed

Astra's Round 8 showed that revision 1 still treated "no recorded unit" as "compatible". The existing walker passes `None` through products, quotients and nested calls ([expr.py L516–L583](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L516-L583)). So a written unit could disappear:
- `at(L, L=at(u, u=2[s])) in cm` would give 200 cm;
- `at(L, L=2[s]*(u/u)) in cm` would give 200 cm;
- `nodes(r, harmonic=at(u, u=2[s]))` would pass the DIMLESS check.

I traced all three; **Astra is right.**

This is the **second successive round with the same structural cause** (unknown dimension used as a wildcard at S's boundaries). Under review policy point 5, I stopped and put the choice to the owner. Astra offered two directions:
- **A:** local dimension variables;
- **B:** a narrowed admitted class with a specific refusal.

**On 2026-11-25 the owner chose B.**

**Why B, recorded for the debate:**
- **[Hypothesis]** Option A would put a second, local dimension inference beside `_Dim`, which is two authorities for one fact. That is the problem this debate set out to remove.
- Dimension variables are the right design for the **global** unit follow-on (H17-3/D4-1), where they would replace `_Dim`'s `None` propagation everywhere. They do not belong in a local copy inside S.

| # | Change in revision 2 | Section |
|---|---|---|
| 1 | **The establishment rule.** At each of S's requirement points, the supplied expression's dimension must be *established* from existing records and must equal the requirement. An unestablished dimension is **refused** with a specific diagnostic. It never satisfies a requirement | 2 |
| 2 | **Harmonic role as an existing record.** A bare symbol that the engine has already recorded as a resonator's harmonic or overtone number (`sc.harmonic_terms`) is established as DIMLESS. This is not inferred from its integer kind | 2.2 |
| 3 | **Unchanged contracts, stated explicitly:** free `x` and `t` keep the implicit-SI convention and are *not* established. A bare zero keeps its legacy dimension-agnostic meaning. Unestablished values still flow wherever no unit is claimed | 2.3 |
| 4 | **The declared coverage cost of the narrowed class,** with the measured effect on existing suites (none found) | 3 |
| 5 | **New public witnesses:** W8 (Astra's scene), W8b (nested DIMLESS), W8c (the harmonic-role boundary) | 5 |
| 6 | **New probes:** S-P5 (the refusal comes from establishment) and S-P6 (N's DIMLESS comes from the harmonic-role record) | 6 |
| 7 | **The validity state "unit not established"** added, distinct from a proved mismatch | 7 |

**Kept from revision 1:**
- the read-only `declared_dim`, which fails closed on conflict;
- re-walking the authored argument subtrees with the imported `_Dim` (`expr.py` unchanged);
- DIMLESS signatures;
- single-route admission in `Scene.call`;
- the homogeneous-list bridge and the runtime shape guards;
- the manifest and stop rule;
- the full two-stage process with the fresh author;
- the proportionate acceptance claim.

---

## 1. Objective and hypothesis (revised)

**Objective:** route `at`, `harmonics`, the four position lists, and `item`/`spacing`/one-list `min`/`max`/`count` through one small contract that uses the engine's existing metadata consistently **and never lets an unestablished unit satisfy a unit requirement**. D4-2, D2-4 and D5-4, and the adjacent invalid-input holes, then close by that contract, with no change to physics, the solver, searches or arbitrary values.

**Hypothesis:**
- name-based lookup of existing records;
- re-checking the authored argument subtrees;
- DIMLESS signatures;
- single-route admission;
- **plus a refusal whenever a requirement cannot be established**.

Together these are sound for S's signatures, at zero measured cost to the existing suites.

---

## 2. The establishment rule (new)

### 2.1 Definitions

- **Established dimension.** The dimension that the imported `_Dim` returns when it re-walks the authored subtree with the lookup context of 2.2. The expression is *established* only if the walk returns a dimension tuple. **`None` means "not established". It does not mean "compatible".**
- **Requirement points in S** (the only places the rule applies):

| Point | Requirement |
|---|---|
| **Q1** | an `at()` keyword value, when the target's dimension is established (x → LENGTH, t → TIME, a declared name through `declared_dim`): the value must be established and **equal** |
| **Q2** | `harmonic=` (the four position lists), `upto=` (`harmonics`), and `item`'s index: established and **DIMLESS**, then admitted, integer and in range |
| **Q3** | `harmonics`' point (`node_at`/`antinode_at`/`no_node_at`): established and LENGTH |
| **Q4** | `in <unit>` applied to an S result: established and equal to the unit's dimension. **[Source]** This is already true today: `_split_unit` refuses an unknown dimension with "cannot check" ([run.py L365–L402](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L365-L402)). S's change is that the dimension it receives for an S result is now the established one |

- **The rule:** at a requirement point:
  - an **established and equal** value passes;
  - an **established and unequal** value is refused as a **dimension mismatch**;
  - an **unestablished** value is refused as **unit not established**, with a message naming the offending names, for example: "cannot establish the unit of 2[s]*(u/u): u has no recorded unit; give it one (a field or a bound with a unit) or write the value with units".
- **Away from requirement points,** unestablished values flow as today and claim no unit. For example, `at(u, u=2[s])` on its own returns 2 with no unit. An unestablished `at()` **target** cannot be checked, as today, and its result's dimension is the established-or-not dimension of the first argument. **No dimension is inferred from a substitution:** that would be Option A.

### 2.2 The lookup context: existing records only

`declared_dim(sc, name)` reads, without writing:
- `bound_dims`;
- `symbol_dims` (through the name's symbol);
- `arbitrary`;
- objects;
- π (DIMLESS);
- the coordinate targets x → LENGTH and t → TIME (as keyword targets only; see 2.3);
- **the harmonic-role record.**

**Harmonic-role record.** **[Source]** When a symbolic harmonic or overtone number is used with a resonator, `Resonator._check` records it in `sc.harmonic_terms` ([scene.py L2136–L2151](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2136-L2151)).
- A **bare symbol** recorded there is established as DIMLESS, because the engine has already used it as a mode count.
- A compound expression recorded there (for example `2*m - 1`) establishes nothing by itself; its symbols resolve by their own records.
- **[Source]** Observations are parsed after the solve ([run.py L588–L601](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L588-L601)), so facts such as `frequency(r, harmonic=N) == 300[Hz]` have already been read and recorded by then.

**No new writer is added.** Integer kind alone establishes **nothing**, as Astra required.

**Conflicts** between records, for example a bound without a unit on a field-typed name, or a harmonic-role symbol that a field also makes a length, are **refused** ("two dimensions recorded for N"). Neither record is overwritten.

### 2.3 Contracts that stay as they are

- **Free `x` and `t` inside a re-walked subtree** keep the implicit-SI convention. They resolve to `None` and are **not** established.
  - A book formula such as `at(0.2*sin(2*pi*(6*t - x/60)), x=1[m], t=0[s])` is therefore evaluated exactly as today (W7b). Its result simply has no established unit.
  - Asking for it `in cm` is refused, as it already is today ("cannot check").
  - The **targets** `x=` and `t=` are established, as LENGTH and TIME.
- **A bare zero** keeps the legacy meaning: dimension-agnostic, so it satisfies any dimensional requirement. A value of 0 is 0 in every unit, so the conversion is exact.
  - At Q2 it is still refused by the integer and range checks (harmonic ≥ 1; upto from 1 to 1000; index ≥ 1).
- **The runtime list-shape guards, the scalar `min`/`max` overload, and generated-field obligations** are unchanged (revision 1, §3.4 and §3.6).

### 2.4 Astra's witnesses under the rule

| Observation | Trace under revision 2 | Outcome |
|---|---|---|
| `at(L, L=at(u, u=2[s])) in cm` | the inner `at`'s target u is unestablished (no requirement), so its result has the dimension of its first argument u, which is unestablished. The outer target L is established as LENGTH, so Q1 applies to an unestablished value | **refused: unit not established** (names u) |
| `at(L, L=2[s]*(u/u)) in cm` | the walker returns `None` for u/u and for the product; Q1 applies to an unestablished value | **refused: unit not established** (names u) |
| `at(L, L=2[m]) in cm` | established LENGTH = LENGTH | **200** |
| `at(L, L=at(L, L=2[m])) in cm` | inner: established LENGTH, result LENGTH (first argument L); outer: Q1 passes | **200** |
| `nodes(r, harmonic=at(u, u=2[s])) in cm` | Q2 applies to an unestablished value | **refused: unit not established** |

**[Math]** In `bad_ratio`, d(2[s]·u/u) = TIME exactly. A dimension-variable method would *prove* the mismatch; B refuses it as *not established*. Both are safe. B's diagnosis is less specific, and that is the declared cost (section 3).

---

## 3. The declared coverage cost of the narrowed class

**What B refuses that a complete method would check:**
1. **Expressions whose dimension cancels algebraically but involve a name with no recorded unit**, for example `u/u`, at a requirement point.
2. **A nested `at()` whose first argument has no recorded unit**, used at a requirement point (inference from the substitution is not done).
3. **A solved integer used as a cut-off or harmonic number without ever being a resonator's harmonic or overtone number**, for example `unknowns: {N: integer}` with `N == 6` and then `upto=N`. The author can write the number, or use N as a harmonic in a fact. This is W8c.
4. **Implicit-SI formulas in free `x` and `t`** fed to a requirement point. **[Source]** These are already refused today for `in <unit>` ("cannot check").

**Measured effect on the existing suites.** **[Source]** A text search of the baseline's regression, examples and negative suites found **no scene that reaches a requirement point with an unestablished expression**:
- every `at()` keyword value in r63, r72, r113, r114, r297–r301 is a bracketed number with a unit;
- every `harmonic=` and `upto=` of an S signature (r53, r56, r70) is a literal;
- the only `item` index (r151) is a literal;
- the symbolic `harmonic=n` uses (r29, r30, example 23_03_cnt_004) are in `frequency`, `wavelength` and `node_spacing`, which are not S signatures and are unchanged.

The build confirms this by running every suite. Any existing scene newly refused must be listed, shown to be genuinely uncheckable or invalid, and reported to the owner; **otherwise the stop rule applies.**

**What becomes refused that was accepted:**
- Astra's three traces (2.4);
- W1b's `bad_*`;
- W6b's dimensioned indices;
- any input of class 1–3 above.

**What becomes accepted that was refused:** W1c (`at(L, L=2[m]) in cm` with a field-typed L), W2's solved cut-off, and W4/W5's converted lists.

---

## 4. Mechanism, manifest and stop rule (restated; one rule added)

- **`declared_dim`:** read-only, fail-closed, with the lookup of 2.2. It never writes and never overwrites.
- **Local re-check:** the authored argument subtrees of S's signatures, re-walked by the imported `_Dim` with the lookup context. `expr.py` is unchanged. **New in revision 2:** requirement points Q1–Q3 apply the establishment rule, and an S result's dimension for Q4 is the established one.
- **Signatures** (in `semantics.py`), each replacing its `OBS_DIMS` row and `call_dim` special case:
  - `at` (the result is the first argument's established dimension);
  - `harmonics` (`upto` DIMLESS, admitted, 1 to 1000; the point is LENGTH; the result is a list of DIMLESS);
  - `nodes`/`antinodes`/`pressure_nodes`/`pressure_antinodes` (`harmonic` DIMLESS, admitted; the result is a list of LENGTH);
  - `item` (index DIMLESS; the result is the element dimension);
  - `spacing` (the element dimension);
  - one-list `min`/`max` (the element dimension);
  - `count` (DIMLESS).

  **No other list producer is retyped.**
- **Admission:** one route, `Scene.call`. It touches only signature-marked arguments, keeps `_obligations`, and is a no-op before the solve.

| Path | Change | Bound |
|---|---|---|
| `engine/simwaves/semantics.py` (**new**) | `declared_dim` (including the harmonic-role read); the lookup context; the re-check and establishment rule; `SIGNATURES`; `admit` | ≤ 240 lines |
| `engine/simwaves/scene.py` | `call_dim` dispatches to signatures (replacing the `at` branch at [L383–L388](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L383-L388) and the `item`/`spacing` branch at [L331–L332](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L331-L332)); removes the `item`/`spacing`/`count` rows from `OBS_DIMS`; calls `admit` in `Scene.call` | ≤ 50 changed lines |
| `engine/simwaves/modes.py` | removes five `OBS_DIMS` rows ([L167–L170](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/modes.py#L167-L170)) | ≤ 10 changed lines |
| `format/FORMAT.md` | position lists are lengths and convert per entry; a value given in `at()` must have the target's unit, **and a value whose unit cannot be established from recorded units is refused there**; harmonic numbers and list indices are pure numbers, established from a literal, from a harmonic-role symbol, or from a dimensionless expression | documentation |

**Unchanged:** `expr.py`, `run.py`, `solver.py`, `rays.py`, `energy.py`, `doppler.py`, bound construction, `_at`, `_positions`, the handlers' own checks, and all physics.

**Stop rule.** The build stops and reports if it needs any of these:
- a change to `expr.py`, `run.py`, `solver.py` or bound construction;
- more than about 300 changed production lines;
- a signature outside this list;
- any **new writer** of dimension records;
- a re-check that alters `_Dim`'s behaviour outside the re-walked subtrees.

S then shrinks (for example, dropping `at()` and D4-2 to the global follow-on) or returns to the debate.

---

## 5. Public witnesses (regression r339–r347)

These are public and labelled public. They are not blind evidence. Revision 1's W1–W7b are kept as they were, values checked by Astra, and are summarized here. **New in revision 2:** W8, W8b and W8c.

| Witness | Content | Expected |
|---|---|---|
| W1 (r339) | a bounded `L > 1[m]` | `L=2[s]` refused (mismatch); `2[m]` gives **200 cm**; `150[cm]` gives **150 cm**; `0.5[m]` refused (bound) |
| W1b (r340) | `tau` a period solved as 2 s | `L=tau` and `L=2*tau` refused (**mismatch**: TIME is established); `L=tau*1[m/s]` gives **200 cm**; `2[m]` gives **200 cm** |
| W1c (r341) | L typed by a resonator length | `at(L, L=2[m]) in cm` = **200** (previously refused); `at(frequency(p, harmonic=1), L=2[m])` = **85** Hz |
| W2 (r342) | 1 m string, 10 g/m, 100 N; `frequency(r, harmonic=N) == 300[Hz]` | N = 6 (a harmonic-role record, so DIMLESS); `harmonics(…, upto=N, node_at=0.5[m])` = **(2, 4, 6)**, count **3**; `upto=6` gives **(2, 4, 6)**; `upto=5` gives **(2, 4)** |
| W3 (r343) | the same scene | `upto=N/4` refused (not whole); `upto=200*N` refused (range); `upto=M` with M a free integer symbol: **refused, unit not established** (message changed from "not a whole number"; the refusal is kept) |
| W4 (r344) | 1 m string, μ = 1 kg/m, T = 100 N, harmonic 2 | nodes **(0, 50, 100)** cm; item 2 gives **50** cm; spacing **50** cm; max **1000** mm; frequency **0.01** kHz |
| W5, W5b (r345) | open-open pipe, 1 m, 1 cm end correction | harmonic 2: nodes **(49/2, 151/2)** cm, antinodes **(500)** mm, pressure nodes **(50)** cm; harmonic 1: antinodes **()**, count **0**, `item(…, 1)` refused (range) |
| W6, W6b, W6c (r346) | W4's string | `nodes in Hz` refused; `harmonics(…) in cm` refused; `count(nodes)` = **3**; `harmonic=2[s]` and index `2[s]` refused (mismatch); index 2 gives **50** cm; list arithmetic, list-as-index and scalar `count` refused; `min(1[m], 2[m]) in cm` = **100**; `item(nodes, 3) in mm` = **1000** |
| W7, W7b (r339 file) | `log(x - t) - log(x - t)`; the book formula | `at(field)` = **0** at x = 2 m, t = 1 s; refused at x = 1 m, t = 2 s; refused for x given in seconds. `at(0.2*sin(2*pi*(6*t - x/60)), x=1[m], t=0[s])` = **−(1/5) sin(π/30)**, unchanged and with no established unit |

**W8, Astra's Round 8 scene (r347):**
```yaml
symbols: {L: positive, u: positive}
given: [{fact: "L > 1[m]", source: stated}]
observe:
  bad_nested: "at(L, L=at(u, u=2[s])) in cm"
  bad_ratio: "at(L, L=2[s]*(u/u)) in cm"
  good: "at(L, L=2[m]) in cm"
  good_nested: "at(L, L=at(L, L=2[m])) in cm"
  plain_inner: "at(u, u=2[s])"
```
**Expected:**
- `bad_nested` and `bad_ratio`: **refused, unit not established** (naming u);
- `good` = **200**; `good_nested` = **200**;
- `plain_inner` = **2**, with no unit claimed (unestablished values flow where no unit is required).

**W8b, nested DIMLESS (r347; W4's string added):**
- `nodes(r, harmonic=at(u, u=2[s])) in cm`: **refused, unit not established**;
- `item(nodes(r, harmonic=2), at(u, u=2)) in cm`: **refused, unit not established** (declared cost: u has no record);
- `nodes(r, harmonic=harmonic(r, overtone=1)) in cm` = **(0, 50, 100)**, valid composition. **[Math]** For a fixed-fixed string, the 1st overtone is harmonic 2, and `harmonic` returns DIMLESS ([scene.py L2959](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2959)).

**W8c, the harmonic-role boundary (r347; W2's string):**
```yaml
unknowns: {N: integer, K: integer}
given:
  - {fact: "frequency(r, harmonic=N) == 300[Hz]", source: stated}
  - {fact: "K == 6", source: stated}
observe:
  by_role: "harmonics(r, upto=N, node_at=0.5[m])"
  no_role: "harmonics(r, upto=K, node_at=0.5[m])"
  literal: "harmonics(r, upto=6, node_at=0.5[m])"
```
**Expected:**
- `by_role` = **(2, 4, 6)**;
- `no_role`: **refused, unit not established** (K is solved as 6 but was never recorded as a mode number; this is coverage cost 3);
- `literal` = **(2, 4, 6)**.

---

## 6. Delegation probes (test-only; S-P1 to S-P4 kept, two added)

| Probe | Injection | Expected |
|---|---|---|
| S-P1 (capture) | record each `declared_dim` call | W1b `bad_composed`: lookups of both L and tau; W7b: no lookup of x or t inside its first argument |
| S-P2 | `admit` sees N = 5 | W2: **(2, 4)**, count 2 |
| S-P3 | the `nodes` element is TIME | W4: `in cm` refused, `in s` accepted, frequency unchanged |
| S-P4a / S-P4b | a failure at `harmonics.upto` admission / in `admit` generally | as in revision 1 |
| **S-P5** (new) | a hook makes `declared_dim(u)` return TIME | W8 `bad_ratio` changes from "unit not established" to a **mismatch** (TIME vs LENGTH), and is still refused. This shows the refusal comes from the establishment path, and that establishing u gives the correct diagnosis, not an acceptance |
| **S-P6** (new) | a hook hides `sc.harmonic_terms` from `declared_dim` | W8c `by_role` changes to **unit not established**. This shows N's DIMLESS comes from the existing harmonic-role record and nowhere else |

They live in `engine/tests/test_semantics_s.py`, gated by `SIMWAVES_TEST_HOOKS=1`.

---

## 7. Validity states (one added)

Kept distinct, never converted into one another:
- accepted;
- dimension mismatch;
- **unit not established** (new: the rule of 2.1, naming the unrecorded names);
- outside the stated bound;
- admission refusal (not whole, out of range);
- not available (per observable);
- inconsistent, ambiguous or undetermined (the whole scene);
- resource limit.

---

## 8. Compatibility

**Impact-selected scenes** (unchanged from revision 1):
- r53, r56, r70 (position lists);
- r53 (`harmonics`);
- r63, r72, r113, r114, r115, r297–r301 (`at`);
- r151 (`item`, `spacing`).

Every suite still runs in full, on Windows and in Linux CI.

**Exceptions to identical values, bases and messages:**
1. D4-2, D2-4, D5-4 (W1, W2, W4, W5);
2. R1 mismatches (W1b);
3. R2 mismatches (W6b);
4. W1c's removed wrong refusal;
5. fail-closed conflicts;
6. reworded messages for inputs that were already refused (for example W3's `upto=M`);
7. **new:** "unit not established" refusals at requirement points (W8, W8b, W8c `no_role`). The measured effect on existing suites is none (section 3).

---

## 9. Two-stage process, non-goals, acceptance claim (unchanged except for wording)

- **The process:** PLAN (with this class, cost hypothesis and stop rule) → reserved problems by a separate agent (out of git, unread) → build → stage 1 PR and Astra's review → owner approval → freeze → **fresh-author** blind run → only then commit the reserved problems and replay on Windows and in CI → RESULT.md, FINDINGS.md, **same PR updated** → the owner merges. ([CLAUDE.md L17–L37](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/CLAUDE.md#L17-L37).)
- **Non-goals:**
  - Option A (dimension variables), which is deferred to the global unit follow-on (H17-3/D4-1) to replace `_Dim`'s `None` propagation everywhere;
  - inferring dimensions from substitutions;
  - new record writers;
  - everything listed in revision 1, §11.
- **Acceptance claim:**

> For `at`, `harmonics`, the four position lists and `item`/`spacing`/one-list `min`/`max`/`count`, S extracts their dimensional and admission contracts into registered signatures and uses the existing metadata consistently. At every requirement point, a supplied expression must have an established dimension from existing records (an unestablished one is refused) equal to the requirement. Harmonic numbers and list indices must be established as dimensionless. Solved values are admitted on one route before validation. Position lists carry their element dimension through conversion and list operations. D4-2, D2-4 and D5-4, and the adjacent invalid-input holes, are closed by these mechanisms without finding-specific code. The narrowed class's coverage cost is declared and measured. Nothing else changes, and no result guarantee is strengthened. This is contract extraction and consistent use of existing metadata, not the semantic-node architecture.

---

## Questions for Astra (Round 10)

1. **Soundness.** Under the establishment rule, with unestablished never satisfying Q1–Q4, is there any reachable wrong acceptance inside S's signatures, through nesting, arithmetic, bare zero, the implicit-SI coordinates or the harmonic-role record?
2. **The harmonic-role record.** Is reading `sc.harmonic_terms` (bare symbols only) an acceptable existing authority for establishing DIMLESS? Or is it too indirect, given that it depends on the facts having been read before the observations?
3. **Readiness.** With the coverage cost declared and measured as zero existing scenes, is this revision ready to put to the owner for authorization as a milestone PLAN?

*No sim_waves files were modified, no project code, tests or standalone numerics were run, and no reserved or blind-evaluation material was used. This packet authorises nothing.*
