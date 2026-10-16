# NeuroPulse CableLab v3.0 — Enhanced, backward-compatible browser workbench

**A single-file, offline-capable multi-compartment Hodgkin–Huxley simulator for education and numerical experimentation.**

> **Not a medical device; not validated for neurostimulation, clinical dosing, in-vivo axonal conduction, or patient-specific predictions.** Parameters such as `gAx` and synthetic synaptic conductances are phenomenological; they are not derived from validated cable geometry.

## Run

1. Open `index.html` in Chrome, Edge, Firefox or another modern browser. It contains its own CSS and JavaScript; no remote service, account, npm or CDN is required.
2. For reliable browser storage, use a local static server rather than `file://`: run `python -m http.server 8000` in the package folder, then visit `http://localhost:8000`.
3. On the left, configure the cable, integrator, stimulus or protocol, then press **Run**. The right-hand panel scrolls through the live charts. Press **Run self-tests** in **Scientific Verification** for an integrity report. **Compare dt / dt÷2** compares isolated RK4 solutions at two step sizes.
4. Use existing CSV / WAV / snapshot / protocol / manifest tools as before; v3 adds **Export report JSON**, **Export live trace CSV**, and **Export complete state JSON**.

The separately extracted `engine.reference.js` is a source reference for review. It is **already embedded in `index.html`**; do not load it a second time or replace it with `original/temp_older.js`.

## Preservation analysis

The primary feature baseline is the **complete 5,844-line HTML attachment** (not the independent 2,934-line JavaScript file, which is older and lacks substantial later functionality). The historic full HTML included 243 unique element IDs and 124 HTML inputs/buttons/selects/textareas. This release retains all 243 IDs and increases the interactive input/control count to 134; 15 original canvas elements remain present. The prior page had a fatal initial-render error and malformed panel nesting, so merely comparing screenshots to it was not sufficient.

| Existing feature family | Status in v3 |
| --- | --- |
| Classic multi-compartment HH (V,m,h,n), M-current (w), cable coupling and spatial targeting | Kept; myelin factors now enter solver; original phenomenological discretization remains |
| Euler, RK2 Heun, RK4; numerical configuration and sliders | Kept; adds Rush–Larsen gated update; changing dt no longer erases physical state |
| Temperature and gradient, Na/K blockers, AIS, voltage clamp | Kept; probe monitor accounts for all ionic terms and AIS/myelin scaling; exported v3 advanced parameters |
| Manual DC, pulses, sine, filtered noise, seeded RNG | Kept |
| NeuroProtocol DSL with blocks, loops, compile / format / templates | Kept; DSL remains an educational simulator script format |
| PID autopilot for rate, sonification, WAV export | Kept (basic smoke verified; detailed signal-by-signal parity not formally proven) |
| Kymograph, probe trace, spatial snapshot, ionic-gate visualization | Kept; fatal duplicate-function bug repaired |
| Spike raster, heatmap, phase plane, F–I, isometric 3D, PSD, I–V, control/event diagnostics | Kept; chart-container nesting fixed; missing control/event drawing functions implemented |
| AMPA, NMDA, GABA and synapse events | Kept; synaptic current now affects V *during* integration, rather than being overwritten before use |
| Myelination, node spacing, spike analytics | Kept; internodal scalings now applied; time step auto-tuned to a conservative axial bound when possible |
| F–I and bifurcation scans | Kept, now running an independent deterministic submodel rather than mutating active state; outdated bifurcation call repaired |
| Preset import/export, snapshot manager, run manifests, CSV recording | Kept; advanced model settings added to v3 state export |
| Keyboard shortcuts, cinema mode, collapsible sections, colormaps, particle background | Kept; visual layout and access improved |

**What feature-preservation tests prove:** all original DOM IDs exist, key canvases are present, the updated application boots cleanly, and a selected set of workflows executes. They do **not** prove every combination of all 134 controls behaves identically, nor do they prove the numerical physiology is correct.

## Concrete fixes to the uploaded original

1. **Fatal render exception:** two declarations named `computeProbeCurrents` existed. JavaScript declaration hoisting caused the later, less complete function to replace the first. The view tried to call `.toFixed` on a missing `IM` value. The legacy duplicate is no longer shadowing the correct implementation.
2. **Malformed DOM:** the original `.viz` and `.vizBottom` were peers of `.right` rather than inside it, leaving a giant empty viewer; HTML nesting was corrected.
3. **Missing drawing functions:** `drawControlAnalytics()` and `drawSpikeAnalytics()` were called but undefined. Both now render their plots.
4. **Synapse simulation:** synaptic currents are integrated into the membrane ODE evaluation. The old post-frame addition to `Iinj` was overwritten at the start of the next substep.
5. **Myelin simulation:** explicit internodal ion-channel / leak / capacitance factors now enter the equation. Auto-retuning reduces `dt` for the reduced effective internodal capacitance; a warning and pause appear if a required `dt` is below the supported range.
6. **Bifurcation:** the old scanner passed the wrong `computeDerivs` signature. It now uses a separate solver and detects whether 0 and max I bracket spike onset. It measures a **finite-time operational threshold**, not a mathematical bifurcation point in the rigorous dynamical-systems sense.
7. **F–I curves:** sweeps no longer reset, mutate or perturb the user's active cable state. They are explicitly deterministic and omit active synapse events, PID, stochastic noise and DSL during the sweep.
8. **Time step changes:** modifying dt / plot-window size now retains the live physical state, rather than allocating uninitialised state arrays.
9. **Numerics:** HH rate singularity handling uses `Math.expm1`; an experimental Rush–Larsen analytical gate update is available. A nonfinite / runaway potential guard pauses execution.
10. **Provenance:** v3 JSON retains previously omitted advanced controls: pharmacology scales, voltage clamp, spatial temperature gradient, channel noise, synthetic plasticity toggles, AIS, synaptic weights and RNG state. Old version-2 presets remain accepted.

## Mathematical model and important limits

Membrane voltage updates approximately follow:

\[
C_{m,i}\frac{d V_i}{dt}=I_{\mathrm{ext},i}+I_{\mathrm{syn},i}+g_{\mathrm{Ax}}(V_{i-1}-2V_i+V_{i+1})-I_{\mathrm{Na},i}-I_{\mathrm{K},i}-I_{\mathrm{L},i}-I_{\mathrm{M},i}.
\]

The Hodgkin–Huxley gating process uses
\(dx/dt=\phi(T)(\alpha_x(V)(1-x)-\beta_x(V)x)\).
For a frozen voltage over one numerical step, Rush–Larsen integrates that gate analytically:
\(x(t+\Delta t)=x_\infty+(x(t)-x_\infty)\exp[-\phi(\alpha+\beta)\Delta t]\).
The model's *voltage* part remains explicitly integrated, so Rush–Larsen is **not** an unconditionally stable implicit-cable method. White channel noise is implemented as a heuristic perturbation, not a calibrated stochastic channel-state model.

The displayed approximate explicit axial stability index is \(2 g_\mathrm{Ax}\Delta t/(C_m c_\mathrm{myelin})\). A value below 1 is merely a **necessary axial check** for this simplified discretization, not proof of stability of the coupled nonlinear model or of accuracy. Reducing dt or using an external reference simulator remains essential before drawing scientific conclusions.

The supplied `dx_um` is used for a geometric distance / displayed velocity estimate; the baseline `gAx` is **not** derived from membrane area, internal resistivity and `dx`. Therefore conduction-velocity claims should not be interpreted as physically calibrated. The myelin implementation is likewise a simple internodal scaling exercise, not a full node-of-Ranvier morphology.

The F–I and threshold analysis use an isolated deterministic submodel with the global HH + M-current, pharmacology, AIS, spatial temperatures, boundary conditions and phenomenological myelin, but intentionally omit active synapse queues, PID, DSL, voltage-clamp, channel noise and STDP. It does not mutate the live state. The dt-convergence tool currently compares only final voltage snapshots, not full trajectories, spike timing or error norms against an external solver.

## Verification (16 October 2026)

The included `regression_report.json` was generated by a Chromium + Playwright browser smoke harness. It records **12/12 passing smoke/regression checks**, no uncaught browser page errors, **243/243 existing DOM IDs preserved**, and **10/10 internal numerical self-tests passing**. Browser tests exercised clean boot without localStorage permission, panel nesting, original canvases, simulation advancement, gate integrator execution, F–I isolation, the threshold scanner and three kinds of export. The original uploaded HTML was separately tested and threw a `TypeError` during first render.

The tests cover selected workflows, **not exhaustive equivalence or validation against NEURON or Brian2**. Integration choice can affect speed and accuracy; no universal speedup is claimed.

## Research anchors and engineering roadmap

- **Brian2 numerical integration methods:** explicit Euler, RK2, RK4, and exponential Euler for conditionally linear HH-type dynamics. This supports adding Rush–Larsen-style exact gate updates while retaining traditional integrators: https://brian2.readthedocs.io/en/2.10.1/examples/IF_curve_Hodgkin_Huxley.html and https://brian2.readthedocs.io/en/2.6.0/user/numerical_integration.html .
- **NEURON compartmental cable equation and geometry:** https://www.neuronsimulator.org/en/latest/nmodl/transpiler/contents/cable_equations.html . Next implement calibrated axial resistance and morphology-based mesh construction, with independent NEURON fixtures for golden tests.
- **NEURON variable-step CVODE:** https://www.neuronsimulator.org/en/9.0.0/nmodl/transpiler/contents/cvode.html . Future work: adaptive tolerances or semi-implicit/implicit axial solve; don't present simple explicit dt guards as a substitute.
- **NEURON d-lambda spatial discretization:** https://neuron.yale.edu/neuron/static/docs/d_lambda/d_lambda.html . Future work: spatial-convergence testing, voltage error vs N, appropriate segmentation.
- **WebGPU:** https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API . This is a future acceleration option, **not implemented in v3**. WebGPU is not universally supported and often requires a secure context, so a CPU-first standalone HTML file remains the compatibility default.

Other research-note proposals (automated protocol synthesis, RL agents, surrogate neurons, neural mesh reduction and combining with the separate Combined-Agents project) are **ideas, not shipped implementations** in this package. The 3D mesh-reduction research section in the attached notes concerns a different project and is not a NeuroPulse feature.

## File layout

- `index.html` — complete self-contained application.
- `engine.reference.js` — the exact JavaScript extracted from `index.html`, for code review only.
- `README.md` — this document.
- `regression_report.json` — machine-readable test outcomes.
- `CHANGELOG.md` — key changes and remaining limitations.
- `original/` — archived copies of the user-provided original files, to enable line-by-line comparison.

**Recommended next milestone:** produce a matching compartmental HH axon reference simulation in NEURON or Brian2, then compare voltage traces, spike timing, conduction velocity and energy across N/dt. Only after those tests should the system claim quantitatively credible biophysical fidelity or measured performance gains.
