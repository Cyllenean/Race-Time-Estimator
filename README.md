# CYLLENEAN — Race Time Estimator

> **Speed · Precision · Minimalism**

An interactive, browser-based race time estimator for CO₂-powered STEM Racing (F1 in Schools) cars. Built by [Team Cyllenean](https://github.com/Cyllenean) at the Australian International School Hong Kong.

---

## Overview

**1.3.0** of the Cyllenean race time estimator simulates the longitudinal dynamics of a CO₂ dragster over a 20 m track. It solves the coupled system

```
dx/dt = v
m_eq(t) · dv/dt = F_thrust − F_drag − F_bearing − F_rolling − F_tether
dJ/dt = F_thrust
```

using an explicit fourth-order Runge–Kutta integrator (`dt = 0.5 ms`). The model accounts for:

- **Time-varying thrust** from a fitted CO₂ discharge curve
- **Mass depletion** as CO₂ is expelled (tracked via cumulative impulse)
- **Aerodynamic drag** (quadratic in velocity)
- **Rolling resistance** (constant, linear and quadratic terms in velocity)
- **Bearing friction** (Coulomb + viscous + quadratic torque model, per bearing)
- **Tether / guide resistance** (constant, linear and quadratic terms)
- **Wheel rotational inertia** via an equivalent translational mass

The estimator is intended to be a *calibrated* tool. Every coefficient has a physically meaningful default, but you are expected to tune them against measured data from your own car.

---

## Features

- **100% client-side** — no server, no build step, no dependencies. One HTML file.
- **Text-editable inputs** for every parameter, with step-arrow precision down to `1e-8`.
- **Live simulation** — every keystroke re-runs the full RK4 integration in under 20 ms.
- **Four diagnostic charts** rendered on canvas:
  - Force balance over time (thrust, drag, bearing, rolling, tether)
  - Velocity vs time
  - Velocity vs position along the track
  - Dynamic mass budget (equivalent mass, cartridge mass, remaining CO₂)
- **Summary statistics** — race time, peak velocity, peak acceleration, peak thrust, total impulse, mass budget.
- **Organised inputs** — *Car Configuration* at the top, *Model Parameters* below.
- **Auto-calculated chassis mass** so the mass budget never double-counts.
- **Branded UI** matching the Cyllenean visual identity (Jost typeface, gold-on-black palette, blueprint grid).

---

## Download & Run

The tool is a single, self-contained HTML file. There is no installation, no `npm install`, no compilation and no hosting required.

### Steps

1. Download **[`Race-Time-Estimator.html`](./Race-Time-Estimator.html)** from this repository:
   - Click the file in the repository listing, then click the **Download raw file** button (the ⬇ icon in the top-right of the file viewer).
   - Or, if you have cloned the repository, the file is already on your machine.

2. Open the downloaded file in any modern browser:
   - **macOS:** double-click the file, or drag it onto the Safari / Chrome / Firefox icon.
   - **Windows:** double-click the file, or right-click → *Open with* → your browser.
   - **Linux:** `xdg-open Race-Time-Estimator.html`, or open it from your file manager.

3. The estimator runs entirely offline. You can move, rename, or email the file, and it will keep working.

> **Note:** Do **not** open the file by pasting a `file:///` path into some editors — open it in a browser, not in a code editor.

### Cloning instead

If you'd rather keep it under version control:

```bash
git clone https://github.com/Cyllenean/Race-Time-Estimator.git
cd Race-Time-Estimator
open Race-Time-Estimator.html    # macOS
# or
start Race-Time-Estimator.html   # Windows
# or
xdg-open Race-Time-Estimator.html  # Linux
```

---

## Input Reference

### Car Configuration

The upper section of the interface. These describe **your physical car**.

#### Vehicle Mass

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Total Car Mass (no cartridge) | g | 50.0 | Regulation minimum is typically 50 g |
| Number of Wheels | — | 4 | Usually 4 |
| Wheel Mass (each) | g | 3.5 | Per wheel |
| Axle Mass (total) | g | 1.0 | Sum of all axles |
| Chassis Mass | g | *derived* | Auto-calculated; read-only |

> **Chassis Mass** = Total Car Mass − (N × Wheel Mass) − Axle Mass. This is what remains after subtracting wheels and axles from your measured car weight.

#### Wheel & Axle Geometry

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Wheel Diameter | mm | 28.0 | Outer diameter |
| Wheel Inertia Factor | — | 0.5 | `0.5` = solid disc, `1.0` = thin ring |
| Axle Diameter | mm | 3.0 | Used for bearing geometry |

#### CO₂ Cartridge

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Empty Cartridge Mass | g | 15.0 | The metal shell alone |
| Initial CO₂ Mass | g | 8.0 | Standard 8 g cartridge |
| Nozzle Diameter | mm | 4.0 | Affects the fitted thrust curve |

#### Aerodynamics

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Drag Coefficient (Cd) | — | 0.60 | Typical for a small dragster |
| Frontal Area | mm² | 2000 | Cross-sectional area |
| Air Density (ρ) | kg/m³ | 1.225 | Sea-level ISA |

### Model Parameters

The lower section. These describe the **physics and coefficients** of the model itself.

#### Thrust Curve

The thrust curve is

```
F(t) = scale · K · (t − T₀)^P · exp(−(t − T₀) / τ)
```

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Thrust Scale | — | **0.75** | **Calibrated value** — tune against measured times |
| K | N | 67.524 | Fitted amplitude |
| T₀ | s | 0.002238 | Ignition delay |
| P | — | 0.51593 | Fitted exponent |
| τ | s | 0.11582 | Time constant |

The total impulse delivered by the cartridge is

```
J_total = scale · K · τ^(P+1) · Γ(P+1)
```

#### Bearing Friction

Per-bearing torque is modelled as:

```
T_b(ω) = T_coulomb + c₁·ω + c₂·ω²
```

| Input | Unit | Default |
|-------|------|---------|
| Number of Bearings | — | 4 |
| Coulomb Torque | N·m | 1.0e-4 |
| Viscous Coefficient | N·m·s | 1.0e-8 |
| Quadratic Coefficient | N·m·s² | 0.0 |

#### Rolling Resistance

```
F_rr = N · (Crr₀ + Crr₁·v + Crr₂·v²)
```

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Crr (constant) | — | 0.02 | Typ. 0.015–0.025 on smooth track |
| Crr (linear) | s/m | 0.0 | Usually negligible |
| Crr (quadratic) | s²/m² | 0.0 | Usually negligible |

#### Tether / Guide Resistance

```
F_tether = F₀ + c₁·v + c₂·v²
```

| Input | Unit | Default |
|-------|------|---------|
| Constant Force | N | 0.01 |
| Linear Coefficient | N·s/m | 0.001 |
| Quadratic Coefficient | N·s²/m² | 0.0001 |

#### Numerical & Environment

| Input | Unit | Default | Notes |
|-------|------|---------|-------|
| Track Length | m | 20.0 | Standard race distance |
| Gravity | m/s² | 9.80665 | Standard gravity |
| Max Simulation Time | s | 5.0 | Upper bound for the solver |

---

## Calibration

The **Thrust Scale** is the single most important calibration parameter. The model was fitted from a specific CO₂ discharge dataset; other cartridges, nozzles and temperatures will produce different thrust profiles. The default value of **0.75** was chosen to match the team's measured Development Class race time of **1.285 s** on a standard 20 m track.

### Procedure

1. Run the tool with a known car setup and record the predicted race time.
2. Race the same car and record the measured time.
3. Adjust **Thrust Scale** until the predicted and measured times agree.
4. For a more robust fit, repeat with several runs and average the scale.

Because drag, rolling resistance and tether losses are all non-linear, the relationship between scale and race time is not perfectly linear — iterate two or three times for convergence.

### Other coefficients

- **Crr** dominates the launch phase. If your predicted time is too *short*, raise `Crr` first.
- **Bearing Coulomb torque** dominates the middle phase. If the car "coasts" too long in the simulation, raise this term.
- **Tether resistance** matters most on tracks where the guide line is under tension. If your car is noticeably slowed near the finish, raise the linear/quadratic tether terms.
- **Cd** and **frontal area** only start to matter above ~15 m/s. For sub-15 m/s cars, they contribute less than 5% of the total retarding force.

---

## Physics Notes

### Why is the equivalent mass larger than the physical mass?

For a wheel of moment of inertia `I` and radius `r`, the rotational kinetic energy at translational speed `v` is `½ I (v/r)²`. Dividing by `½ v²` gives the **rotational equivalent mass**:

```
m_rot = I / r²
```

For four identical wheels, `m_rot_total = 4 I / r²`. This is added to the translational mass in the equations of motion because the wheels must be spun up as the car accelerates. In the defaults, with a solid-disc inertia factor of `0.5`, the four wheels contribute roughly **7 g** of equivalent mass on top of the ~73 g on the track — so the effective mass the thrust must accelerate is closer to **80 g**.

### Why is the chassis mass derived instead of entered directly?

The regulations specify a **minimum car mass without the cartridge** (typically 50 g). This includes the body, wheels, axles and all fittings. Entering the total and subtracting the wheels and axles avoids accidentally double-counting and keeps the mass budget unambiguous.

### What is `J_total`?

The CO₂ mass depletion model estimates the remaining CO₂ as

```
m_CO₂(J) = m_CO₂,0 · (1 − J / J_total)
```

where `J` is the cumulative thrust impulse delivered so far. `J_total` is the integral of the fitted thrust curve over all time (scaled by `Thrust Scale`). This is an approximation: real cartridges do not expel CO₂ at a perfectly linear rate with respect to impulse, but the deviation is small over a 20 m run.

---

## Project Structure

```
.
├── Race-Time-Estimator.html   # The entire tool (self-contained)
└── README.md
```

Everything — physics, integration, plotting, UI — lives inside the HTML file. There is no build step, no bundler and no external JavaScript.

The only external dependencies are:

- **Google Fonts** — Jost typeface
- *(Optional)* **Font Awesome** if you add icons in your fork

Both are loaded from a CDN. If you need a fully offline copy, download the fonts locally and update the `<link>` tags in the HTML.

---

## Browser Support

The tool uses standard HTML5, CSS3 and ES6 JavaScript (including `canvas`, `requestAnimationFrame`, `IntersectionObserver` and template literals). It has been tested on:

- Chrome / Edge 110+
- Firefox 110+
- Safari 16+

Older browsers may render the layout but may not run the simulation correctly.

---

## License

Released under the **MIT License**. See `LICENSE` for details.

You are free to use, modify and redistribute this tool, including for commercial purposes, provided the copyright notice and licence text are retained. Attribution is appreciated but not required.

---

## Contact

**Cyllenean**
Australian International School Hong Kong
3A Norfolk Road, Kowloon Tong, Kowloon, Hong Kong

- Email: [cyllenean.aishk@gmail.com](mailto:cyllenean.aishk@gmail.com)
- Instagram: [@cyllenean](https://www.instagram.com/cyllenean/)
- LinkedIn: [Team Cyllenean](https://www.linkedin.com/in/team-cyllenean-5725653b0/)
- GitHub: [github.com/Cyllenean](https://github.com/Cyllenean)

---

<p align="center">
  <strong>Speed · Precision · Minimalism</strong><br>
  Copyright © 2026 Cyllenean. All Rights Reserved.
</p>
