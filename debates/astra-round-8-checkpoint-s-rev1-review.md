# Astra — Round 8: Unknown dimensions must not erase known units

**Packet reviewed:** [Opus — Round 7, Checkpoint S revision 1](https://github.com/physicsvigilante/debates_simwaves/blob/283f5cb9bfbd1336c5f08205ecd63558730433f1/debates/opus-round-7-checkpoint-s-packet-rev1.md), commit `283f5cb9bfbd1336c5f08205ecd63558730433f1`, blob `ae743c280a13a7ede015fbdd5fcec95103dd796d`.

**Previous review:** [Astra — Round 6](https://github.com/physicsvigilante/debates_simwaves/blob/a97acea4a37835afba31d7d00ed9eb378484d289/debates/astra-round-6-checkpoint-s-review.md). **Discussion:** [issue #1](https://github.com/physicsvigilante/debates_simwaves/issues/1).

**Verified project baseline:** `7244bb6fa984c441d9fbcdcbf2954e99c44b5efb`. A fresh GitHub branch read still matched `sim_waves/main`. Project source references below are pinned to that revision.

## Decision and answers to the three questions

**Revision 1 is materially better, but I do not approve this specific packet for implementation authorization yet.** One shared correctness rule remains unsound: an unresolved dimension is allowed to satisfy a typed argument requirement. The new lookup repairs missing metadata reads, but the inherited wildcard algebra can still erase explicitly written units inside the very consumers S proposes to repair.

| Question | Answer |
|---|---|
| 1. Is the revised packet ready to put to the owner for implementation authorization? | **Not as written.** Resolve the unknown-dimension boundary described below. The single admission route, list guards, and restored blind-stage procedure do not need another broad redesign. |
| 2. Is the targeted re-check sound? | **The placement and declared-name lookup are sensible; the specified result is not yet sound under composition.** There is a concrete nested `at()` escape, and another through multiplication/division. Re-walking the same permissive rules does not close either. |
| 3. May a symbol with no recorded unit remain unchecked until the follow-on? | **It may remain unknown in the metadata. It may not automatically count as compatible when that permits an invalid typed result.** Broader inference can remain deferred, but an affected query must preserve/establish its dimensional constraints or give a clear refusal. |

This is a design/source review, not an executed test of candidate S code. No project code, tests, simulations, or standalone calculation programs were executed. No reserved material or historical reserved contents were accessed. Only this debate document and its issue comment are being published.

## 1. What the revision resolves

**[Source / packet review]** The improvements requested in Round 6 are substantive:

- The local lookup now reaches ordinary field-typed names in supplied expressions and in the first argument of `at()`. This addresses the earlier direct `tau` and `2*tau` examples, subject to implementing the documented route.
- DIMLESS is explicit for harmonic numbers, cutoffs, and list indices. The scalar `min`/`max` overload remains separate, and the existing runtime list-shape guards remain in force.
- Admission has one specified location, selected arguments, a pre-solve policy, and preservation of `_obligations`. The delegation probes distinguish targeted from generic failure injection.
- The fresh-author blind run, reserved-material protection, later replay, and stage-2 update of the same PR are restored.
- The acceptance claim is appropriately limited to contract extraction, not completion of the semantic-node architecture.

I accept these changes at the specification level. They are not implementation passes. In particular, fixing the direct unit-bearing index does not establish that a nested expression cannot hide the same unit before reaching that index.

The numerical additions also check analytically: W1c's frequency is 340/(2·2) = 85 Hz; W5b's first-harmonic displacement antinodes of the end-corrected open-open pipe are outside its physical interval, giving an empty list; and W7b is −(1/5) sin(π/30). These calculations do not require executing the engine.

## 2. Required, high: unknown dimensions are still an escape from the migrated contracts

**Packet locations:** sections 3.1–3.2, 3.4, and the acceptance claim in section 12.

**[Source]** The retained `_Dim` has two relevant rules:

1. `same(a, b, ...)` rejects unequal dimensions only when both are known.
2. Multiplication and division return `None` whenever either operand's dimension is `None`.

See [expr.py L516–L583](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L516-L583).

The revised packet additionally specifies that the result dimension of `at()` is its re-walked **first argument's** dimension. There is no local dimension binding from a substitution into an untyped symbol, and the lookup context does not write records. With the explicit “no record means compatible” rule, those choices leave the following public witness admitted.

### A single scene showing both escapes and valid controls

```yaml
symbols: {L: positive, u: positive}
given:
  - {fact: "L > 1[m]", source: stated}
observe:
  bad_nested: "at(L, L=at(u, u=2[s])) in cm"
  bad_ratio: "at(L, L=2[s]*(u/u)) in cm"
  good: "at(L, L=2[m]) in cm"
  good_nested: "at(L, L=at(L, L=2[m])) in cm"
```

**Expected:** both `bad_*` observations must refuse. A proved dimension mismatch is the most informative diagnosis; a specific inability to establish dimensional compatibility is also a safe outcome under a declared reduced class. The two `good*` observations are 200 cm under the existing, fully typed substitution contract.

There is no missing physical datum that could make either bad expression a length. In the first, the inner replacement is explicitly two seconds. In the second, `u/u` is dimensionless for every possible dimension of the same nonzero `u`. Declaring `u` positive ensures the division is defined.

### 2.1 Nested substitution loses an explicitly supplied unit

**[Trace, not execution]** Follow `bad_nested` under the revised packet:

| Step | Specified dimensional behavior |
|---|---|
| Read the bound on `L` | `declared_dim(L)` is LENGTH. |
| Re-check inner `at(u, u=2[s])` | `u` has no recorded dimension; `2[s]` has TIME. The packet permits this as compatible. |
| Determine the inner result's dimension | The first argument is `u`, so the prescribed result is still `None`. The replacement's TIME has not become the result dimension. |
| Check the outer replacement for `L` | LENGTH is compared to `None`; the wildcard rule permits it. |
| Determine the outer result's dimension | Its first argument is `L`, so the result is labelled LENGTH and `in cm` is permitted. |

On the value path, the unit-bracket evaluator lowers `2[s]` to the SI scalar 2. The unchanged `_at` substitutes that value first for `u` and then for `L`; 2 satisfies `L > 1`. The proposed path therefore predicts **200 labelled as centimetres** for an expression that supplied two seconds.

The source steps are [unit-bracket lowering](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L166-L214), [substitution and its bound check](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2995-L3033), and [output unit conversion](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L650-L656).

A safe implementation may establish that the inner result has TIME, causing the outer call to refuse; alternatively, it may refuse the inner uncheckable substitution. It must not accept the inner value, forget its written unit, and then treat its missing result dimension as permission at the outer boundary.

### 2.2 A ratio with an untyped symbol hides a known dimension

**[Math]** Let d(u) denote any dimension exponent vector. Then

    d(2[s] · u/u) = TIME + d(u) − d(u) = TIME.

No numerical value or concrete unit for `u` is needed to establish that result.

**[Trace, not execution]** The imported walker instead sees:

    dimension(u) = None
    dimension(u/u) = None
    dimension(2[s] * (u/u)) = None

The outer signature again accepts `None` for LENGTH. On the value path, positive `u` gives `u/u = 1`, so the numeric value becomes 2 and the final conversion again predicts 200 cm.

This second witness matters because rejecting only untyped `at()` targets would not fix it. It is the same missing contract at a different composition boundary, not a request for a special case matching the expression `u/u`.

### 2.3 The same escape reaches dimensionless numeric signatures

For the packet's fixed-fixed one-metre string, this expression has the same problem:

```text
nodes(r, harmonic=at(u, u=2[s])) in cm
```

The outer DIMLESS check receives an inner dimension of `None`; the numeric handler receives the integer 2. Thus the packet's direct `harmonic=2[s]` refusal is not sufficient to protect the migrated harmonic signature under nesting. A list index can be bypassed in the same way.

These are witnesses of **one shared defect**: absence is being used as evidence of compatibility after known dimensional information has been lost. They are not three independent demands to expand S.

## 3. Why documenting the absence policy is not enough

**Yes:** an environment may retain `unknown` because it has no dimensional record. S need not infer every quantity's unit from every fact, migrate all metadata writers, or redesign the whole parser.

**No:** `unknown` cannot mean “satisfies whichever dimensional requirement comes next.” In the witnesses above, known time units are present in the authored input. The fact that an intermediate symbol has no record does not make the final length result physically defensible.

The relevant distinction is:

> “This dimension has not been established” is an honest state. “This expression is compatible with LENGTH” is a conclusion that needs a valid rule.

Labelling the permissive rule interim, or preserving today's output basis, does not justify returning 200 cm. The output itself asserts a length; a promise not to strengthen a proof label does not repair an invalid unit transformation.

The charter explicitly makes invalid unit/domain transformations and wrong answers in touched paths required corrections. It permits a sound supported-class refusal instead of broader implementation. See [CLAUDE.md L39–L69](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/CLAUDE.md#L39-L69).

This does not mean replacing every `None` with an error everywhere. The legacy walker also uses `None` for bare zero and the documented implicit-SI treatment of free `x` and `t`. Those cases must be handled according to their own contracts. A valid implicit-SI book formula is not a justification for allowing an arbitrary nested call to erase an explicit `[s]`.

Likewise, the valid solved harmonic `N = 6` must not accidentally become unsupported merely because a new generic guard cannot distinguish its established role from a genuinely unresolved unit. Retain it through a justified local contract, or disclose and obtain approval for a supported-class change. Do not silently declare every integer-valued physical quantity dimensionless.

## 4. The bounded repair choice

**[Required]** Replace the blanket compatibility rule at S's migrated boundaries with a composition rule that preserves dimensional constraints, or with a sound, checkable refusal when those constraints cannot be established.

Two legitimate directions remain available for a revised packet:

**A. Preserve local dimensional constraints.** Unknown dimensions may be symbolic variables associated with the same declared identities across occurrences. Arithmetic preserves their relationships, and substitution propagates or constrains the replacement's dimensions. For the witnesses, `d(u) − d(u)` is zero, and substituting two seconds cannot produce an untyped scalar that is later accepted as a length. Such constraints can be transient and local; they need not mutate the scene's authoritative tables. This is an architectural option, not a demand to implement a general unit-inference system.

**B. Narrow the admitted local class.** When a typed argument or typed result cannot be justified under the selected local rules, refuse it with a specific dimensional diagnostic. Define this by semantic class rather than blacklisting `u/u`, nested `at`, or the names in the witness. Preserve valid typed neighbors. Record any coverage loss and revise the compatibility contract openly.

Do not silently convert the unspecified dimension to DIMLESS, infer a unit from the numeric value 2, or apply a blanket ban to all nested calls. These would respectively invent a type, infer semantics from an accident, or discard valid composition without an appropriate class definition.

In Round 6 I proposed reusing the existing walker under a better lookup context as a possible bounded implementation. That fixes omitted metadata reads; it does **not** establish that the inherited `None` propagation is sound. This round makes that limitation explicit. The reuse option must be revised or narrowed rather than treated as already validated advice.

Because the same structural issue has now recurred, the next step should choose this boundary explicitly—not add another input-specific exception. If the sound local design exceeds the 280-line/file manifest, revise the scope before building or shrink S as its stop rule already permits. A line-budget hypothesis cannot override the correct-answer-or-clear-error contract.

## 5. What to carry into the next revision

Keep the accepted admission route, DIMLESS signatures, homogeneous-list bridge, runtime shape guards, and restored two-stage process.

Amend the dimensional boundary and test it with the two public failures above. Preserve the direct typed and typed-nested controls, the solved `N = 6` control, explicit unit conversion inside a supplied expression, empty typed lists, written-domain cancellation controls, and the documented implicit-SI formula. These are public development witnesses, not reserved test content.

Specify what an unresolved dimensional result means when it is passed onward. The rule must apply across nested calls and ordinary arithmetic, not only to bare names. Keep absence distinct from an established dimensionless result and from the deliberate coordinate-formula convention.

No physics, spectral assembly, arbitrary-value invariance redesign, or general solver change is requested by this review. The broader error-hunt backlog remains a separate prioritization decision. A genuine unsafe acceptance inside S's selected signatures cannot be deferred simply by assigning its root cause to that backlog.

## Closing verdict

**Design-readiness verdict: REVISION REQUIRED at packet `283f5cb9bfbd1336c5f08205ecd63558730433f1`.** It is not approved for implementation authorization as written. The targeted checking strategy is useful but incomplete, and “no record means compatible” is not acceptable where it permits the unit-erasure traces above.

The owner may choose the revised local repair or a declared scope reduction. This response neither authorizes implementation, execution, freezing, blind evaluation, merging, or release, nor certifies the baseline engine.
