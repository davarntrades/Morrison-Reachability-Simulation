<div align="center">

# Morrison Reachability Simulation™

## A Toy System Demonstrating Structural Safety via Reachability Constraints

![Type](https://img.shields.io/badge/Type-Interactive_Simulation-0075ca?style=flat-square)
![Framework](https://img.shields.io/badge/Framework-Morrison_Framework™-0075ca?style=flat-square)
![Foundation](https://img.shields.io/badge/Foundation-Reachability--Based_Safety-2d6a2e?style=flat-square)
![Language](https://img.shields.io/badge/Language-React_·_JavaScript-0075ca?style=flat-square)
![Patent](https://img.shields.io/badge/Patent-GB2600765.8-0075ca?style=flat-square)
![©](https://img.shields.io/badge/©_Davarn_Morrison-555555?style=flat-square)

-----

*“Safety is not what a system says — it is what it can reach.”*

*— Davarn Morrison, 2026*

</div>

-----

This work builds upon the patented pre-semantic trajectory governance framework.

-----

## Overview

This work presents a discrete, interactive simulation demonstrating a structural approach to safety in intelligent systems. Rather than modifying outputs post-generation (e.g. RLHF or filtering), safety is enforced by constraining the reachable state space of the system.

The core principle is:

```
A system is safe if and only if unsafe states are unreachable.

ℛ(t) ∩ Ω = ∅
```

The simulation makes this principle visible: the reachable set ℛ(t) is computed at each timestep under different constraint regimes, and the geometric relationship between ℛ(t) and Ω is observable in real time.

-----

## System Design

The simulation implements a 20×20 discrete state space with:

|Parameter          |Value                                       |
|:-----------------:|:------------------------------------------:|
|State space        |20 × 20 grid (400 states)                   |
|Initial state      |x₀ = (3, 14)                                |
|Transition dynamics|8-directional movement (Moore neighbourhood)|
|Forbidden region   |Ω = 5×5 block of unsafe states              |
|Time horizon       |t = 0 to t = 12                             |

At each timestep, the system computes the reachable set ℛ(t) — the complete set of all states accessible from x₀ under the active constraint regime.

-----

## Modes of Operation

-----

### Mode 1 — Unconstrained (Baseline)

All transitions permitted. ℛ(t) expands freely under the system dynamics with no constraint enforcement.

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   ℛ(t) ∩ Ω ≠ ∅                                          ║
║                                                          ║
║   Unsafe states are inevitably reached.                  ║
║   No constraint on transition function.                  ║
║   Failure is a question of when, not if.                 ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

-----

### Mode 2 — One-Step Constraint

Immediate transitions into Ω are blocked. The safe action set is defined as:

```
A_safe(x) = { u ∈ U | F(x, u) ∉ Ω }
```

This ensures local safety: no single action can move the system directly into the forbidden region. However, it does not account for future trajectories — a sequence of individually safe actions may still lead to Ω.

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   One-step safety:                                       ║
║   F(x_t, u_t) ∉ Ω                                       ║
║                                                          ║
║   Local guarantee only.                                  ║
║   Does not prevent multi-step convergence to Ω.          ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

-----

### Mode 3 — Multi-Step Reachability Constraint (Key Contribution)

Introduces lookahead-based safety enforcement. The system computes a backward reachable unsafe set: all states that can reach Ω within N steps are excluded from the admissible action space.

```
A_safe(x) = { u ∈ U | Reach⁺(F(x, u), N) ∩ Ω = ∅ }
```

This transforms Ω into a structurally expanded forbidden region, removing not just unsafe states, but all states that can lead to unsafe states within the lookahead horizon.

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   Multi-step safety:                                     ║
║   Reach⁺(F(x_t, u_t), N) ∩ Ω = ∅                       ║
║                                                          ║
║   Structural guarantee.                                  ║
║   Ω becomes unreachable within the defined horizon.      ║
║   The reachable set deforms around the forbidden region. ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

The lookahead N is adjustable (1–5 steps). As N increases, the excluded region grows and ℛ(t) shrinks further — more states are removed because they eventually lead to Ω even if they are currently safe.

-----

## Key Mechanism

The multi-step constraint operates by computing backward reachability from Ω:

```
Ω_expanded(N) = { x ∈ X | Reach⁺(x, N) ∩ Ω ≠ ∅ }
```

The safe action set then excludes all transitions into Ω_expanded(N):

```
A_safe(x) = { u ∈ U | F(x, u) ∉ Ω_expanded(N) }
```

This results in:

- Deformation of ℛ(t) — the reachable set visibly reshapes around the expanded forbidden region
- Removal of unsafe trajectories before they occur — not after
- Guaranteed safety within the defined horizon

-----

## Observed Behaviour

|Mode            |ℛ(t) ∩ Ω   |Outcome                                     |
|:--------------:|:---------:|:------------------------------------------:|
|Unconstrained   |≠ ∅        |ℛ(t) intersects Ω — failure inevitable      |
|One-step        |= ∅ (local)|Immediate violations blocked                |
|Multi-step (N=3)|= ∅        |Ω structurally unreachable — safety achieved|

Δ|ℛ(t)| quantifies the deformation induced by constraints. The difference in reachable set cardinality between unconstrained and constrained modes at each timestep provides a direct numerical measure of how governance reshapes the system’s accessible futures.

-----

## Deformation Overlay

The simulation includes a deformation overlay mode that highlights the difference between the unconstrained and constrained reachable sets at each timestep. States that were reachable without constraints but are excluded under the active constraint regime are rendered in a distinct colour, making the geometric reshaping of ℛ(t) directly visible.

This visualises the core claim: governance does not filter outputs — it reshapes what the system can become.

-----

## Technical Significance

This simulation demonstrates three properties:

```
════════════════════════════════════════════════════════════
  1. MINIMAL IMPLEMENTATION
     Reachability-based safety in ~200 lines of code.
     Zero dependencies beyond React.

  2. STRUCTURAL VS BEHAVIOURAL SAFETY
     The difference between:
       Probabilistic: unsafe states unlikely
       Structural:    unsafe states unreachable

  3. BRIDGE BETWEEN FIELDS
     Connects:
       AI safety          (alignment, governance)
       Control theory     (reachability, barrier functions)
       Dynamical systems  (state space, trajectories)
════════════════════════════════════════════════════════════
```

-----

## Next Steps

|Extension                                    |What It Achieves                                          |
|:-------------------------------------------:|:--------------------------------------------------------:|
|Continuous state spaces                      |Move from grid to manifold — closer to real systems       |
|Neural network policy integration            |Apply constraints to learned policies, not toy dynamics   |
|Hamilton-Jacobi reachability methods         |Formal multi-step safety with continuous-time guarantees  |
|Computational trade-off analysis             |Characterise lookahead N vs safety guarantee strength     |
|Integration with mechanistic interpretability|Use feature decomposition to approximate Ω in latent space|

-----

## Conclusion

This work provides a concrete demonstration that constraining system dynamics at the level of reachability can eliminate unsafe states by construction, rather than attempting to suppress them after emergence.

The system is minimal. The invariant is general.

The invariant is falsifiable: if any trajectory reaches Ω under the constraint regime, the framework fails. This simulation provides a concrete environment in which that test can be performed.

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   Safety is not a property of outputs.                   ║
║   It is a property of reachable futures.                 ║
║                                                          ║
║   ℛ(t) ∩ Ω = ∅   ∀t                                     ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

-----

## How to Run

```bash
# Clone the repository
git clone https://github.com/davarn/morrison-framework.git

# Navigate to the simulation
cd morrison-framework/simulation

# Install and run
npm install
npm start
```

**Requirements:** Node.js 16+, React 18+.

**Live Demo:** [Morrison Reachability Simulation — Interactive](https://claude.ai/public/artifacts/d2ca07a4-78a2-41a7-938d-e852e119ee1b)

-----

Morrison Reachability Simulation™ · Morrison Framework™ · Geometric Control Theory of Cognition

GB2600765.8 · GB2602013.1 · GB2602072.7 · GB2602332.5

© 2026 Davarn Morrison — All Rights Reserved

-----

## Related Work

- [Morrison Reachability Guard — Universal Agent Plugin](https://github.com/davarn/morrison-framework) — Production middleware implementation
- [Morrison Safety Invariant — Structural AI Safety](https://github.com/davarn/morrison-framework) — The core safety condition
- [Success as Geometry — A Reachability Theory of Achievement](https://github.com/davarn/morrison-framework) — Framework applied to human performance
- [Why This Matters — Measuring Structural Alignment](https://github.com/davarn/morrison-framework) — Structural measurement model
- [Geometric Control Theory of Cognition](https://github.com/davarn/morrison-framework) — The base theory
- [Licensing, Citation, and IP](https://github.com/davarn/morrison-framework) — How to cite and licence
