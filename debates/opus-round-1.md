# Opus — Round 1: the future of sim_waves composability

**Participant:** Claude Opus, builder and co-architect of sim_waves.
**Debate baseline:** sim_waves `main` at [`7244bb6fa984c441d9fbcdcbf2954e99c44b5efb`](https://github.com/physicsvigilante/sim_waves/commit/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb) (the merge of milestone 15a, PR #18). Every source link below is pinned to that commit.
**Kind:** architectural debate. This is not a PR review, and it authorises nothing: no implementation, validation, freeze or release.

## 0. Evidence rules and disclosures

Each claim carries one of these labels:

- **[Source]** I read it in the code or documents at the baseline.
- **[History]** It is recorded execution evidence from earlier runs: milestone results, review reports, error-hunt issues. I did not re-run it.
- **[Math]** An independent derivation, written out here so that it can be checked.
- **[Hypothesis]** My judgement, open to refutation.
- **[Experiment]** A proposed test that could falsify a recommendation.

For this round I read the source and documents at the baseline. I did not run project code, run tests or simulations, or modify sim_waves.

**Disclosure on reserved material.** As builder, I ran milestone 15a's stage 2 with the owner's approval, so I have seen its reserved scenes. I do not use them, or any other reserved or historical blind-evaluation material, as evidence in this debate. The passing-car case below is cited only through Astra's public assessment in [issue #15](https://github.com/physicsvigilante/sim_waves/issues/15).

**Bias disclosure.** I built most of what I criticise below. Where I defend an existing decision, it is because the evidence supports it, and I mark where it does not.

---

## 1. Verdict in one paragraph

The engine is **compositional in its physics laws but not in its representation.** The individual laws are first-principles and reusable:
- impedance-based reflection;
- emission-time Doppler;
- acoustic amplitude relations;
- string energy densities.

But the things they act on (waves, rays, resonators, emitters, pulses) are separate object families with separate interfaces. They are reached through a flat, global table of about 80 named functions, dispatched by `isinstance` and patched at import time. Values travel as bare SymPy expressions, while their dimension, domain, dependencies, provenance and guarantee travel in side channels filled at different times.

These two facts explain most of the recent defect record:
- the 37 confirmed error-hunt findings (of 38 reported) reduce to Astra's two shared causes: incomplete searches, and known metadata used too late;
- milestone 14 took 23 review rounds;
- run 7 had a hand step in 48% of its answers.

So the architecture is **neither fundamentally right nor fundamentally wrong.** The physics core and the correctness discipline are worth keeping. The representation layer should be redesigned incrementally, around two shared mechanisms:

1. **A typed quantity record** that carries its metadata through every operation.
2. **A one-dimensional waveguide network** that turns strings, pipes, rods, junctions, ends and resonators into one model. Modes, reflection, transmission, pointwise pressure and particle velocity, composite resonators and finite multi-interface propagation are then *derived* from it, not each implemented separately.

I recommend against both a wholesale rewrite and continued service-by-service growth.

---

## 2. What the system is today

### 2.1 Size and responsibility

**[Source]** At the baseline, `engine/simwaves/` has about 9,250 lines in 17 modules.
- About 2,100 of them are containment, harness, export and CLI.
- [`scene.py`](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py) alone is 3,156 lines, about 44% of the physics and semantics code.
- It holds scene construction, every medium and wave class, bound substitution, dimension inference, the observable table, limits, the global-extremum machinery and the numerical fallback.

### 2.2 The pipeline

**[Source]** A run goes through these stages:

1. **Parse:** `Scene.__init__` builds media, junctions, waves (plus their derived reflected and transmitted copies), resonators, legacy emitters, then sources, points, reflectors and rays ([scene.py L68–L160](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L68-L160)).
2. **Solve:** `solve_given` solves the `given` facts with a purpose-built branch enumerator over SymPy relations. It handles trig on the unit circle, monotonic functions, log-linear systems, `abs` sign cases and stated bounds. Each solution is then re-checked ([solver.py L431–L478](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/solver.py#L431-L478), [L548–L602](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/solver.py#L548-L602), [L704–L754](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/solver.py#L704-L754)).
3. **Observe:** each `observe` text is parsed *after* the solve into a SymPy expression containing the unknowns. Only then is the solution substituted, deferred calls resolved, and units converted ([run.py L588–L668](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L588-L668)).
4. **Arbitrary values:** the *whole scene* is rebuilt and re-solved at four further numeric scalings. A result is kept only where all five runs agree ([run.py L155–L240](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L155-L240)).

### 2.3 How a value's metadata travels

**[Source]** A value is a bare SymPy expression in exact SI. Its other properties are kept elsewhere:

| Property | Where it lives | When it is filled |
|---|---|---|
| Dimension | a separate AST pass (`_Dim`, [expr.py L516–L583](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L516-L583)); `symbol_dims`; a static `OBS_DIMS` table ([scene.py L2950](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2950)) | at parse time, by text |
| Domain obligations | a side list of conditions per reading (`translate`, [expr.py L50–L91](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/expr.py#L50-L91)) | while reading, before simplification |
| Stated bounds | the symbol is rewritten as `bound + offset` and recognised again later (`bound_variable`, `show_back`, [scene.py L162–L252](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L162-L252)) | at scene construction |
| Result basis (exact, proved, sampled) | the `sc.basis` dict, per observable name | after evaluation |
| Solved values | `sc.solution`, substituted late | after the solve |

Keeping domain obligations beside values, before simplification, was the correct fix in milestone 14: Astra said so in issue #15, and I agree. The problem is that it is one side channel among five, and each service must remember to consult each one.

### 2.4 Representations of "a wave"

**[Source]** There are four families that do not interconvert:

| Family | Model | Where it works | Examples of what it cannot do |
|---|---|---|---|
| Field waves (`Harmonic`, `GivenWave`, `SumWave`, `ShapedPulse`) | displacement y(x, t) on a 1D medium | strings; harmonic waves on gas, fluid or rod | `given:` waves only on strings ([scene.py L1182–L1190](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L1182-L1190)); energy only on strings ([energy.py L16–L31](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/energy.py#L16-L31)) |
| Rays (`Source`, `Ray`, `Reflector`) | a pressure phasor along a path, with optional Doppler tracing | 2D/3D sound interference, Doppler, echoes | on a moving ray, path lengths, phases, intensities, `k` and `omega` are refused ([FORMAT L478–L480](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L478-L480)) |
| Resonators | closed-form mode formulas: λ = 2L/n or 4L/n ([scene.py L2153–L2158](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2153-L2158)) | uniform string, pipe or rod with two ends | composite or non-uniform resonators; any field at a point inside the resonator other than the tabulated patterns |
| Emitters (legacy) | geometric spreading | point, line, plane | overlaps with powered sources ([FORMAT L526–L533](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L526-L533)) |

**Interfaces:**
- **[Source]** A junction creates `w_reflected` and `w_transmitted` for harmonic waves and shaped pulses only. It models one reflection; derived waves are not reflected again ([scene.py L2035–L2090](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2035-L2090); [FORMAT L169–L214](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L169-L214)).
- **[Source]** The physics of a junction (continuity at the boundary, impedance) and of a resonator's end (a node or an antinode) are the same boundary physics, but they are implemented twice, in unrelated classes.

### 2.5 Observable dispatch

**[Source]** `OBSERVABLES` is a module-level dict of about 80 names ([scene.py L3105–L3153](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L3105-L3153)).
- `rays.py` *replaces* entries such as `frequency`, `wavelength`, `phase` and `amplitude` at import time with wrappers that test `isinstance(w.obj, (Ray, Source))` before falling back to the original ([rays.py L515–L586](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/rays.py#L515-L586)).
- `modes.py` and `energy.py` add their own entries; the registration imports sit at the end of `scene.py` ([L3154–L3156](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L3154-L3156)).
- An observable's argument kinds, result dimension, domain rules and guarantee are spread across the handler, `OBS_DIMS`, the parser's special cases and FORMAT prose.
- **[Source]** About 60 lines of FORMAT.md describe a refusal, an error or a v0 limit. Many are physically justified. Several exist only because two families never met: "only `harmonic:` waves are reflected", "energy is defined here for waves on strings", and "a stated-length ray from a powered source is refused".

---

## 3. Question 1: is the engine compositional, and where does composition break?

### 3.1 Where composition genuinely works

**Doppler (milestone 10).** **[Source]** The engine solves the emission time te from the retarded-time equation |R(t) − E(te) − wind·(t − te)| = v(t − te). It takes the heard frequency as f·dte/dt, and extends this to moving walls by image motion and to light by special relativity ([FORMAT L418–L425](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L418-L425); [doppler.py L511–L556](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/doppler.py#L511-L556)).
- New combinations need no new formula: wind with a moving listener, an echo from a receding wall, a circular source.
- **This is the model of composition I want everywhere:** a few primitives (trajectories, a medium with a frame, a propagation law), and observations derived from them.

**Facts and unknowns.** **[Source]** Tagged facts, unknowns, and a single-situation guarantee (inconsistent, ambiguous or undetermined, never a guess) compose across every service that produces a SymPy relation. This is the engine's strongest contract.

**Medium-derived quantities.** **[Source]** For example, mu from density and area, speed from Y and density, and impedance. These compose with any wave placed on the medium.

### 3.2 Where composition breaks

Each break below is a structural cause, not a missing feature.

**B1. Metadata travels separately from values.** **[Source, History]** Four mechanisms resolve dimension, domain and dependencies at different stages, and late substitution adds a fifth.
- **[History]** Astra's error hunts traced the following to this one cause: H17-3, D1-5, D1-6, D2-4, D2-5, D4-1 to D4-9, and D5-4.
- Examples: a known symbol's dimension dropped in a fact; `at()` accepting seconds as a length; `equivalent()` proving a length equal to a time; a solved cut-off unused by `harmonics()`; Celsius offsets lost.
- These are not one-off misses. They are what happens when every service must re-assemble a value's metadata from side tables.

**B2. Completeness is the responsibility of each search.** **[Source, History]** Roots, extremes, resonances, ray maxima and minima, trig branches, `frequencies_in` and `width` each enumerate candidates their own way.
- **[History]** Findings H17-1, H17-2, D1-1, D1-4, D1-7, D2-1, D2-2 and D2-3 are each a search that returned a valid-looking but **incomplete** candidate set.
- Re-checking the returned candidates cannot detect a candidate that was never returned (Astra makes the same point in #20).
- **[History]** Milestone 14's rounds 17–22 were mostly this cause, rediscovered extreme by extreme ([REPORT](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/review14/REPORT.md)).

**B3. Representations are families, not views of one model.** **[Source]**
- A harmonic sound wave in a pipe, a resonator mode in the same pipe, and a ray from a source in the same air are three objects with three vocabularies.
- A standing wave in a pipe can report `pressure_nodes`, but not the pressure or particle velocity at a point and time. A ray can report `pressure_amplitude`, but has no field.
- **[History]** Run 7's gaps "pressure/compression at a point" and "a repeated pulse train with reflections" are exactly the places where no family covers the combination ([run 7 GRADES, top gaps](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/run7/GRADES.md#L221-L231)).

**B4. Interfaces do not compose.** **[Source]** One junction, one reflection, harmonic waves and shaped pulses only. A transmitted wave cannot meet a second junction; the scene is refused, and the author must "model one junction per scene".
- **[History]** Astra cites JEE Advanced 2025 Paper 1 Q7, with three connected strings and first encounters at two junctions, as a real exam case outside the model.

**B5. Shared entities cannot carry several observations.** **[History]** In Astra's assessment, the passing car was authored as two source copies at ±1000 m plus an identity assertion. One physical entity became two because an observation cannot select an event (before or after passing) on one trajectory.

**B6. Arbitrary values are implemented by re-running the whole scene.** **[Source]** Every service must survive being re-run at x0.01 to x89, so service guards interact with the sampler.
- **[History]** In run 7, an arbitrary sound speed was sampled at 340,000 m/s and tripped the "not sound" guard.
- This also creates two assurance levels that composition does not respect: sampled agreement for ordinary values, a symbolic proof for `nearest` ([run.py L155–L240](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L155-L240)).

**B7. Mechanical inputs are a menu, not a model.** **[Source]** String tension comes from a fixed set of routes: `hanging`, `weight` (with `buoyancy`), `stretch` (Y or k), `thermal` and `free` ([FORMAT L89–L112](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L89-L112)).
- **[History]** The first item in Astra's gap ordering is mixed string and pipe problems with changing mechanical conditions (inclines, pulleys, strain). Each such problem needs another route, or an author-typed force balance.

### 3.3 Is it "an expanding collection of problem-specific services"?

**[Hypothesis]** Partly. The *physics* is general. The *surface* grows by the product of object families × observables × options:
- `frequency` has five meanings, depending on its argument and keywords;
- `wavelength` has four;
- `amplitude` has three, with two refusals.

Each new family (rays in m5–m6, Doppler in m10, EM in m12) multiplied the special cases in the families that already existed.

---

## 4. Question 2: fundamental primitives of a JEE wave engine

**[Hypothesis]** I propose seven primitives. A primitive earns its place only if several observables or problem types are *derived* from it.

| Primitive | What it is | What is derived from it |
|---|---|---|
| **Medium** | a material with constitutive data (string: T, μ; fluid: B or γ, P, ρ; rod: Y, ρ; EM: c, n), optionally moving (wind or flow) | speed, impedance, intensity–amplitude relations, sound level |
| **Waveguide segment** | a medium over a 1D interval [a, b], uniform or with a coordinate profile | travelling-wave solutions, travel time, energy densities, modes (with ends) |
| **Boundary or interface** | a point condition joining segments, or ending one: fixed/free, closed/open (with end correction), or a junction of two or more segments | reflection and transmission, mode spectra, pressure and displacement nodes |
| **Source** | an emitter with a trajectory, a rest frequency, an amplitude or power, a phase and coherence, and spreading | rays, Doppler, intensity at a point |
| **Observer or point** | a location with a trajectory (possibly still) | received phase, frequency and intensity; events |
| **Event** | an instant defined by a condition on trajectories: emission, reception, closest approach, passing, first arrival at a junction | before/after branches; "when the source is at A" |
| **Fact** | a tagged equation or inequality | unknowns, branch selection, consistency |

**Not primitives:**
- Rays, resonators and pulses are **views**. A ray is the geometric-acoustics view of a source, a medium and an observer. A resonator is the eigen-view of a bounded waveguide network. A pulse is the time-domain view of a field on a network with given initial data.
- **"Wave" stops being a primitive.** A wave is a *solution* on a segment or network, in one of three views: travelling field, modal, or ray.

**The test of this choice.** **[Hypothesis]** An observation such as "the pressure amplitude at x = 30 cm in the 3rd harmonic of a closed pipe" should need no new service. It is the field view of the network (pipe plus ends), at a mode found by the eigen-view, read through the medium's constitutive law.

---

## 5. Question 3: can these interact through one coherent architecture?

### 5.1 The proposal: a 1D waveguide network

Every 1D wave problem at JEE level is a **network of uniform segments joined at points**:
- strings;
- pipes (air columns are 1D at JEE level);
- longitudinal rods.

Each segment carries one **across** variable and one **through** variable:

| Medium | Across variable (continuous at a junction) | Through variable (balanced at a junction) | Impedance Z |
|---|---|---|---|
| String | displacement y | transverse force −T ∂y/∂x | √(Tμ) |
| Air column (area S) | pressure p′ = −B ∂ξ/∂x | volume velocity S ∂ξ/∂t | ρc/S |
| Rod (area S) | displacement ξ | axial force −YS ∂ξ/∂x | S√(Yρ) |

**Boundary conditions are statements about these variables:**
- a fixed string end: y = 0;
- a free string end: T ∂y/∂x = 0;
- a closed pipe end: volume velocity = 0;
- an open pipe end: p′ = 0, at L + end correction;
- a clamped or free rod end: ξ = 0, or the axial force = 0.

**Everything currently special-cased follows:**
1. **Reflection and transmission** at a junction come from continuity of the across variable and balance of the through variable. In displacement form: r = (Z₁ − Z₂)/(Z₁ + Z₂), t = 2Z₁/(Z₁ + Z₂).
2. **Modes** are the eigenvalues of the network with its ends: the frequencies at which a nonzero field satisfies every condition.
3. **Pointwise acoustics** (pressure, particle velocity, compression versus rarefaction) are linear operators on the field: p′ = −B ∂ξ/∂x and u = ∂ξ/∂t.
4. **Finite multi-interface propagation** in time is d'Alembert characteristics plus the junction coefficients, unrolled for a declared, finite number of events.

**On equation-based modelling.** This is the transmission-line (impedance) formalism of classical acoustics, not a copy of Modelica. It takes one idea from acausal modelling, connectors with across and through variables, where it fits exactly: point junctions of 1D waveguides. It takes nothing else.

**[Hypothesis]** A general equation-based DAE architecture does *not* suit this project:
- JEE answers are exact closed forms of linear PDEs in the frequency domain, or of d'Alembert solutions;
- time integration would replace exact results with numerical ones;
- that would weaken the result guarantees the project exists to protect.

### 5.2 Independent derivations that support the proposal

**[Math] D-1: the closed-open pipe from boundary conditions alone.**
- Take the closed end at x = 0 and ξ(x, t) = A sin(kx) cos(ωt). The closed end gives volume velocity S ∂ξ/∂t = 0 at x = 0, which holds.
- Then p′ = −B ∂ξ/∂x = −BAk cos(kx) cos(ωt).
- The open end gives p′(L_eff, t) = 0, so cos(kL_eff) = 0, so kL_eff = (2n − 1)π/2.
- Hence λ = 4L_eff/(2n − 1). This is the engine's formula, with only odd harmonics.
- From the same field:
  - displacement nodes are at sin(kx) = 0;
  - pressure nodes are at cos(kx) = 0, so the closed end is a pressure antinode and the open end a pressure node;
  - the particle velocity is −Aω sin(kx) sin(ωt), a quarter period behind the pressure in time.
- **Consequence:** `wavelength`, `frequency`, `nodes`, `pressure_nodes`, `mode_shape` and the missing pointwise pressure and particle velocity all come from one field and two boundary statements. Today they are separate code paths ([scene.py L2153–L2158](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L2153-L2158); `modes.py`). The pointwise quantities do not exist.

**[Math] D-2: a composite string. This is the falsification case for the whole proposal.**
- Two strings of equal length L, both ends fixed, equal tension T, μ₂ = 4μ₁.
  - Then v₂ = v₁/2, k₂ = 2k₁, and Z₂ = 2Z₁.
- Field on each side:
  - y₁ = a sin(k₁x) on [0, L];
  - y₂ = b sin(k₂(2L − x)) on [L, 2L].
- **Junction conditions:**
  - continuity: a sin(k₁L) = b sin(k₂L);
  - force balance: a k₁ cos(k₁L) = −b k₂ cos(k₂L).
- **Family 1 (sines nonzero):** dividing gives k₁ cot(k₁L) + k₂ cot(k₂L) = 0. At equal tension k ∝ Z, so this is Z₁ cot(k₁L) + Z₂ cot(k₂L) = 0.
  - With θ = k₁L and k₂ = 2k₁: cot θ + 2 cot 2θ = 0.
  - Using cot 2θ = (cot²θ − 1)/(2 cot θ): (2cot²θ − 1)/cot θ = 0, so **cot θ = ±1/√2**.
  - So θ = arctan √2 ≈ 0.95532, θ = π − 0.95532 ≈ 2.18628, and these plus multiples of π.
  - Check at θ = 0.95532: cot θ = 0.70711; 2θ = 1.91063 and cot 2θ = −0.35355; so cot θ + 2 cot 2θ = 0.
- **Family 2 (sin(k₁L) = 0):** then sin(2k₁L) = 0 too, and the force condition fixes b/a = ∓1/2. So θ = nπ: these are the "node at the junction" modes.
- **The full spectrum** in θ = k₁L is {0.95532, 2.18628, π, 4.09691, 5.32791, 2π, …}.
- **The physics:** the true fundamental, θ ≈ 0.9553, has **no node at the junction**. Its frequency is 0.9553/π ≈ 0.304 times the "node at the joint" frequency that JEE problems usually ask for, which is θ = π.
- **The architectural point:**
  - A JEE statement that says "a node at the joint" is a **fact** selecting family 2. The engine should be told it, as a tagged fact, and must derive the rest.
  - An engine that composes the junction correctly must also *refuse to assume* that fact. Today's per-resonator formulas cannot even represent this question.
  - A correct composite answer can therefore disagree with a careless book reading. **That is the intended behaviour**, but it raises the bar for authors (section 11).

**[Math] D-3: three strings, first transmissions and a power check.**
- Equal tension, μ₁ : μ₂ : μ₃ = 1 : 4 : 16, so Z₁ : Z₂ : Z₃ = 1 : 2 : 4 and v₂ = v₁/2, v₃ = v₁/4.
- Coefficients:
  - at J₁: r₁₂ = −1/3, t₁₂ = 2/3;
  - at J₂: r₂₃ = −1/3, t₂₃ = 2/3.
- A unit incident amplitude gives a first transmitted amplitude into string 3 of (2/3)² = 4/9. The first reflection back from J₂ has amplitude −2/9 in string 2.
- **Power check,** with P ∝ Z A²:
  - at J₁: 1 = 1·(1/9) + 2·(4/9), that is 1/9 + 8/9;
  - at J₂: 2·(4/9) = 8/9 = 2·(4/81) + 4·(16/81), that is 8/81 + 64/81 = 72/81.
- First-arrival times follow from each segment's length and speed.
- This is the independent oracle class for the finite multi-interface propagation that run 7 and the 2025 Advanced paper need.

### 5.3 Where the waveguide network does not reach, and what stays

These keep their current models and join the network only through shared primitives:

- **Rays** (2D/3D interference, image reflections and Doppler). Sources, observers, media and events are shared with the network; ray observables stay geometric. **[Hypothesis]** A ray is the far-field view of a source and needs no waveguide.
- **Non-uniform strings** (a hanging rope, μ(s)). Travel time ∫ds/v and front positions stay as now. They are refused for modal and scattering views unless an exact solution is available. **[Source]** Today they are already excluded from resonators.
- **End corrections** stay as stated lengths (a JEE convention), not as a radiation impedance.

---

## 6. Question 4: SymPy-centric, a typed IR, or something different?

### 6.1 Options compared

| Option | For | Against |
|---|---|---|
| (a) SymPy-centric, as now | exact arithmetic; mature simplification; the existing physics is written in it | simplification erases domains; it has no units or quantity kinds; its `solve` gives no completeness guarantee; its behaviour shifts between versions (milestone 14's records) |
| (b) **A thin typed layer over SymPy** | metadata travels with values; SymPy stays the algebra backend; incremental adoption | another layer to maintain; every operation needs a rule for each field |
| (c) Numeric-first with intervals | certified enclosures; bounded run time | loses exact symbolic answers (π/25 W, 2125/4 Hz), which JEE options and proofs use |
| (d) A new CAS or term-rewriting core | full control | far too costly; reproduces SymPy badly |

### 6.2 Recommendation

**[Hypothesis]** Option (b), plus **certified numerics as a service, not as the core**:

- **Exact first:** SymPy for algebra and exact evaluation, as now.
- **Certified fallback with a declared class:**
  - for root isolation and extrema in one variable on a bounded interval, with a function class whose derivatives can be bounded (polynomials, trigonometric polynomials with numeric coefficients, sums of Gaussians);
  - use interval arithmetic or Sturm/Descartes isolation;
  - the result is a **certified enclosure**, or a **specific refusal** when the budget or resolution is reached.
- **[Hypothesis]** This would replace the bespoke extremum enumerator and the uncertified golden-section fallback ([scene.py L1790–L1869](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L1790-L1869)) with one contract. That fallback currently returns a best sample with no stated precision.
- **[History]** Issue #15 already listed `step(sin(10**17*(x-1)))` as the kind of input that consumed milestone 14's review rounds. Under a certified contract, such an input gets "cannot resolve within budget" by construction, not by a growing list of special cases.

---

## 7. Question 5: propagating metadata through composed operations

**[Hypothesis]** I propose one record, `Q`, as the only value type between the scene and SymPy:

```
Q = (value     : SymPy expression in exact SI
     dim       : dimension exponent vector
     kind      : quantity kind, finer than dimension
                 (frequency vs angular frequency; pressure vs pressure amplitude;
                  absolute temperature vs temperature difference; displacement vs path length)
     domain    : conditions for the value to be real and finite, each tagged with its origin
     deps      : the symbols it depends on, each with its role
                 (unknown, free, arbitrary, bounded) and its declared kind and domain
     basis     : exact | proved | certified(tolerance) | sampled | numeric-uncertified
     provenance: the facts and engine laws used)
```

**Rules for operations:**

| Operation | dim | kind | domain | deps | basis | provenance |
|---|---|---|---|---|---|---|
| Algebra (+, ×, sin, log, …) | multiplies or checks | per a small kind algebra (e.g. ω/2π is a frequency) | union, plus the operation's own conditions (log needs > 0) | union | the weakest of its inputs | union |
| Binding a solved value | — | — | substituted, then re-decided | the unknown removed | — | the solve's facts added |
| Differentiation or a change of variable | adjusted | adjusted | carried as the transform requires | — | — | — |
| A search (extremum, roots) | — | — | the domain must be decided over the search region | **every dependency must be bound before the search runs** | per the service's declared contract | — |

**[Source, History]** That last rule is the direct fix for "known metadata used too late" (D2-4, D2-5).

**Specific decisions:**

1. **Stated bounds become part of a symbol's domain, not an affine rewrite.** The rewrite in [scene.py L162–L252](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/scene.py#L162-L252) forces every service to recognise `base + s·p` and undo it. **[History]** That was a recurring source of milestone 9 and 14 defects.
2. **Arbitrary values become symbols with role `arbitrary`.** A result is reported as independent of them only when ∂value/∂a ≡ 0 can be proved (the engine already does this for `nearest`, [run.py L104–L147](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/engine/simwaves/run.py#L104-L147)).
   - Otherwise the result is NOT AVAILABLE, or carries the explicit `sampled` basis that a strict workflow refuses.
   - This retires whole-scene numeric re-runs as the default, and with them the sampler-versus-guard interactions (B6).
   - **[Math]** No finite sample set can certify independence. For any finite set S of sampled values and any function g, g(a) + Π_{s∈S}(a − s)·h(a) agrees with g on S. Sampling is evidence, not proof; the project's own policy (point 4) already says so.
3. **Every observable is registered with a signature:** argument kinds, result dim and kind, domain rules, guarantee, and the views it applies to. That one registration replaces `OBSERVABLES`, `OBS_DIMS`, the `_wrap_obs` patches and the scattered prose. **[Hypothesis]** It makes refusals such as "frequency(ray) needs t=" mechanical, not handwritten.
4. **Quantity kind is minimal and closed.** I would start with the kinds that confirmed findings needed:
   - ordinary versus angular frequency (D4-7);
   - absolute temperature versus difference (D4-5, D4-10);
   - list element dimension (D5-4).

   I would not build a general ontology.

---

## 8. Question 6: branches, discontinuities, singularities, limits, extrema; exact versus numerical

**[Hypothesis]** Each search service declares a contract with four parts:

1. **Class:** for example, one real variable on a closed bounded interval; finitely many breakpoints, listed exactly; analytic between them.
2. **Guarantee:** a complete exact set; or a complete certified enclosure with tolerance τ; or a refusal.
3. **Preconditions:** every dependency bound; the domain decided over the whole region.
4. **Budget:** time and memory per call.

**Rules:**
- **Branches** (trig solves, `abs` cases, periodic families) are enumerated within the declared bound.
  - An unbounded periodic family is reported as such: ambiguous, with the family named. It is never truncated.
  - **[Source]** The solver already does this for phases; D1-1 and D1-4 show where it leaks.
- **Discontinuities and singularities** are part of the class, never discovered by sampling. A jump or pole inside the region either splits it at a known point or refuses.
- **Limits:** keep milestone 14's narrowing (numbers only, apart from the variable). **[History]** It ended a seven-round chain. Reopening symbolic limits needs a declared class and a proof method first.
- **Exact versus numeric:** an exact result keeps `exact`.
  - A certified enclosure reports `certified(τ)` and prints only the digits it guarantees.
  - An uncertified numeric value is never printed beyond the digits its method guarantees, and is never compared exactly.

---

## 9. Question 7: are the repeated review cycles symptoms of deeper weaknesses?

**[History, Hypothesis]** Yes for milestone 14 and the error hunts. Mostly no for milestone 15a.

- **Milestone 14 (23 rounds):**
  - rounds 2–16 found ways for a domain or symbol to be lost through SymPy simplification before a service saw it;
  - rounds 17–22 found incomplete extremum and breakpoint searches.

  These are B1 and B2, rediscovered locally each time. The fix that worked, keeping obligations beside values, was a shared mechanism, which supports the shared-mechanism thesis.
- **Error hunts #17, #19–#23 (38 findings, 37 confirmed):** Astra's own synthesis names two causes, completeness and late metadata, which are B2 and B1.
- **Milestone 15a (12 rounds):** about process containment under concurrent cancellation, not physics. The lesson there was procedural: declare the supported class before building. That is now policy, and it matches section 8's contracts.

**The strongest counter-argument.** Part of the cycle comes from the format's *breadth*, not its architecture.
- `observe` accepts nearly arbitrary expressions, so an adversarial reviewer will always find an expression that some service mishandles.
- A typed core reduces the *classes* of failure; it does not remove adversarial surface.
- **[Hypothesis]** The complementary measure is to **narrow the expression language**: typed quantities in, a fixed operator set, and searches only through declared services.

---

## 10. Question 8: where the author supplies physics the engine should derive

**[History]** Run 7 had a hand step in 21 of 44 answers (48%), and 6 were decided by hand ([GRADES](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/eval/run7/GRADES.md)). Grouped by cause:

| The author supplied… | Example | The fix in this proposal |
|---|---|---|
| Pressure or compression at a point | run 7 "pressure at a point" | the field view of the waveguide network (D-1) |
| A train of pulses with reflections | run 7 "repeated pulse train" | finite time-domain propagation on the network (D-3) |
| One entity represented as two | the passing car, via two source copies | events on shared trajectories |
| Mechanical equilibrium for tension | incline or pulley statics typed as a fact | **open; see Q5 to Astra** |
| Sign and phase conventions | converting `A sin(kx − ωt + φ)` to `phase: π − φ` ([FORMAT L135–L143](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L135-L143)) | **[Hypothesis]** accept the textbook form as written and derive the phase; `given:` with `on:` already does this for strings only |
| The choice of representation | choosing ray, wave or resonator for the same air | the network makes this a view choice; the medium and objects are shared |
| An accelerating source | run 7 integer/1 | events plus a bounded trajectory class: a declared extension, not a default |

**[Source]** FORMAT's rule that "the LLM never writes a physics formula" ([L13–L15](https://github.com/physicsvigilante/sim_waves/blob/7244bb6fa984c441d9fbcdcbf2954e99c44b5efb/format/FORMAT.md#L13-L15)) conflicts with the grading convention that keeps an author-typed mechanical relation "verified". Astra raised this. I propose a provenance field, `external_relation`, on facts, so that scorecards count such facts separately, and so that a future mechanics provider can replace them.

---

## 11. Question 9: the most coverage per unit of complexity

**[Hypothesis]** My ranking, with the evidence for each:

1. **The waveguide network, harmonic regime.** One model covers:
   - string, pipe and rod modes (existing);
   - single-junction reflection (existing);
   - composite resonators (new);
   - pointwise pressure, particle velocity and phase relations (new; Astra's gap 2);
   - pressure and displacement nodes (existing, now derived).

   It removes two families (resonator formulas, junction copies) and replaces them with one. **This is coverage gained by subtraction.**
2. **The typed quantity core with registered signatures.** By itself it adds little *coverage*. It is the *correctness multiplier* that most of the 38 findings call for, and the network should be built on it from the start.
3. **Finite time-domain propagation** on the network: pulses through N junctions over a stated event horizon (Astra's gap 3 and the 2025 Advanced Q7). It reuses (1)'s coefficients.
4. **Events on shared entities** (Astra's gap 4; Advanced-weighted).
5. **Certified search service:** correctness and run-time containment, low direct coverage.

**The cost for authors.** **[Hypothesis]** A composing engine exposes physics that book keys often leave implicit (D-2's node-at-junction family).
- More scenes will be correctly *ambiguous* until the author states the fact the problem implies.
- I count that as the engine working, but it will look like lower coverage in a naive scorecard.
- Scorecards should therefore separate "ambiguous because a stated fact was omitted" from "unsupported".

---

## 12. Question 10: designing from scratch today

| Retain | Remove | Redesign |
|---|---|---|
| The correct-or-clear-error contract; inconsistent, ambiguous and undetermined | Legacy `emitters` (overlap with sources) | The value type: `Q` records (section 7) |
| Tagged facts with provenance; an independent re-check of every solution | Import-time patching of `OBSERVABLES` | Waves as views of a waveguide network, not separate families |
| Exact SI with SymPy as the backend | The affine rewrite of stated bounds | Observables as registered signatures |
| The first-principles emission-time Doppler model | Whole-scene numeric re-runs as the *default* arbitrary-value mechanism | Searches as contract services with certified numerics |
| Domain obligations recorded before simplification | `given:` waves restricted to strings | Arbitrary values as symbolic parameters with proved independence |
| Per-request containment (milestone 15a); pinned environment; CI | Per-family duplicates of boundary physics | The expression language narrowed to typed operations |
| The two-stage milestone process and blind transfer | | |

**[Hypothesis]** From scratch, I would *not* change the physics laws, and I would not change the scene format's surface very much. Most current scene syntax can be lowered onto the new core. The redesign is internal: values, views and searches.

---

## 13. Comparing the three strategies

| Strategy | What it is | For | Against | Verdict |
|---|---|---|---|---|
| **Incremental evolution** | keep adding services and fixes per finding | lowest cost per step; no migration risk | defects grow with families × observables × options; the history shows the same causes found again round after round; author workarounds persist | **reject as the default** |
| **Extracting shared mechanisms** | a typed core, the waveguide network, events and certified search, introduced behind the existing syntax, each with parity gates | removes causes, not instances; incremental; each step falsifiable; preserves the existing scenes | needs strict parity discipline; temporarily two code paths; risk of an abstraction that never retires the old code | **recommend** |
| **Substantial redesign** | a new core (an equation-based DAE, or a new CAS), with the format rewritten | clean conceptual model | discards about 7,000 lines of reviewed physics and 339 regression scenes; equation-based time integration weakens exactness; a long period with no capability gains | **reject**, unless the first experiment fails |

**The strongest arguments against my recommendation:**

1. **The network may be the wrong unifier.**
   - JEE uses a handful of closed-form mode formulas. A general eigen-solver adds transcendental equations (D-2), which then need certified root isolation. That is more machinery for problems the formulas already solve.
   - If composite resonators and multi-junction problems are rare in JEE, the payoff is small.
   - **[Experiment]** E1 below measures this.
2. **Two code paths during migration** have historically been where silent divergence hides.
   - Parity tests must compare outputs *and* bases on every existing scene, not only values.
3. **A typed record per value may be too costly** in SymPy-heavy paths (performance), or drift out of sync with the expression.
   - An alternative is a constraint store keyed by expression identity; see Q2 to Astra.
4. **Exposing omitted facts (D-2)** could lower measured coverage, and frustrate LLM authors, before it raises answer quality.

---

## 14. Migration strategy and roadmap

**[Hypothesis]** This reorders, but does not discard, the roadmap adopted after issue #15 (m15b contracts; m16 point acoustics; m17 events; m18 interfaces).

| Step | Scope | Exit criterion |
|---|---|---|
| **m15b, Typed core and contracts** (absorbs the error-hunt fix milestone) | the `Q` record on the parse → solve → observe path; a registered observable signature for every function; stated bounds as domains; dependencies bound before searches; the minimal kind set | the D4, D5-4, H17-3, D1-5/6 and D2-4/5 findings closed **by the mechanism**, with no finding-specific code; parity on all 339 + 42 + 12 scenes, bases included |
| **Gate 1, Waveguide network, harmonic regime** (the first experiment, section 15) | segments, ends and two-segment junctions for string, pipe and rod; modes, r/t, field-at-a-point views | parity, plus a frozen-code transfer |
| **m16′, Network completion** (replaces m16 and the harmonic half of m18) | N-segment networks; composite resonators; pointwise acoustics in every 1D view | oracle tests from D-1, D-2, D-3 |
| **m17, Events on shared entities** | unchanged from the adopted roadmap | the passing car as one source; closest approach; before/after branches |
| **m18′, Finite time-domain propagation** | pulses on the network with a stated event horizon; cycles refused | the D-3 class of oracle; multi-junction first-encounter problems |
| **Certified search** | the contract service of section 8, used by peak, `max_in`, maxima and minima, resonances | the D2 findings closed; a sub-resolution input refused by construction |
| **Broad stratified evaluation** | as Astra recommended | full coverage ∩ correct, reported separately from sampled answers and refusals |

**Compatibility requirements throughout:**
- every existing regression, example and negative scene passes with **identical value and basis**, or with a strictly stronger basis justified in its PR;
- no change to scene syntax without a documented lowering for old scenes;
- no silent widening of a refusal into an acceptance without an oracle test.

---

## 15. The first implementation experiment (proposed Gate 1 outline)

**Hypothesis under test:** one 1D waveguide network can replace the resonator and junction families with no loss, and can answer new combinations without production changes.

**Scope (bounded):**
- uniform segments of string, gas or fluid, and rod;
- ends: fixed or free, closed or open (with end correction), clamped or free;
- one junction of two segments;
- the harmonic regime only.

**Views:**
- modes (the eigen-condition), solved exactly where the closed form exists;
- otherwise certified isolation within a stated frequency band, or a refusal;
- field at a point (displacement, slope, pressure, particle velocity);
- reflection and transmission coefficients.

**Built on `Q` from the start, only for its own inputs and outputs:** a local adoption, not the global m15b refactor.

**Parity (Gate 1A):** route the existing `Resonator` frequency, wavelength, node and antinode functions, `pressure_nodes`, and harmonic-wave junction coefficients through the network. Every existing scene using them must give identical values and bases.
- **[Experiment]** A delegation check: perturb one network coefficient in a test-only probe. Every routed consumer must change. This shows that the old path is really retired.

**Independent oracles (no engine imports):**
- D-1 (closed-open pipe: λ, nodes, pressure and velocity phase);
- D-2 (composite string: both families; the fundamental arctan √2 / k₁L);
- D-3 (power balance at each junction);
- the standard string, pipe and rod mode formulas.

**Frozen-code transfer (Gate 1C):** after the freeze, a separate agent writes reserved scenes that **combine** supported pieces in ways the developer did not test:
- a composite string with and without a stated node at the joint;
- the pressure at a point in a pipe mode;
- a rod joined to a rod;
- a junction with a free far end.

**What falsifies the hypothesis:**
- any production change needed for a reserved scene;
- any parity break;
- the network code not retiring at least the resonator mode formulas and the junction coefficient code.

**Explicit non-goals for Gate 1:**
- time-domain pulses;
- more than two segments;
- non-uniform media;
- rays and Doppler;
- arbitrary-value changes;
- the global typed-core refactor.

---

## 16. Non-goals and limitations of this proposal

- **Not proposed:**
  - a general PDE solver, 2D or 3D fields beyond rays, or modal analysis of membranes;
  - general rigid-body mechanics (tension from statics is an open question, Q5);
  - wave optics, nonlinear acoustics, arbitrary trajectories;
  - a CAS rewrite, or replacing SymPy.
- **The coverage estimates here are judgements.** The frequency of composite and multi-interface problems in JEE has not been measured. The proposed stratified evaluation should precede any coverage claim.
- **D-2 has a closed form only because the lengths are equal and the impedance ratio is 2.** The general case is transcendental, which is precisely why the certified-search contract must come with the network, not after it.

---

## 17. Five questions for Astra

1. **The unifier.** Is the 1D waveguide network, with across and through connectors (continuity and balance at junctions) and boundary conditions on the same variables, the right single model for strings, pipes, rods, junctions and resonators? Or does it overfit, given end corrections treated as lengths, non-uniform (hanging) strings, and the JEE habit of *stating* a node at a joint (D-2)? What concrete case would you use to falsify it before Gate 1 is approved?

2. **Where metadata lives.** Should dimension, kind, domain, dependencies, basis and provenance travel in one record per value (my `Q`), or in a separate constraint store keyed by SymPy expression identity? Give the case in which my record design fails: under substitution, differentiation, or SymPy re-simplification of a record's value.

3. **Certified numerics.** Do you accept a certified enclosure, printing only its guaranteed digits, as an acceptable result basis for JEE answers, distinct from `exact` but sufficient for options and comparisons? Should it *replace* the bespoke symbolic branch and extremum enumeration in the declared classes, or only back it up? What exact function class makes that sound within a fixed budget?

4. **Arbitrary values.** Should whole-scene numeric rescaling be retired as the default, in favour of symbolic parameters with proved independence, accepting NOT AVAILABLE where no proof is found? What coverage loss would you accept? Is there a proof route stronger than ∂R/∂a ≡ 0 that JEE problems need (for example, dimensional analysis when the arbitrary values form a dimensionally independent set)?

5. **Sequencing and mechanics.** Your issue #15 ranking puts mechanical assembly (incline, pulley, strain) first among the gaps. Should sim_waves (a) grow a minimal statics vocabulary, (b) take tension from a separate mechanics engine through a declared `external_relation` interface, or (c) keep the route menu and declare general statics out of scope? And should the first experiment be the waveguide network on a *local* typed core (my proposal), or should the global typed core (m15b) come first, with the network after it?

---

*End of Opus — Round 1. No sim_waves files were modified, no project code or tests were run, and no reserved or blind-evaluation material was used as evidence.*
