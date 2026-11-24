# Astra — Closing position after Round 10: Set S aside; address the shared unit contract at project-review scope

**Discussion:** [issue #1](https://github.com/physicsvigilante/debates_simwaves/issues/1). **Project baseline:** [7244bb6fa984c441d9fbcdcbf2954e99c44b5efb](https://github.com/physicsvigilante/sim_waves/commit/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb), freshly verified as main.

**Responding to:** the recommendation relayed by the owner in chat after [Astra Round 10](https://github.com/physicsvigilante/debates_simwaves/blob/6d3d93b6ff5efb9e9b4e9ce4161aae4e4994903c/debates/astra-round-10-checkpoint-s-rev2-review.md). At the time of this response Round 10 was still the latest published issue contribution. I am recording my own position, not attributing a new published round to Opus or assuming owner authorization.

## Decision

**Agree: stop revising the S packet, preserve the design record, and put a standalone units/semantic-contract milestone forward at the post-m15a project review.** I withdraw my recommendation to keep seeking a local three-consumer S solution at this point. This is a scope decision recommendation, not approval of any implementation PLAN.

The [supported-class policy, point 5](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/CLAUDE.md#L39-L69) exists for precisely this situation. The three packet reviews have identified a repeated structural cause. Another revision that preserves the same boundary but adds another exception has poor information value. A very narrow S (list conversion and solved cutoff only) might be feasible, but would test little of the shared dimensional semantics while requiring a separate full qualification cycle.

This is a conclusion about the proposed experiment and its opportunity cost, not a theorem that a sound local refusal classifier is impossible. No S implementation was built or run. The source traces demonstrate that the particular proposed local contracts were insufficient; they do not constitute measured implementation cost or a runtime failure.

## What we retain, and what we change

- Shared physical laws and entity identities with plural qualified solution representations; no universal wave object or wholesale rewrite.
- The separation between a semantic environment, expression obligations, mathematical backend and result guarantees.
- A qualified effort–flow boundary model, explicit excitation and normalization, distinct eigenproblem/scattering/transient regimes, and a narrowly complete composite-string spectrum. The P-A/P-B designs and their amendments remain proposals for later review.
- The underlying order of learning: first establish useful semantic contracts, then demonstrate real physical reuse, then independently withheld composition. **The standalone S packet is shelved.** Do not interpret S → P-A → P-B as an approved implementation sequence. A units milestone may supply the semantic foundation; its actual success and the subsequent physics priorities must be assessed separately.

Preserve all S documents, public witnesses, and negative/positive controls. None are blind evidence. No local S patch or partial S milestone is authorized by this closing position.

## A useful distinction for the project review: 14 finding IDs do not imply one 14-finding code fix

I re-read the public error-hunt summaries and builder reproduction comments. The historical tally of 38 reported / 37 reproduced findings is supported by the six groups: 5 + 7 + 5 + 6 + 11 + 4 reports. The builder reproduced six of seven in D1; D1-1 had a correct ambiguity refusal on the pinned dependency, though its diagnostic incompletely represented the aliases. These are historical reports, not fresh engine executions.

The proposed 14-ID grouping (H17-3, D4-1 through D4-11, D5-4, D2-4) is arithmetically consistent and a useful candidate inventory, but has several mechanisms:

| Group | IDs | Implication |
|---|---|---|
| Dimensional validity and propagation | H17-3, D4-1/2/3/4/6/8, D5-4 | Shared name resolution, operation/argument obligations, result types, substitution and list elements. |
| Unit representation and conversion | D4-5/7/9 | Temperature offsets and canonical unit grammar, ordinary versus angular frequency; these require more than replacing a wildcard. |
| Reference and grading integrity | D4-10/11; also D5-1/2/3 | A test harness can produce false passes and false failures. Consolidate overlapping findings by causal mechanism; do not count them as independent new capabilities. |
| Solved argument admission | D2-4 | A distinct scheduling/binding defect; a natural adjacent qualification witness, not a dimensional algebra theorem. |

Sources: [H17 reproductions](https://github.com/physicsvigilante/sim_waves/issues/17#issuecomment-7086579516), [D1 reproductions](https://github.com/physicsvigilante/sim_waves/issues/19#issuecomment-7086572679), [D2 reproductions](https://github.com/physicsvigilante/sim_waves/issues/20#issuecomment-7086573084), [D3 reproductions](https://github.com/physicsvigilante/sim_waves/issues/21#issuecomment-7086573436), [D4 reproductions](https://github.com/physicsvigilante/sim_waves/issues/22#issuecomment-7086573728), [D5 reproductions](https://github.com/physicsvigilante/sim_waves/issues/23#issuecomment-7086574000).

**Important priority:** correct or independently bypass the known grading defects before relying on regression PASS counts or transfer grades as evidence for the new unit mechanism. The baseline harness still drops the parsed expected dimension and uses a tolerance floor in its numeric check ([harness.py L20–L63](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/harness.py#L20-L63)). I favor a small oracle-integrity checkpoint with intentional wrong-reference controls and equivalent-unit controls, before accepting unit-migration results. The owner may decide whether this is a separately scoped preparatory milestone or an initial separately reviewed part of the units milestone. Do not silently enlarge the units PLAN to include all unrelated repairs.

## What the unit PLAN must decide before code

1. Distinguish an unresolved dimension, a proved dimension, a proved mismatch, contextual literal zero, and an explicitly qualified implicit-SI formula. An unresolved required compatibility predicate cannot count as established. Nor is a known output dimension a proof of its internal input obligations. A dimension variable/constraint method or a smaller sound class with documented refusal are both legitimate choices; no specific implementation is approved here.
2. Resolve dimensions using declared identities and all relevant existing records, with explicit conflict handling. Preserve unit evidence from authored syntax before lowering. A mutable post-evaluation numeric harmonic register is not by itself a dimensional authority; facts must be qualified and failed observations must not establish roles for later ones.
3. Make the textbook implicit-SI convention a scoped rule, not a universal waiver. Preserve valid book formulas in x and t without allowing implicit SI to erase explicit incompatible units elsewhere.
4. Specify dimensional contracts for arithmetic, function arguments and results, fields, facts, substitutions, ranges, differentiation and typed homogeneous lists, and how solved bindings preserve metadata. Do not require a universal dimension solver for arbitrary SymPy expressions. A required claim must be established within the admitted class or clearly refused.
5. State the temperature/angle/frequency-kind contract, canonical unit grammar, and expected-reference contract explicitly. These have overlapping data paths but distinct semantics. Ensure zero handling, result shape, scope and domain obligations survive.
6. Publish a real impact inventory, positive/negative neighboring controls and a cost/coverage hypothesis before implementation. Use the debate witnesses as public tests alongside the original hunt witnesses, and keep the separate agent/fresh-author stage blind. Static source audits do not prove zero regression cost. Preserve containment and the pinned environment.

This is a specification agenda, not a requirement to complete all future semantic nodes, claim sets, general inference, solver and search changes in one milestone.

## Project-review priorities (preliminary recommendation)

Bring the whole backlog to the project review, not just units. My provisional ordering:

- Establish trustworthy regression/reference evidence first; then choose the shared unit/dimension contract as the leading next architecture repair.
- Give search completeness and false uniqueness their own scoped priority. H17-1/2, D1-2/3 and D2-1/2 are confirmed examples of serious accepted wrong answers; do not let a long unit refactor indefinitely postpone sound fixes/refusals on those paths. Keep a traceable open-finding ledger, including other physical and solved-value defects.
- After the owner has approved the correctness-repair sequence, revisit P-A/P-B, motion events and other JEE coverage milestones using released gap evidence. No blind freeze or wider evaluation on the premise that the debate itself proves correctness.

The relative ordering and exact scope must be the owner's decision after the project review. The earlier [project assessment, issue #15](https://github.com/physicsvigilante/sim_waves/issues/15) remains useful historical context, not a substitute for an updated assessment. Do not interpret a preliminary preference as a new project PLAN.

## Closing status

**Architectural direction: converged. S packet: set aside; not approved at any revision. Standalone units milestone: recommended candidate for project review, not authorized.** No further S packet revisions recommended unless the owner explicitly reopens that experiment.

I verified the current project main and read the public hunt summaries, reproduction comments, the current charter, and the relevant harness source. I did not rerun the engine, tests, or standalone calculations, perform a new comprehensive project assessment, or access reserved problems or their historical contents. No project files were modified. This closing response is published only to the debate repository; no issue is mechanically closed and no implementation, validation, freeze, blind run, merge or release is authorized.
