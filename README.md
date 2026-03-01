# NeuroPulse CableLab

A browser-based neuroengineering workbench that upgrades classic Hodgkin–Huxley into a **spatial axon (multi-compartment cable model)** with **real-time propagation/kymograph visualization**, a **NeuroProtocol stimulation DSL**, and **dataset export (CSV)** — all in a lightweight HTML/JS bundle. ([GitHub][1])

---

## What’s inside

NeuroPulse CableLab is designed for interactive neurophysiology experimentation:

* **Multi-compartment HH “cable” axon** (N compartments) with adjustable **axial coupling** `gAx`, selectable boundary conditions (sealed/clamped), and configurable compartment spacing `dx`. ([GitHub][1])
* **Propagation analytics**: real-time **space–time voltage kymograph** plus probe traces and spatial snapshots. ([GitHub][1])
* **NeuroProtocol DSL**: script stimulus blocks (dc, pulse trains, ramps, chirps, noise, etc.) with **spatial targeting** and Gaussian spread. ([GitHub][1])
* **Closed-loop Autopilot**: PID-based rate control at the probe by adjusting DC offset or pulse amplitude. ([GitHub][1])
* **Neuro-sonification**: probe-driven audio modes (ionic synth / spike percussion / FM gating) with a **“Render WAV”** export path. ([GitHub][1])
* **Recording + CSV export**: record time series and export a dataset including probe voltage, stimulus, firing rate, velocity estimate, and checkpoint voltages along the cable. ([GitHub][1])
* **Extras**: synaptic inputs (AMPA/NMDA/GABA), myelination mode, F–I sweep generator, rheobase/threshold finder, phase-plane and spectral tooling, snapshot manager (JSON). ([GitHub][1])

---

## Quick start

### Option A — run locally (recommended)

Some browser APIs (clipboard import/export, file download behavior) work more reliably on `http://localhost` than from a `file://` URL.

```bash
git clone https://github.com/kai9987kai/NeuroPulse-CableLab.git
cd NeuroPulse-CableLab

# Python 3
python -m http.server 8000
```

Open `http://localhost:8000` and you should see the CableLab UI.

### Option B — open the file directly

You *can* try opening `index.html` directly in a browser, but features like clipboard read can be blocked depending on browser security settings. ([GitHub][2])

---

## Repository layout

* `index.html` — the UI/controls surface for CableLab. ([GitHub][3])
* `temp.js` — the simulation + rendering engine (HH state arrays, DSL compiler/runtime, kymograph canvases, export handlers, etc.). ([GitHub][2])
* `LICENSE` — MIT. ([GitHub][4])

---

## Model + numerics (implementation notes)

### Cable coupling

The “cable equation” is implemented as coupling currents between adjacent compartments (discrete Laplacian form):

* `Iax(i) = gAx * [(V(i-1) - V(i)) + (V(i+1) - V(i))]`
* sealed ends are handled by mirroring the boundary; clamped ends hold near rest. ([GitHub][1])

### Hodgkin–Huxley core + stability helpers

The JS engine includes classic HH rates, temperature scaling via `phi()` using `Q10`, and a numerically stable `vtrap()` helper to avoid division blow-ups near zero. ([GitHub][2])

### Integrators + stiffness

You can switch integrators (Euler / RK2 Heun / RK4). Multi-compartment coupling increases stiffness; the UI explicitly warns that raising `gAx` typically requires reducing `dt` for stability. ([GitHub][1])

---

## NeuroProtocol DSL

### Mental model

A protocol is a list of **time blocks**. Each line defines:

* duration (`300ms`, `2s`, or a number interpreted as ms),
* a stimulus type,
* optional parameters,
* optional spatial targeting: `at <index>` and `spread <sigma>`.

The DSL compiler parses `block ...` lines, ignores `#` comments, and builds a timed segment list executed sequentially (optionally looping). ([GitHub][1])

### Built-in examples (from the UI)

```text
block 300ms dc 0
block 700ms dc 8 at 8 spread 0
block 800ms pulse amp 18 period 20 duty 5 at 8
block 1200ms ramp from 0 to 12 at 8 spread 2
block 1500ms chirp amp 3 f0 2 f1 30 at 8
block 800ms noise amp 0.6 lp 40 at 8
```

Supported types include: `off, dc, hold, pulse, sine, noise, ramp, chirp`. ([GitHub][1])

### Spatial targeting

Targeting supports a **Gaussian injection profile**:

* `Iinj[i] = I0 · exp(-(i-c)^2 / (2σ^2))`
* if `σ = 0`, the stimulus applies only to the center compartment. ([GitHub][1])

---

## Data export

### CSV recording

Enable Recording, run the sim, then export. The CSV includes columns like:

* `t_ms`, `V_probe`, `I0`, `rate_Hz`, `vel_mps`
* plus multiple checkpoint voltages across the cable. ([GitHub][1])

### Example analysis (Python)

```python
import pandas as pd

df = pd.read_csv("cablelab_YYYY-MM-DD_HH-MM-SS.csv")
print(df.head())
print(df[["t_ms", "V_probe", "rate_Hz", "vel_mps"]].describe())
```

---

## Keyboard shortcuts

* `Space` Run/Pause
* `R` Reset
* `A` Toggle Audio
* `C` Compile DSL
* `P` Panic ([GitHub][1])

---

## Safety / intended use

This project is an educational/engineering sandbox for simulation and visualization. It is **not** a medical device and should not be used for clinical decision-making.

---

## Contributing

Issues and pull requests are welcome. If you add new protocol blocks, keep the DSL grammar consistent with the existing `block <duration> <type> ...` structure and update the UI help text accordingly. ([GitHub][2])

---

## License

MIT License (Copyright © 2026 Kai Piper). ([GitHub][4])

[1]: https://raw.githubusercontent.com/kai9987kai/NeuroPulse-CableLab/main/index.html "NeuroPulse CableLab — Multi-Compartment HH + NeuroProtocol DSL + CSV Export"
[2]: https://raw.githubusercontent.com/kai9987kai/NeuroPulse-CableLab/main/temp.js "raw.githubusercontent.com"
[3]: https://github.com/kai9987kai/NeuroPulse-CableLab/tree/main "GitHub - kai9987kai/NeuroPulse-CableLab"
[4]: https://raw.githubusercontent.com/kai9987kai/NeuroPulse-CableLab/main/LICENSE "raw.githubusercontent.com"
