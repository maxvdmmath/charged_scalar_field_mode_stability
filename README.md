# Mode stability of the charged scalar field for every angular momentum, in Lean 4

This repository contains a Lean 4 / Mathlib proof of mode stability for the massless charged
scalar field on a subextremal Reissner–Nordström exterior, for **every spherical-harmonic degree
`L ∈ ℕ₀`**. The mathematical source is [`mode_stability_all_angular_momenta.tex`](mode_stability_all_angular_momenta.tex)
(see [`sources/README.md`](sources/README.md)); line numbers below refer to it.

**Status.** The main theorem `CSF.AllL.no_common_outgoing_all_L` is proved. There is no `sorry`
and no project axiom: every constant of the libraries `CSF` and `CSFAllL` depends only on
`propext`, `Classical.choice` and `Quot.sound`.

**Scope.** The formalization covers Section 1 of the source, from the normalized radial equation
(eq:main-radial-equation-hintz) on. The introduction's reduction from the wave equation to that
radial equation is not formalized, and neither are existence and uniqueness of the normalized
outgoing solutions `u_H`, `u_I`. The theorem needs neither: it constrains every solution with the
outgoing asymptotics. See [*Not formalized*](#not-formalized).

## The theorem

**`CSF.AllL.no_common_outgoing_all_L`** (`CSFAllL/MainTheorem.lean`; Theorem
`thm:mode-stability-charged-scalar-field`, lines 391-406, with the common-solution argument of
Proposition `prop:sigma-zero-threshold`, lines 911-951, and of the proof of the Theorem,
lines 966-977):

```lean
theorem no_common_outgoing_all_L :
    ∀ (L : ℕ) {κ lam : ℝ}, 0 < κ → κ ≤ 1 → 0 < lam →
    ∀ {σ : ℂ}, 0 ≤ σ.im →
    ∀ {r : ℝ → ℝ}, IsTortoise κ r →
    ∀ {U : ℝ → ℂ}, IsRadialSolXL κ lam σ L r U →
    ∀ {BH BI : ℂ}, OutgoingHX lam σ r U BH →
    (if σ = 0 then ThresholdOutgoingIXL κ L lam r U BI
      else ∃ ΦR, IsPhaseR κ lam σ ΦR ∧ OutgoingIX κ r ΦR U BI) →
    (σ ≠ 0 → BI = 0) ∧ (σ = 0 → BH = 0)
```

For every `L`, `0 < κ ≤ 1`, `λ > 0`, `Im σ ≥ 0` and tortoise coordinate `r(x)`: let `U` solve
(eq:main-radial-equation-hintz) of degree `L`, behave like `B_H e^{-iλ(σ-1)x}` as `x → -∞`
(eq:horizon-outgoing-definition), and behave at infinity like `B_I e^{iΦ}` with a long-range
phase `Φ` if `σ ≠ 0` (eq:infinity-outgoing-definition), or like `B_I r^{s_L(λ)}` (Frobenius
convention (eq:threshold-outgoing-definition) of degree `L`) if `σ = 0`. Then `B_I = 0` if
`σ ≠ 0` and `B_H = 0` if `σ = 0`. Since "outgoing" means this asymptotic behaviour with a
nonzero amplitude (Definition "Outgoing at both ends", lines 375-377), no solution is outgoing at
both ends.

The amplitudes `B_H`, `B_I` are arbitrary (possibly zero), as in the source's proof
(lines 412-417), and no existence statement is assumed. The other formulations of the theorem
are proved from it (`CSFAllL/Corollaries.lean`):

* `CSF.AllL.mode_stability_all_L_wronskian`: the Wronskian form ("Equivalently,
  `𝒲_{κ,λ}(σ,L) ≠ 0`", lines 396-401, and (eq:no-zero-at-sigma-zero)). For any solutions
  `u_H`, `u_I` with nonzero amplitudes, `W(u_H,u_I)(x) ≠ 0` for every `x`. As in the proof of
  the Theorem (lines 978-992), no ODE uniqueness is used: where `W` vanishes, one solution is a
  constant multiple of the other near the end at which the latter has a nonzero amplitude.
* `CSF.AllL.no_solution_outgoing_at_both_ends_all_L`: the literal sentence
  `¬ (B_H ≠ 0 ∧ B_I ≠ 0)`.

Going back from the Wronskian form to the theorem would need existence and uniqueness of the
outgoing solutions at each end, which are not formalized; only the literal sentence follows
directly from the Wronskian form, by taking `u_H = u_I` (since `W(U,U) = 0`).

Definitions (`CSFAllL/Defs.lean`): `CSF.AllL.IsRadialSolXL` (the equation of degree `L` in `x`,
lines 316-321), `CSF.AllL.thresholdExponentL` (the selected exponent, lines 883-909),
`CSF.AllL.ThresholdOutgoingIXL` (the threshold convention of degree `L` in `r`, lines 890-904).
The horizon condition `CSF.OutgoingHX`, the long-range phase `CSF.IsPhaseR`, the condition at
infinity `CSF.OutgoingIX`, the tortoise coordinate `CSF.IsTortoise` and the Wronskian
`CSF.wronskianX` do not depend on `L` (lines 302-389) and come from the `L = 0` library `CSF`.

The case `L = 0` (`CSFAllL/Regression.lean`): the `L = 0` theorems `CSF.mode_stability_part_i`,
`CSF.no_common_outgoing_x` and `CSF.threshold_no_common_x` are recovered, with exactly their
types (`type_of%`), from the theorem at `L = 0` (`CSF.AllL.mode_stability_part_i_of_all_L`,
`CSF.AllL.no_common_outgoing_x_of_all_L`, `CSF.AllL.threshold_no_common_x_of_all_L`);
conversely, the theorem at `L = 0` follows from them
(`CSF.AllL.no_common_outgoing_all_L_zero_of_x`).

## How to check

Toolchain (pinned; do not run `lake update`): `lean-toolchain` `leanprover/lean4:v4.35.0`;
Mathlib tag `v4.35.0` (commit `56a005855ea17e0b0958f73ec801ae140b0a1679`, `lake-manifest.json`).

```bash
lake exe cache get
```

```bash
lake build
```

`lake build` builds the libraries `CSF` and `CSFAllL`.

```bash
lake env lean scripts/AxiomsAllL.lean
```

Prints `[propext, Classical.choice, Quot.sound]` for each of the 54 named declarations (the
theorem, its corollaries, the propositions of the proof and the case `L = 0`), then checks every
constant declared in a module of `CSF` or `CSFAllL`, auxiliary constants included, and prints
`EXHAUSTIVE AXIOM AUDIT OK`. It fails if any constant uses another axiom (such as
`sorryAx`), or if the modules it loads from these libraries differ from the Lean source files of
the checkout (for example a stale build file of a deleted module). Run it from the repository
root. `lake env lean scripts/Axioms.lean` prints the same report for the 48 named declarations of
the `L = 0` library `CSF`.

```bash
scripts/kernel_replay.sh
```

Replays every module in Lean's kernel with `leanchecker`, one module at a time (20 modules), and
prints `KERNEL REPLAY OK`. The module of the main theorem alone is checked by
`lake env leanchecker CSFAllL.MainTheorem`. Do not pass a library root such as `CSFAllL` to
`leanchecker`: without `--fresh` it replays every module whose name starts with the argument,
concurrently, which ran out of memory on a 16 GB machine.

## Files

| File | Content |
|---|---|
| `mode_stability_all_angular_momenta.tex` | the mathematical source (introduction and Section 1) |
| `CSFAllL/Defs.lean` | the equation of degree `L` and the threshold convention |
| `CSFAllL/MainTheorem.lean` | the main theorem |
| `CSFAllL/Corollaries.lean` | the Wronskian form and the literal sentence of the theorem |
| `CSFAllL/Conjugation.lean` | conjugated equation, current `𝒥_ε` with the angular term, current identity, integrated identity |
| `CSFAllL/ModeStability.lean` | `IsRadialSolA` (eq:y-equation with `V + A`), endpoint currents, seam, master identity, current argument |
| `CSFAllL/Flux.lean` | flux, localization, Props `no-upper-half-plane-modes`, `no-real-superradiant-modes`, real frequencies |
| `CSFAllL/Threshold.lean` | exponent of degree `L`, threshold flux for any `Re s ≤ 1/2`, Prop `sigma-zero-threshold` |
| `CSFAllL/PartOneX.lean` | `x → y` transfer of the angular term; the theorem and propositions in the original variables |
| `CSFAllL/WronskianX.lean` | the Wronskian in `x`: independence of `x`, proportionality near an end, germ lemmas |
| `CSFAllL/Asymptotics.lean` | (eq:sigma-zero-asymptotic-equation) of degree `L` |
| `CSFAllL/Regression.lean` | the case `L = 0`, compared with the library `CSF` in both directions |
| `CSF/*.lean` | the `L = 0` library: `Basic` (coefficients `Ω²`, `K`, `g`, `μ`, `V`, `H`, key factorization), `Conjugation`, `ModeStability`, `Flux`, `Threshold`, `PartOne` (Wronskian in `y`), `Tortoise` (tortoise coordinate), `PartOneX` (original variables `x`, `r`), `Asymptotics` (long-range phase) |
| `scripts/` | axiom audits and kernel replay |

## Formulation choices

* **Wronskian sign.** The Lean Wronskian is `W(f,h) = f ∂ₓh - (∂ₓf)h` (`CSF.wronskianX`); the
  source (lines 379-381) uses `(∂ₓf)h - (∂ₓh)f`, its negative. The two vanish at the same points,
  so the nonvanishing statements are the same.
* **Angular constant.** The `y`-coordinate machinery is proved for a real constant `A`
  added to `V` in the equation (`IsRadialSolA`), with `A = L(L+1)` in the source-facing
  statements (`IsRadialSolXL.radialSolA_toY` turns `L(L+1)Ω²/r²` into `μ·L(L+1)`). `μ`,
  `V`, `H`, the key factorization, the square completion and the coercive density are those of
  the case `L = 0`; the angular term enters only the equation, the current `𝒥_ε` and the
  boundary terms. The master identity holds for every real `A`; `A ≥ 0` is used only in
  the sign argument.
* **Outgoing.** "Outgoing" means the asymptotic behaviour itself (e.g. `u` behaves like
  `e^{-iλ(σ-1)x}` as `x → -∞`), with a nonzero amplitude (lines 375-377, 412-417). The Lean
  conditions `CSF.OutgoingHX`, `CSF.OutgoingIX` and `CSF.AllL.ThresholdOutgoingIXL` are these
  asymptotic conditions with an amplitude `B`, including the derivative bounds.
* **Normalized solutions.** The theorem assumes no existence statement. The Wronskian
  corollary is stated for every given pair of outgoing solutions. Existence and uniqueness of the
  normalized `u_H`, `u_I` are not formalized.
* **Regularity.** Solutions are assumed twice differentiable.
* **Form of the theorem.** The theorem gives `B_I = 0` for `σ ≠ 0` and `B_H = 0` for `σ = 0` for
  every solution satisfying both asymptotic conditions, as stated in the proof of the Theorem
  (lines 975-977); this implies that no solution is outgoing at both ends. The stronger `U ≡ 0`
  holds for `Im σ > 0` and for `σ ∈ (0,1)` (`no_upper_half_plane_modes_all_L`,
  `no_real_superradiant_modes_all_L`); for `σ = 1`, `σ < 0`, `σ > 1` and `σ = 0`, deducing
  `U ≡ 0` from a vanishing amplitude would need uniqueness at one end, which is not
  formalized.
* **Threshold.** `thresholdExponentL L λ` is `1/2 - √((L+1/2)² - λ²)` for `λ ≤ L + 1/2` and
  `1/2 - i√(λ² - (L+1/2)²)` otherwise; it satisfies the indicial equation and the branch
  selection (`thresholdExponentL_indicial`, `_of_le`, `_transition`, `_of_gt`). The real
  exponent is negative when `λ² < L(L+1)`; the flux limit `tendsto_flux_pow` uses only
  `Re s ≤ 1/2` and therefore covers it.
* **Coordinates.** The core argument is proved in the compactified coordinate
  `y = 1 - 1/r ∈ (0,1)` and transferred to the source's variables (tortoise coordinate `x` with
  `dr/dx = Ω²(r)`, `r` at infinity; `IsTortoise κ r`, whose existence is proved by
  `CSF.exists_tortoise`).
* **Scope.** Every statement is for fixed `L`; nothing is uniform in `L`, and no continuity
  as `σ → 0` is asserted (the threshold value is defined separately, as in the source).

## TeX ↔ Lean correspondence (`mode_stability_all_angular_momenta.tex`)

Status: ✅ verified · ↪ verified by a different argument (see notes) · reused = declaration of the
`L = 0` library `CSF` · — not formalized.

| TeX (lines) | Lean | Status |
|---|---|---|
| Introduction: (wave_eqn), (RN_metric), mode solutions (mode_solution), Definition `def:mode_solution` with (bc_horizon), (bc_infinity), the reduction to `q₀e > 0`, the rescaling to `κ, λ, σ`, (radial_eqn) and its agreement with (eq:main-radial-equation-hintz) (line 322), (superradiant_condition) (72-276, 322) | — | — |
| (eq:Omega2-definition), (eq:tortoise-Omega), (eq:K-and-g) with `K > 0` (290-313) | `CSF.Omega2`, `CSF.IsTortoise`, `CSF.exists_tortoise`, `CSF.Kcoef`, `CSF.two_Kcoef_ge`, `CSF.gcoef`, `CSF.gcoef_pos` | reused ✅ |
| (eq:main-radial-equation-hintz) (315-321) | `CSF.AllL.IsRadialSolXL` | def |
| (energy1) (324-330) | `CSF.AllL.hasDerivAt_flux_A` (in `y`) | ✅ |
| (eq:horizon-outgoing-definition) (333-340) | `CSF.OutgoingHX` | reused def |
| (eq:long-range-phase-derivative), (eq:long-range-phase-expansion) (343-356) | `CSF.IsPhaseR`, `CSF.long_range_phase_expansion` | reused ✅ |
| (eq:short-range-phase-expansion) (357-363) | replaced by the horizon phase `CSF.phaseH`, `CSF.IsTortoise.isPhase_phaseH`, `CSF.IsPhase.sub_const`, `CSF.OutgoingHX.outgoingH_toY` | ↪ |
| (eq:infinity-outgoing-definition) (364-371) | `CSF.OutgoingIX` | reused def |
| Definition "Outgoing at both ends" (375-377); amplitudes in the proof (412-417) | the hypotheses `OutgoingHX … BH` and `OutgoingIX … BI` (`ThresholdOutgoingIXL … BI` at `σ = 0`) with arbitrary `BH`, `BI`; `BH ≠ 0`, `BI ≠ 0` in `CSF.AllL.no_solution_outgoing_at_both_ends_all_L` | def |
| Wronskian convention, (eq:Wronskian-hintz-section); independent of `x` (379-389) | `CSF.wronskianX` (opposite sign), `CSF.AllL.wronskianX_const` | ✅ |
| **Theorem** `thm:mode-stability-charged-scalar-field` (391-406) | **`CSF.AllL.no_common_outgoing_all_L`**; corollaries `CSF.AllL.mode_stability_all_L_wronskian` (the Wronskian form, (eq:no-zeroes-nonzero-sigma)) and `CSF.AllL.no_solution_outgoing_at_both_ends_all_L` | ✅ |
| Lemma `lem:localization-real-part` (420-454) | `CSF.AllL.localization_real_part_all_L`, `CSF.AllL.localization_real_part_A` | ↪ |
| (eq:outgoing-exponential-decay) and the integration of (energy1) over `ℝ` (430-444) | replaced by `CSF.AllL.tendsto_flux_left_zero_A`, `CSF.AllL.tendsto_flux_right_zero_A`, `CSF.AllL.eq_zero_of_signed_weight_A` | ↪ |
| (eq:imaginary-energy-identity) (445-449) | `CSF.AllL.flux_sub_eq_integral_A`, `CSF.AllL.imaginary_energy_identity_A` | ✅ |
| (eq:y-compactification), (eq:mu-explicit), (eq:x-to-y-derivatives) (456-477) | `CSF.rOfY`, `CSF.Omega2_div_sq_eq_mu`, `IsTortoise.mu_deriv_toY` | reused ✅ |
| (eq:y-equation) with `[V + L(L+1)]` (478-482) | `CSF.AllL.IsRadialSolA`, `CSF.AllL.IsRadialSolXL.radialSolA_toY` | ✅ |
| (eq:V-explicit), (eq:V-positive), (eq:V-endpoints) (483-500) | `CSF.gcoef_rOfY`, `CSF.Vpot_pos`, `CSF.Vpot_zero`, `CSF.Vpot_one` | reused ✅ |
| (eq:current-parameter-range), (eq:Phi-definition), (eq:phase-derivative), (eq:two-sided-conjugation) (502-534) | `CSF.IsPhase`, `CSF.exists_phase`, `CSF.IsPhaseR.comp_rOfY`, `CSF.conjFun`, `CSF.conjDeriv` | reused ✅ |
| Lemma `lem:conjugated-equation`, (eq:conjugated-equation) (536-568) | `CSF.AllL.conjugated_equation_A`, `CSF.AllL.IsRadialSolA.conjSolA` | ✅ |
| (eq:v-minus-endpoint), (eq:v-plus-endpoint) (570-582) | `CSF.OutgoingH`, `CSF.OutgoingI`, transfers `OutgoingHX.outgoingH_toY`, `OutgoingIX.outgoingI_toY` | reused ✅ |
| (eq:H-definition), Lemma `lem:key-factorization`, (eq:H-strict-positive) (585-622) | `CSF.Hfun`, `CSF.key_factorization`, `CSF.H_strict_pos` | reused ✅ |
| (eq:modified-Hintz-current) with `[V + L(L+1)]` (624-635) | `CSF.AllL.hintzCurrentA` | def |
| Lemma `lem:differential-current-identity`, (eq:current-derivative) (637-666) | `CSF.AllL.differential_current_identity_A`, `CSF.AllL.hasDerivAt_hintzCurrentA` | ✅ |
| Prop `prop:coercive-integrated-identity`, (eq:coercive-integrated-identity), (eq:V-integration-by-parts), (eq:complete-square) (668-726) | `CSF.AllL.coercive_integrated_identity_A`, `CSF.AllL.reducedCurrentA_sub`, `CSF.complete_square`, `CSF.coerciveDensity_nonneg`, `CSF.AllL.eq_zero_on_Icc_of_integral_coerciveDensity_eq_zero_A`; endpoint case in `CSF.AllL.master_mode_identity_A` | ✅ |
| (eq:v-derivative-seam), (eq:seam-current-equality) (730-742) | `CSF.seam_current_equality`, `CSF.AllL.hintzCurrentA_seam` | ✅ |
| (eq:endpoint-currents) (743-754) | `CSF.AllL.endpoint_current_left_A`, `CSF.AllL.endpoint_current_right_A` | ✅ |
| (eq:left-seam-sign), (eq:right-seam-sign), (eq:master-mode-identity) (756-784) | `CSF.AllL.master_mode_identity_A` | ✅ |
| Prop `prop:no-upper-half-plane-modes` (788-804) | `CSF.AllL.no_upper_half_plane_modes_all_L`, `CSF.AllL.no_upper_half_plane_modes_A` | ✅ |
| Prop `prop:no-real-superradiant-modes` (806-824) | `CSF.AllL.no_real_superradiant_modes_all_L`, `CSF.AllL.no_real_superradiant_modes_A`, `CSF.AllL.current_vanishing_A` | ✅ |
| (eq:usual-flux), (eq:real-frequency-fluxes) (826-843) | `CSF.flux`, `CSF.AllL.hasDerivAt_flux_A`, `CSF.AllL.real_frequency_fluxes_A` | ✅ |
| `σ < 0`, `σ > 1`, `σ = 1`; (eq:no-real-zeroes-except-threshold) (846-860) | `CSF.AllL.real_flux_exclusion_A`, `CSF.AllL.no_common_outgoing_A`, `CSF.AllL.no_common_outgoing_x_all_L` | ✅ |
| (eq:sigma-zero-asymptotic-equation) (864-870) | `CSF.AllL.sigma_zero_potential_asymptotics_L` | ✅ |
| (eq:sigma-zero-r-equation), analyticity, regular singular point, Frobenius series (871-882) | — | — |
| (eq:threshold-indicial-equation) (883-886) | `CSF.AllL.thresholdExponentL_indicial` | ✅ |
| Prop `prop:sigma-zero-threshold`, (eq:threshold-outgoing-definition), three branches, remainder (890-904) | `CSF.AllL.thresholdExponentL`, `CSF.AllL.ThresholdOutgoingIXL`; `CSF.AllL.thresholdExponentL_of_le`, `_transition`, `_of_gt`, `_branch`; `CSF.AllL.ThresholdOutgoingIXL.powOutgoingI_toY` | ✅ |
| (eq:no-zero-at-sigma-zero) (905-908) | `CSF.AllL.mode_stability_all_L_wronskian` at `σ = 0` | ✅ |
| (eq:sigma-zero-horizon-asymptotic), (eq:sigma-zero-horizon-current) (912-923) | `CSF.AllL.tendsto_flux_left_A` and constancy, inside `CSF.AllL.threshold_no_common_A` | ✅ |
| zero flux for `0 < λ ≤ L + 1/2` (926-937) | `CSF.AllL.tendsto_flux_pow`, `CSF.AllL.tendsto_flux_thresholdL` | ✅ |
| (eq:sigma-zero-infinity-current) (938-946) | `CSF.AllL.tendsto_flux_thresholdL` | ✅ |
| common-solution conclusion at `σ = 0` (947-950) | `CSF.AllL.threshold_no_common_A`, `CSF.AllL.threshold_no_common_x_all_L` | ✅ |
| Remark `rmk.choice`, meaning of the threshold convention (953-963) | interpretive; the flux signs are `CSF.AllL.tendsto_flux_thresholdL` | — |
| Proof of the Theorem: `B_I = 0` for `σ ≠ 0`, `B_H = 0` for `σ = 0` (966-977) | `CSF.AllL.no_common_outgoing_cases_all_L` (used by the proof of the theorem) | ✅ |
| Proof of the Theorem: the Wronskian form (978-992) | `CSF.AllL.mode_stability_all_L_wronskian`, using `CSF.AllL.eventually_ne_zero_of_outgoingHX`, `CSF.AllL.ThresholdOutgoingIXL.eventually_ne_zero`, `CSF.AllL.eq_const_mul_on_Iio_of_wronskianX_zero`, `CSF.AllL.eq_const_mul_on_Ioi_of_wronskianX_zero`, `CSF.AllL.outgoingHX_congr_mul`, `CSF.AllL.ThresholdOutgoingIXL.congr_mul` | ✅ |

## Arguments that differ from the TeX

1. **Localization lemma.** The source integrates (energy1) over `ℝ`, using the exponential decay
   (eq:outgoing-exponential-decay). The formal proof integrates the flux identity over compact
   subintervals and shows that the flux tends to `0` at both ends (the angular term is real and
   does not enter the imaginary part).
2. **Horizon phase.** The source states (eq:short-range-phase-expansion), with remainder
   `O(e^{κx})`, and uses it to pass from (eq:horizon-outgoing-definition) to
   (eq:v-minus-endpoint). The formal proof uses a phase adapted to the horizon (`CSF.phaseH`),
   shows that any two phases in the sense of (eq:phase-derivative) differ by a constant
   (`CSF.IsPhase.sub_const`), so that the endpoint conditions change only by a nonzero factor,
   and needs no rate.

## Remarks on the source

No mathematical error was found. The direction "`𝒲 ≠ 0` ⟹ no solution outgoing at both ends" of
the Theorem's "Equivalently" uses uniqueness of the outgoing solutions at one end, which the
source takes from the literature (footnote at line 148); the formal proof needs only the other
direction (lines 978-992).

## Not formalized

* **The introduction** (lines 72-276): the wave equation, mode solutions and
  Definition `def:mode_solution`, the reduction to `q₀e > 0`, the rescaling to `κ, λ, σ`, and the
  agreement of (radial_eqn), (bc_horizon), (bc_infinity) with the normalized formulation of
  Section 1 (lines 322, 371).
* **Existence of the normalized outgoing solutions**: the horizon solution for every
  `Im σ ≥ 0` (including the bounded solution at `σ = 1`), the long-range solution at infinity for
  `σ ≠ 0`, and the threshold Frobenius solution in all three branches (lines 148, 333, 364,
  878-882). The theorem needs no existence statement; the Wronskian corollary holds for any given
  solutions and is vacuous only if they do not exist.
* **Uniqueness of outgoing solutions up to scalars**: needed only for the converse direction of
  the Theorem's "Equivalently", and for `U ≡ 0` when `σ ∈ {0} ∪ [1, ∞) ∪ (-∞, 0)`.
* (eq:short-range-phase-expansion) as stated, (eq:sigma-zero-r-equation) and the
  regular-singular-point discussion (not used by the formal proof).
* The interpretive remarks: the regularity of the combined phase at `y₀` (line 531), the
  footnote and remark after the Theorem (lines 404, 408-410), and Remark `rmk.choice`
  (lines 953-963).
