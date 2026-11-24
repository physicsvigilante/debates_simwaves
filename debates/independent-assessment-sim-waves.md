# Independent assessment of sim_waves

## Executive judgment

**Preserve the engine, but change the way it establishes and communicates correctness. Do not restart the waveguide implementation or revive Checkpoint S yet.**

My diagnosis is more specific than “good physics, weak mathematical infrastructure”:

> **sim_waves contains useful physical models, but its execution pipeline does not consistently preserve the conditions under which an expression, a solution, or a verification result is trustworthy.**

That problem crosses dimensions, domains, solved values, physical applicability, search completeness, and reference grading. A typed expression layer would help substantially, but it would not repair every category.

My recommendation for the **very next milestone is reference-verification integrity**, followed by a bounded, engine-wide dimensional-semantics repair. Confirmed silent numerical and physical errors should receive explicit repair or refusal decisions rather than wait indefinitely behind that architectural work.

I also found an additional source-level grading defect during this assessment: the harness can treat a symbolic comparison’s `(False, "counterexample")` return value as truthy. This is separate from the original 37 findings and strengthens the case for repairing verification first. Details appear below. ([harness.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L35-L65); [scene.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py))

### Evidence boundary

The assessed project revision is **`7244bb6fa984c441d9fbcdcbf2954e99c44b5efb`**, which still matched `main` when checked. The builder’s 37 reproductions concern the earlier revision `4aeb3e42…`; they are historical execution evidence, not tests I reran on the assessment baseline. I inspected relevant pinned implementation paths, the finding and reproduction records, project documentation, released evaluation reports, and the architectural debate. I did not execute code, collect tests, modify either repository, or access reserved directories or their historical contents. This is not an exhaustive line-by-line correctness certification. ([Project baseline](https://github.com/physicsvigilante/sim_waves/commit/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb); [D1 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/19#issuecomment-7086572679))

---

## 1. Root-cause classification of all 37 reproduced findings

The headline count requires two qualifications.

First, **D1-1 is not one of the 37 reproduced-as-predicted findings**. On the builder’s pinned SymPy environment, it produced an ambiguity refusal rather than the predicted unique wrong answer. Its incomplete presentation of the alias family remains a diagnostic concern. Second, the findings are not statistically independent: several D4 and D5 entries expose overlapping grading mechanisms. Counting them is useful for tracking obligations, not estimating defect prevalence. ([D1 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/19#issuecomment-7086572679); [D4 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/22#issuecomment-7086573728); [D5 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/23#issuecomment-7086574000))

I assign each finding one primary repair category, with secondary causes where necessary:

| Code | Primary mechanism | Finding records |
|---|---|---:|
| **T** | Type, dimension, unit-representation, or conversion contract | 11 |
| **B** | Binding or constraint information applied at the wrong stage | 5 |
| **S** | Incomplete solution-set reasoning or an unjustified mathematical claim | 9 |
| **P** | Incorrect physical/observable applicability contract | 6 |
| **N** | Numerical algorithm invariant violated | 1 |
| **V** | Verification or reference-grading contract violated | 5 |
| | **Total** | **37** |

These categories are my architectural classification, not categories supplied by the issue authors.

### Initial error hunt: H17

The builder reported all five reproductions. ([H17 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/17#issuecomment-7086579516))

| Finding | Primary cause | Underlying failure—not merely its symptom |
|---|---|---|
| **H17-1** | S | Resonance enumeration stops after observing a local trend, without establishing that later modes cannot return to the admissible range. |
| **H17-2** | S | Principal inverse solutions are treated as sufficient for a periodic search; additional period branches are omitted. |
| **H17-3** | T | Recorded symbol dimensions do not participate consistently in fact and expression checking. |
| **H17-4** | B | Emission-start causality is decided before substituting an already-solved start time. |
| **H17-5** | P | “Field depends on position” is used as a prerequisite for energy, incorrectly excluding the valid zero-field limit after cancellation. |

### Solver findings: D1

Six findings reproduced as predicted; D1-1 is excluded from this table. ([D1 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/19#issuecomment-7086572679))

| Finding | Primary cause | Underlying failure |
|---|---|---|
| **D1-2** | S | A parameter-dependent coefficient is divided out without preserving its zero case, inventing uniqueness in a degenerate family. |
| **D1-3** | S | Integer membership that is not disproved is treated as established; a conditional harmonic family is reported unconditionally. |
| **D1-4** | S | Phase canonicalization is applied inconsistently with the actual period of a fractional-angle equation, introducing invalid candidates. |
| **D1-5** | B, secondary S | Field-domain constraints are applied after the uniqueness decision rather than during candidate feasibility. |
| **D1-6** | B | Phase extraction examines an unresolved amplitude sign instead of its solved value. |
| **D1-7** | S | Candidate generation is driven by equalities and fails to construct the discrete feasible set determined by inequalities and integrality. |

### Search findings: D2

All five reproduced. The support, extrema, and binding problems are distinct even though they occur in search-related services. ([D2 closing summary](https://github.com/physicsvigilante/sim_waves/issues/20#issuecomment-7086236426); [D2 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/20#issuecomment-7086573084))

| Finding | Primary cause | Underlying failure |
|---|---|---|
| **D2-1** | S | Extremum candidates omit finite nondifferentiable points; derivative roots alone are not a complete candidate set. |
| **D2-2** | S | The phase-order range is incomplete before inverse-root enumeration begins; checking returned roots cannot recover omitted orders. |
| **D2-3** | S | Failure to establish where an unspecified profile vanishes is converted into a definite support and width. |
| **D2-4** | B | A harmonic cutoff is validated before applying its solved integer value. |
| **D2-5** | B | Support analysis runs on a symbolic pulse scale even after that scale has been solved. |

### Physical-observable findings: D3

All six reproduced. These directly contradict any blanket claim that the physics implementation is already sound. ([D3 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/21#issuecomment-7086573436))

| Finding | Primary cause | Underlying failure |
|---|---|---|
| **D3-1** | P | The existence of an ordinary second derivative is inferred from second derivatives of open branches, ignoring a jump in the first derivative. |
| **D3-2** | P | A stationary-medium transit formula is applied to a moving medium, while endpoint sorting removes propagation direction. |
| **D3-3** | P | A phase query bypasses the causal existence condition enforced by another observation of the same ray. |
| **D3-4** | P | Unknown absolute phase is confused with unknown relative phase; shared-source correlation is lost. |
| **D3-5** | P | A frequency-discontinuity guard is reused for a continuous position query. |
| **D3-6** | N | Arrival-time bisection updates the wrong endpoint and lacks a decisive postcondition check on the defining equation. |

### Units, dimensions, and output: D4

All eleven reproduced. This cluster contains several mechanisms—not eleven instances of one missing dictionary lookup. ([D4 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/22#issuecomment-7086573728))

| Finding | Primary cause | Underlying failure |
|---|---|---|
| **D4-1** | T | A permissively checked relation is treated as evidence for the requested output unit; lack of contradiction becomes unit inference. |
| **D4-2** | T | Substitution checks depend on the lowered expression’s shape rather than declared quantity identity. |
| **D4-3** | T | A trailing field unit validates the outer quantity while internal dimensional obligations go unchecked. |
| **D4-4** | T | Range-argument contracts are incomplete and differ between positional and keyword syntax. |
| **D4-5** | T | Affine temperature semantics are attached to a particular spelling rather than a canonical unit representation. |
| **D4-6** | T | Algebraic equality is checked without first establishing compatible physical quantities. |
| **D4-7** | T | A text-level angular-frequency guard substitutes for quantity-kind propagation and rejects valid conversions. |
| **D4-8** | T | Coordinate and operator semantics do not propagate result dimensions through frequency extraction and differentiation. |
| **D4-9** | T | Output conversion uses a different unit grammar from the shared unit parser. |
| **D4-10** | V | Reference parsing loses dimensional and affine-conversion information. |
| **D4-11** | V | A tolerance floor depends on numerical display scale, so grading changes with the chosen unit. |

### Verification and documentation: D5

All four reproduced. D5-1 overlaps the tolerance mechanism in D4-11; D5-2 and D5-3 overlap the reference-conversion mechanisms in D4-10. Preserve every witness, but do not count these as separate architectural capabilities. ([D5 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/23#issuecomment-7086574000))

| Finding | Primary cause | Underlying failure |
|---|---|---|
| **D5-1** | V | Percentage tolerance acquires an unintended absolute floor and accepts a grossly wrong small value. |
| **D5-2** | V | Reference dimensions are discarded, allowing a frequency to match a length reference. |
| **D5-3** | V | Celsius reference conversion applies an inconsistent offset transformation. |
| **D5-4** | T | Collection element dimensions are absent from the result contract and are not preserved through selection or conversion. |

**Conclusion from the mapping:** dimensional semantics, binding order, and solution-set reasoning account for **25 of the 37 records under this classification**. They dominate the backlog, but the remaining physical, numerical, and verification defects require their own contracts.

---

## 2. The deeper architectural diagnosis

### 2.1 The central problem is premature interpretation

The engine often reduces a richer object to a simpler one before the information needed to justify that reduction has been established.

Examples include a physical quantity becoming a bare SI number, a conditional solution becoming a single binding, a partially explored set becoming a list, and a comparison result becoming a Boolean. The current parser evaluates expressions before its separate dimension pass; the solver filters some obligations before uniqueness and others afterward; observations perform another round of parsing, binding, and deferred evaluation. These boundaries are visible in `expr.py`, `solver.py`, and `run.py`. ([expr.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py); [solver.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/solver.py); [run.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L588-L668))

**Architectural inference:** “metadata loss” is therefore only part of the diagnosis. Sometimes the metadata still exists, but its authority, scope, timing, or required strength is undefined.

### 2.2 “Unknown” has several incompatible meanings

The legacy dimension walker uses `None` for unrecorded dimensions, free-coordinate conventions, and bare zero. Its addition rule can return a known dimension after encountering an unknown operand. Those are not equivalent situations, yet callers receive the same representation. ([expr.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L516-L583))

The repair is not merely:

```text
Replace every None with an error.
```

It is:

> Distinguish missing information, contextual zero, explicitly qualified numerical coordinates, and an unresolved compatibility obligation.

A requirement is satisfied only when the relevant relationship is established. But an unknown dimension can legitimately be *inferred* from consistent constraints; it need not always cause refusal.

### 2.3 Validation is not consistently compositional

A function’s output dimension does not certify its inputs. A returned candidate satisfying the equations does not certify completeness. A sampled agreement does not certify invariance.

This is the common logical structure behind much of the backlog, but calling everything “unjustified certainty” is too broad to guide implementation. The engineering task is to identify the exact obligation at each boundary: dimensional compatibility, branch coverage, domain membership, causal existence, or numerical enclosure.

SymPy itself offers distinct representations for solved and unresolved solution sets, including `ConditionSet`. That supports the design principle, but replacing `solve` with `solveset` indiscriminately would not qualify the engine’s surrounding transformations or physical assumptions. ([docs.sympy.org](https://docs.sympy.org/latest/modules/solvers/solveset.html))

### 2.4 There is an overlooked state-management problem

The debate eventually exposed a mutable harmonic-role register being considered as dimensional evidence. Expression evaluation can modify such state before a subsequent validation fails. The source confirms the relevant ordering: evaluation precedes dimensional checking, and harmonic validation records symbolic uses. ([scene.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2136-L2151); [expr.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py))

**Recommendation:** observation checking should be transactional. A rejected observation must not establish facts or types for another observation. Results should depend on the checked scene and query, not query order.

### 2.5 The verification boundary is itself unsafe

The harness shares production parsing and equivalence machinery, discards expected dimensions, and applies scale-sensitive tolerances. That creates common-mode failures: the system checking a result can make the same semantic mistake as the system producing it. ([harness.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L20-L65))

This is why a larger regression suite alone is not the answer.

---

## 3. An additional source-level verification defect

This finding is **not included in the original 37**.

At the assessed revision:

- `scene.equivalent(a, b)` returns **`(truth, basis)`**.
- The symbolic branch of `harness.check` returns that tuple as its first result.
- `_run_file_local` assigns the first result to `ok` and tests its truth value.

Consequently, a result such as:

```text
(False, "counterexample")
```

is a nonempty tuple and therefore truthy. The same problem applies to an unresolved equivalence result packaged in a nonempty tuple. The relevant paths are `harness.py:35–65,125–131` and `scene.py`’s `equivalent`. ([harness.py:35–65](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L35-L65); [harness.py:125–131](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L125-L131); [scene.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py)) Python’s documented truth-value rules confirm that a nonempty tuple is not made false by containing `False`. ([docs.python.org](https://docs.python.org/3.10/library/stdtypes.html?highlight=atomic))

An independent mathematical witness is simply:

$$
A>0,\qquad \text{actual}=A,\qquad \text{reference}=2A.
$$

These cannot be equal. Yet the inspected symbolic grading path can classify the comparison as successful because it tests the tuple rather than its truth component.

**Evidence status:** this is a source-grounded interface defect, not a captured engine run. It needs a reproduction and regression test in the pinned environment before anyone credits a correction.

It also illustrates a limitation of the earlier debate: **not every important defect requires a sophisticated semantic architecture. Some require an ordinary, explicit return type and a correctly checked interface.**

---

## 4. Is the physics foundation sound?

**Useful and worth retaining: yes. Established as broadly sound: no.**

The source contains reusable first-principles constructions: string energy densities, impedance-based reflection, modal boundary formulas, and retarded-time propagation. Those are strong foundations. But physical correctness includes applicability, causality, correlations, and regularity—not just whether a familiar equation appears in the code. ([energy.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/energy.py); [scene.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2035-L2174); [doppler.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/doppler.py))

Three independent checks make the distinction clear.

**Moving-medium transit.** For ground-fixed endpoints separated by $d$, sound speed $c$ relative to the medium, and collinear wind $w$,

$$
t_{\rightarrow}=\frac{d}{c+w},\qquad
t_{\leftarrow}=\frac{d}{c-w}.
$$

At $d=360\text{ m}$, $c=340\text{ m/s}$, $w=20\text{ m/s}$, the answers are $1\text{ s}$ and $9/8\text{ s}$. Returning the same $18/17\text{ s}$ both ways is a physical-model error, not merely a unit error. The builder confirmed that behavior. ([D3-2](https://github.com/physicsvigilante/sim_waves/issues/21#issuecomment-7086154754); [D3 reproduction report](https://github.com/physicsvigilante/sim_waves/issues/21#issuecomment-7086573436))

**A triangular pulse’s cusp.** Locally,

$$
y(t)=A-b|t-t_*|.
$$

Velocity jumps from $+b$ to $-b$; ordinary finite acceleration does not exist at $t_*$. Differentiating open linear branches twice gives zero but does not establish the derivative at the join. ([D3-1](https://github.com/physicsvigilante/sim_waves/issues/21#issuecomment-7086144838))

**Zero-field energy.** If two permitted fields cancel exactly,

$$
y=0,\qquad
\mathcal E=\frac12\mu y_t^2+\frac12T y_x^2=0.
$$

The formula remains valid. Refusal merely because the simplified field no longer depends on $x$ is an applicability error in the implementation. ([energy.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/energy.py))

The released evaluations also support caution. Run 7 reported 34/44 full-coverage cases, but separately reported 20 proved, 18 sampled, and six unverified results; educational usefulness was not assessed. Those aggregates do not establish a 77% fully proved end-to-end success rate. ([Run 7 scorecard](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/run7/SCORECARD.md))

**Revised diagnosis:** the project has valuable physical models with incomplete semantic and applicability contracts. “The physics is sound” is too optimistic as a project-wide statement.

---

## 5. Critique of Astra, Opus, and the S experiment

### What both participants got right

The debate correctly moved away from a wholesale rewrite, a universal wave representation, and a single ordered result-basis label. It also recognized that exact specializations are useful, domain obligations must survive simplification, and physical composition must be demonstrated by new combinations rather than common function names. The later packets preserved the two-stage process and made increasingly explicit scope and compatibility commitments. ([Opus — Round 3](https://github.com/physicsvigilante/debates_simwaves/blob/810a219e14ca96c9d558ed0c9e26ee31d8a21a0b/debates/opus-round-3.md); [Opus — Round 7](https://github.com/physicsvigilante/debates_simwaves/blob/283f5cb9bfbd1336c5f08205ecd63558730433f1/debates/opus-round-7-checkpoint-s-packet-rev1.md); [Opus — Round 9](https://github.com/physicsvigilante/debates_simwaves/blob/e222d1a606578675f81fd7e3fcd2cf4b424452bf/debates/opus-round-9-checkpoint-s-packet-rev2.md))

### Where Opus’s reasoning overreached

Opus initially grouped too much under two causes and expected a typed record plus waveguide network to provide broad architectural leverage. The mapping above shows why that is incomplete: Celsius normalization, tolerance semantics, bisection invariants, causal phase, and same-source correlation are not interchangeable metadata problems.

The S packets then treated a returned dimension tuple—and later an existing numeric-validation register—as stronger evidence than those mechanisms actually provided. The late specification still equated “the walker returned a type” with “the expression’s obligations were checked.” ([Opus — Round 9](https://github.com/physicsvigilante/debates_simwaves/blob/e222d1a606578675f81fd7e3fcd2cf4b424452bf/debates/opus-round-9-checkpoint-s-packet-rev2.md); [Astra — Round 10](https://github.com/physicsvigilante/debates_simwaves/blob/6d3d93b6ff5efb9e9b4e9ce4161aae4e4994903c/debates/astra-round-10-checkpoint-s-rev2-review.md))

### Where Astra’s reasoning also needs correction

Astra’s small-slice strategy was reasonable as an experiment, but the advice to reuse the dimension walker with improved lookup was initially insufficient. More importantly, the debate spent too long adjusting an imposed file/line footprint after the required semantic boundary had clearly outgrown it.

Astra also overstated two conclusions in earlier summaries:

**“A local patch cannot be sound” is not established.** A closed, recursively checked local language could be sound. The rejected packets did not implement such a language; they depended on permissive legacy operations. A simpler local repair of list typing or solved-cutoff admission was also possible, although it would test little of the larger architecture.

**“Most physics is sound” is not supported by the defect sample alone.** The hunt was targeted, not random, and several confirmed defects are genuinely physical.

### Was shelving S correct?

**Yes, as a prioritization decision—not as an impossibility theorem.**

S had become a poor experiment: it tried to demonstrate semantic composition while forbidding changes to the part responsible for semantic composition. Continuing would require either a second local checker or a sharply restricted language whose coverage cost might outweigh the pilot’s value.

The useful lesson is:

> Scope should be bounded by an invariant and its dependency closure, not by a preselected list of files that must remain untouched.

---

## 6. The smallest adequate semantic architecture

I recommend a **small checked front end over the existing physical models and SymPy backend**, not a new computer algebra system.

Three records are sufficient as a starting design:

| Record | Minimum responsibility |
|---|---|
| **Expression node** | Operation, operands, source/law origin, scalar/list shape, quantity type, and obligations created by that operation. |
| **Checked environment** | Stable symbol identities, declarations, dimensional constraints, domains, and checked solved bindings. |
| **Result** | Value/set/enclosure plus the specific claims established about that query and their assumptions. |

A single operation registry should connect an operation’s argument contract, result contract, applicability requirements, and evaluator. It should not contain a general theorem prover.

### Dimensions should be constraints, not wildcards

For the admitted arithmetic language, dimension exponents can be handled by small linear constraints:

$$
D(ab)=D(a)+D(b),\quad
D(a/b)=D(a)-D(b),\quad
D(a+b): D(a)=D(b).
$$

Thus $D(u/u)=0$, even when $D(u)$ is unknown. The separate value-domain obligation $u\ne0$ must remain.

This establishes dimensional relationships without inventing a physical parameter value. A supplied expression with insufficient or contradictory constraints is refused; the requested display unit must not become evidence for the answer’s dimension.

### Preserve textbook coordinates through an explicit interpretation

The implicit-SI convention should be modeled as a numerical-profile context, for example

$$
X=\frac{x}{1\text{ m}},\qquad
\tau=\frac{t}{1\text{ s}},\qquad
y=(1\text{ m})F(X,\tau).
$$

That preserves ordinary textbook coefficients while explaining derivative scaling. It is safer than globally treating physical $x$ and $t$ as untyped. The current format’s convention is real and must be migrated deliberately, not accidentally broken. ([FORMAT.md](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md))

### Separate three-valued decisions from computation outcomes

For a predicate, use **established**, **disproved**, or **unresolved**. Separately distinguish unsupported input, resource exhaustion, and internal failure.

Claims also need subjects:

- `domain_valid(expression, region)`
- `complete(solution_set, region)`
- `unique_value(query)`
- `unique_state(model)`
- `encloses(value, interval)`

A unique frequency does not require a unique amplitude or phase. A unique maximum value does not imply a unique maximizing location.

The current solver explicitly aims for one complete situation. Moving toward query-specific determinacy could improve coverage, but it would be a deliberate later contract change—not an incidental consequence of the unit repair. ([solver.py](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/solver.py))

### Obligations must survive every transformation

Simplification preserves authored exclusions. Substitution changes values and their obligations together. Differentiation introduces regularity requirements. A range query quantifies obligations over the whole region. A search result needs both candidate validity and a coverage argument.

Exactness, completeness, numerical accuracy, and independence are not one hierarchy. For example, an exact expression may come from an incomplete search, while a certified interval may support a reliable inequality without yielding a closed form.

---

## 7. Alternatives and the waveguide proposal

### Architectural alternatives

| Approach | Advantage | Principal risk | Assessment |
|---|---|---|---|
| Continue local fixes | Fast for genuinely isolated defects | Repeated semantic omissions and duplicated guards | Use for isolated algorithmic faults, not the dimensional core |
| Restricted front-end validator | Potentially small and auditable | Becomes a second authority unless it owns the full admitted syntax | Viable only with a clearly closed language |
| Checked expression/contracts layer over SymPy | Preserves existing physics and supports systematic composition | Migration and temporary dual-path complexity | **Preferred** |
| General equation-modeling framework | Strong component and connection abstractions | Much larger modeling/solver scope than needed | Borrow ideas; do not adopt wholesale |
| Numeric-first engine | Useful exploratory simulation | Does not solve unit, causality, completeness, or verification defects | Complement, not replacement |
| Full rewrite | Freedom to redesign | Discards useful behavior before a replacement is qualified | Not justified by present evidence |

### Independent assessment of waveguides

The waveguide idea is mathematically credible for **stationary, linear, uniform one-dimensional segments with qualified junctions and terminations**.

For a string,

$$
J=-Ty_x y_t=Fv,
$$

so force and velocity form a power pair. For a duct, the corresponding power pair is pressure and volume velocity, $p'U$. Their adapters are not interchangeable: $U=S\xi_t$, so displacement conversion includes area.

Equation-based connection assembly is a useful precedent, but does not require numerical time integration; connection equations and solver choice are separate design decisions. ([specification.modelica.org](https://specification.modelica.org/maint/3.6/connectors-and-connections.html))

The composite-string witness is a meaningful composition test. With equal segment lengths, equal tension, and $\mu_2=4\mu_1$, the undivided fixed-fixed boundary determinant becomes

$$
2k_1\sin\theta(3\cos^2\theta-1)=0.
$$

It preserves both joint-node and non-node families. A formula selecting joint-node modes alone would miss the actual fundamental. The debate’s subsequent corrections to mixed endpoints and zero-frequency handling were necessary. ([Astra — Round 4](https://github.com/physicsvigilante/debates_simwaves/blob/efecda1ca69b3bd7004d63602ff594d383ae21b8/debates/astra-round-4.md))

However, **architectural plausibility is not yet a demonstrated payoff**. The decisive tests are whether the shared assembly:

1. becomes the actual authority used by existing consumers;
2. handles a new admitted composition without production changes;
3. reduces duplicated applicability logic without destroying useful symbolic specializations.

Pressure and particle velocity derived from a selected pipe mode may deliver useful value before general composite spectra. I would not make certified transcendental root isolation a prerequisite for that smaller physical improvement.

Also retain the distinction between first arrival and steady state. A transmitted first pulse is not the infinite-reflection harmonic response. A resonator eigenmode is not a driven response. A shared network does not erase those differences.

---

## 8. Priorities and a bounded next milestone

### Recommended roadmap

| Order | Work | Reason |
|---|---|---|
| **1** | **Reference-verification integrity** | Establish trustworthy evidence before measuring a core migration |
| **2** | **Dimensional semantics and checked binding interfaces** | Largest coherent cross-cutting repair opportunity |
| **3** | **Qualified solution-set and search contracts** | Prevent missing branches, invented support, and false uniqueness |
| **4** | **Remaining observable applicability repairs** | Causality, frame handling, coherence, regularity, and valid zero limits |
| **5** | **Physical composition experiments** | Revisit pipe fields and qualified waveguide assembly |
| **6** | **Broader coverage and educational evaluation** | Measure actual benefit rather than infer it from architecture |

This is not a strictly serial permission to leave dangerous known defects active. D3-6’s broken bisection and other accepted wrong-answer paths deserve separately bounded correction or refusal decisions. They do not need to wait for an ambitious semantic framework.

### Why verification comes before the units milestone

The dimension repair has greater direct capability impact, but it also has a wider regression surface. Starting it while the harness can accept incompatible references, unit-dependent tolerances, and false symbolic comparisons makes every apparent success harder to interpret.

A verified comparator does not prove the physics. It does establish that the evidence reported by the harness means what it says.

### Proposed next milestone: **Reference Contract Integrity**

**Objective:** make reference comparison honest, typed where required, unit-invariant, and explicit about unresolved cases.

**In scope:** expected-value parsing, scalar/list comparison, quantity conversion at the reference boundary, tolerance semantics, symbolic comparison status, and expected-error matching. Include the D4-10/11 and D5-1/2/3 witnesses and the additional tuple-truthiness trace.

**Out of scope:** changing physical formulas, global dimensional inference, new searches, waveguide assembly, arbitrary-parameter proofs, or upgrading SymPy merely to obtain different answers.

#### Declared supported class

Initially support finite real scalar results, Booleans, finite homogeneous tuples, exact integer/rational references, and a bounded symbolic equality class such as polynomial or rational-function identities with retained domain conditions. Approximate comparisons need explicit tolerances.

Dimensioned comparisons require a trustworthy recorded dimension or an explicitly declared result contract. If the production path cannot supply that, return **unverified/uncheckable**, rather than infer the dimension from the expected unit.

This may reduce apparent pass counts. Such reductions must be reported as previously unverified coverage, not concealed by fallback acceptance.

#### Required guarantees

**Typed comparison.** A Boolean cannot silently become a number; a list cannot become a scalar; incompatible physical dimensions cannot compare successfully.

**Canonical conversion.** Compare in a canonical physical representation. Absolute temperatures and temperature differences need distinct affine rules.

**Explicit tolerance semantics.** For ordinary scalar quantities, use a documented rule such as

$$
|g-w|\le a_{\mathrm{tol}}+r_{\mathrm{tol}}|w|,
$$

where absolute tolerance has the quantity’s dimension. There must be no hidden `max(1, |w|)` floor in display units.

**Explicit proof status.** A symbolic comparison returns a structured decision, never a tuple implicitly used as a Boolean. Sampled agreement is not a proved identity.

**Honest outcome naming.** A numerical reference match means the represented value matches the reference under the tolerance. It does not certify the numerical method’s error bound or establish physical correctness.

**Specific negative tests.** An expected refusal must match the intended observation and error category. An unrelated internal failure must not make a negative test pass.

### Independent acceptance criteria and oracles

These are public development criteria, not reserved-test specifications.

| Acceptance property | Independent oracle |
|---|---|
| Unit invariance | $1\text{ m}=100\text{ cm}$; equivalent presentations receive identical grades |
| Incompatible units rejected | $1\text{ m}$ cannot match $1\text{ s}$, even when scalar values coincide |
| Temperature conversion | $20^\circ\text{C}=293.15\text{ K}$; a 20 K difference receives no Celsius offset |
| Small-value tolerance | $10^{-12}$ versus $10^{-10}$ must not match under a 1% relative tolerance |
| Symbolic mismatch | $A$ versus $2A$, $A>0$, must fail |
| Symbolic identity | $A+A$ versus $2A$ can pass by exact polynomial identity |
| Domains preserved | Cancellation of $(x-1)/(x-1)$ does not erase its exclusion at $x=1$ |
| Collection integrity | Length, order, element types, and empty-list behavior are checked explicitly |
| Failure integrity | Internal errors, resource limits, and physical refusals remain distinct |

The tests should intentionally mutate otherwise correct references—wrong sign, unit, scale, offset, element, and symbolic coefficient—and verify rejection. Expected outcomes should be computed independently of the production comparator.

Follow the existing charter: owner-approved PLAN, separate reserved-problem author, stage-1 review, separately approved freeze, fresh-author blind stage, and stage-2 update of the same PR. No step is authorized by this assessment. ([CLAUDE.md](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/CLAUDE.md#L17-L37))

---

## 9. Reducing review cycles without lowering correctness

The long cycles are **not inevitable consequences of symbolic physics**, but neither are they evidence that every patch created a new bug. Some reviews uncovered pre-existing defects, some widened the examined class, and some exposed integration failures.

The history shows repetition well before m14: solved-value handling, dimension checks, cancellation, and range guards were repaired in earlier milestones and later failed through another path. Conversely, m15a’s repeated process-creation/cancellation defects concern a different state-machine invariant. ([FORMAT findings history](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FINDINGS.md); [m15a result](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/m15a_containment/RESULT.md))

I recommend three changes.

**Review invariants before reviewing examples.** For each milestone, identify the invariant, all producers and consumers that rely on it, and what constitutes proof or a justified refusal. A list of edited files is not a dependency analysis.

**Test transformations and compositions systematically.** Alongside specific regressions, qualify unit rescaling, substitution before versus after binding, observation reordering, equivalent syntax, zero/degenerate limits, and nesting through admitted functions. These tests should come from the declared grammar, not only from the latest counterexample.

**Keep separate ledgers for defects, scope, and evidence.** Track confirmed wrong answers, justified new refusals and their coverage cost, and claims supported only by sampling or static inspection. A new PASS count must not erase an unresolved mechanism.

For search work, require explicit candidate coverage. For numerical routines, require algorithm-specific invariants. For bisection, that means preserving the sign bracket and returning a qualified interval or error—not merely a plausible decimal. A residual check is valuable, but without conditioning information it is not generally an error bound.

---

## 10. Strongest objections to this recommendation

**Verification-first delays direct repairs.** That is true. The response is to keep it tightly bounded and make immediate risk decisions for known wrong-answer paths—not to turn the verification milestone into a complete new semantics engine.

**A typed front end may become another complicated framework.** Also true. It earns its place only if it removes duplicate semantic authorities and supports a clear admitted language. A large ontology, universal prover, or elaborate plugin architecture is unnecessary.

**Strict checking may reduce usable coverage.** Yes. Some previously accepted answers were never established. But the project should also distinguish unavoidable uncertainty from an unnecessarily restrictive implementation; valid-neighbor controls and explicit coverage reporting are essential.

**The waveguide work may have higher educational payoff.** It might. The current reports do not establish that payoff quantitatively, and educational usefulness remains largely unmeasured. That is a reason for a later bounded comparison of alternatives, not automatic rejection of waveguides. ([Evaluation scorecard](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/SCORECARD.md); [Run 7 scorecard](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/run7/SCORECARD.md))

**The defect corpus is biased.** Absolutely. It supports architectural hypotheses about observed failure mechanisms, not estimates of the probability that a random JEE scene fails. Neither 37 findings nor a successful small transfer set supplies that probability.

---

## Final recommendation

**Retain:** the existing physical models where independently supported, exact algebra, source-fact provenance, domain preservation before simplification, containment, and blind-transfer safeguards.

**Replace:** permissive wildcard dimensional reasoning, implicit truthiness of verification results, fragmented applicability checks, and search interfaces that return lists without establishing their coverage.

**Simplify:** the next step. Do not combine reference grading, global units, all solver branches, arbitrary-value invariance, and waveguide composition into one milestone.

**Defer:** the S packets as written, a general waveguide network, broad certified numerical services, and any claim that the architecture alone closes the backlog.

The most defensible sequence is:

> **Make verification trustworthy; establish checked dimensional and binding contracts; qualify solution-set reasoning; then demonstrate new physical composition.**

The important correction to the earlier discussion is that the project does not need one grand abstraction to repair all 37 findings. It needs a small number of explicit, independently testable contracts at the boundaries where it currently turns incomplete information into an accepted answer.
