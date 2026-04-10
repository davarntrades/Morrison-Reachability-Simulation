<div align="center">

# Morrison Reachability Simulation — v2 Development Specification

![Status](https://img.shields.io/badge/Status-Specification-555555?style=flat-square)
![Phase](https://img.shields.io/badge/Phase-Post--Grant_Engineering-0075ca?style=flat-square)
![Framework](https://img.shields.io/badge/Framework-Morrison_Framework™-0075ca?style=flat-square)
![Owner](https://img.shields.io/badge/Owner-Research_Engineer-2d6a2e?style=flat-square)
![©](https://img.shields.io/badge/©_Davarn_Morrison-555555?style=flat-square)

-----

*“Move from visual demonstrator to reproducible proto-system.”*

*— Davarn Morrison, 2026*

</div>

-----

This work builds upon the patented pre-semantic trajectory governance framework.

-----

## Purpose of This Document

This specification defines the v2 upgrade of the Morrison Reachability Simulation. The current v1 (artifact form) is a visual demonstrator suitable for grant calls and presentations. The v2 must be a testable, reproducible proto-system that supports trajectory-level safety validation, measurable deformation tracking, and deterministic governance constraints.

This document is the engineering brief for the research engineer once Phase 1 funding is secured. It is not intended for artifact-level implementation.

-----

## Design Constraints

The v2 system must remain:

- **Deterministic** — no randomness in core system; randomness only in adversarial mode toggle
- **Measurable** — every claim must reduce to a computable metric
- **Reproducible** — given identical inputs and seed, identical outputs
- **Observable** — all internal state queryable through the interface

These constraints are non-negotiable. They are what convert the simulation from a demonstrator into a validation instrument.

-----

## v1 → v2 Status

**Live v1 Demo:** [Morrison Reachability Simulation — Interactive](https://claude.ai/public/artifacts/7b346ba2-0381-4d89-bd20-78ee02939bef)

The current v1 simulation is the baseline this specification extends. Engineers should run and explore v1 before beginning v2 implementation.

|Component                     |v1 (Current Artifact)|v2 (Target)                            |
|:----------------------------:|:-------------------:|:-------------------------------------:|
|Reachable set computation     |✓ Implemented        |✓ Inherited                            |
|One-step constraint           |✓ Implemented        |✓ Inherited                            |
|Multi-step constraint         |✓ Implemented        |✓ Inherited (extend with continuous λ) |
|Causal accumulation C(t)      |✓ Added in v1.1      |Extend with formal threshold logic     |
|Distance-to-Ω metric          |✓ Added in v1.1      |Extend with continuous risk gradient   |
|Comparison dashboard          |✓ Added in v1.1      |Extend with full irreversibility column|
|Irreversibility (set-recovery)|Not implemented      |**v2 deliverable**                     |
|Trajectory memory             |Not implemented      |**v2 deliverable**                     |
|Continuous Λ                  |Binary only          |**v2 deliverable**                     |
|Identity signature            |Not implemented      |**v2 deliverable**                     |
|Adversarial mode              |Not implemented      |**v2 deliverable**                     |
|Safe manifold visualisation   |Not implemented      |**v2 deliverable**                     |

-----

## v2 Deliverable 1 — Irreversibility (Refined Definition)

### Goal

Detect when the system enters a state where previous reachable structures are no longer reachable under the same transition constraints.

### Definition

Irreversibility is **not** defined as set size changing. It is defined as **loss of reachable structure**.

Formally: the system at time t is irreversible relative to time t₀ if there exists no future time t+k such that ℛ(t₀) ⊆ Reach⁺(x_{t+k}, k’) for any k’ ≥ 0 under the active constraint regime.

### Implementation

Store the last N reachable sets:

```javascript
const reachableHistory = []; // sliding window of length N
```

At each step, check whether ℛ(t₀) is recoverable from any future reachable set under the current dynamics. If not, mark as irreversible.

```javascript
function isIrreversible(historyIndex, currentReachable, transitionFn) {
  const past = reachableHistory[historyIndex];
  // Check if past set is contained in any forward closure of current
  return !canRecover(past, currentReachable, transitionFn);
}
```

### Output

- `irreversibleStep` — the timestep at which irreversibility was first detected
- Visual indicator: banner reading **“⚠ Irreversibility detected at t = X”**
- Comparison dashboard column: irreversibility status per mode

### Critical Constraint

Irreversibility detection must reflect actual structural loss, not numerical changes. The implementation must distinguish between transient set fluctuations and permanent topological loss.

-----

## v2 Deliverable 2 — Continuous Governance Parameter Λ

### Goal

Convert Λ from a binary on/off constraint into a continuous deterministic parameter that controls the strength of governance.

### Implementation

```javascript
const [lambda, setLambda] = useState(1.0); // range: 0.0 → 1.0

function admissibleAction(x, u, lambda, lookahead) {
  const risk = computeRisk(x, u, lookahead); // 0 if safe, 1 if leads to Ω
  return risk <= (1 - lambda); // λ=1 blocks all risky, λ=0 blocks none
}
```

### Behaviour

|λ value     |Behaviour                                           |
|:----------:|:--------------------------------------------------:|
|λ = 0.0     |No constraint — system behaves as unconstrained mode|
|λ = 0.5     |Partial admissibility — moderate risk tolerance     |
|λ = 1.0     |Full constraint — current multi-step behaviour      |
|Intermediate|Continuous interpolation between regimes            |

### Critical Constraint

The risk function must be deterministic. No probabilistic sampling in the core system. Continuous Λ controls deterministic admissibility, not stochastic acceptance.

### UI

- Slider control: λ ∈ [0, 1] with 0.05 increments
- Live update of reachable set as λ changes
- Display: current λ value and resulting admissibility threshold

-----

## v2 Deliverable 3 — Trajectory Memory (Causal Layer)

### Goal

Track how states are reached, not just whether they are reachable. Maintain the causal history of trajectories through state space.

### Data Structure

```javascript
// For each reachable state, store all paths that reached it
const trajectoryMap = new Map(); // state -> array of paths
// Each path is a sequence of states from x₀

// Each path is tagged
const pathTag = {
  type: "SAFE" | "UNSAFE",
  intersectsOmega: boolean,
  length: number
};
```

### Tagging

For each path:

- **SAFE** — path never intersects Ω
- **UNSAFE** — path leads to Ω at some point

### Output

For each state, display:

- Number of safe paths reaching it
- Number of unsafe paths reaching it
- Example: *“State (8,12) reachable via 12 safe paths and 3 unsafe trajectories”*

### Optional Extension

Compute approximate conditional probability:

```
P(reach Ω | currently at x) = unsafe_paths(x) / total_paths(x)
```

This converts the binary safe/unsafe distinction into a continuous risk score.

-----

## v2 Deliverable 4 — Distance-to-Ω Continuous Risk Field

### Goal

Already implemented in v1.1 as a heatmap overlay. v2 extends this with:

- Per-state numeric distance display (tooltip)
- Continuous gradient instead of discrete colour bands
- Integration with risk-based admissibility (couples to Deliverable 2)

### Definition

```
d(x, Ω) = min { k : Reach⁺(x, k) ∩ Ω ≠ ∅ }
```

The minimum number of steps required to reach Ω from state x under the active dynamics.

### Implementation

Maintain a precomputed BFS distance map updated whenever Ω changes (relevant for adversarial mode in Deliverable 6).

-----

## v2 Deliverable 5 — Identity Signature

### Goal

Track structural identity changes over time without overcomplication.

### Definition (v2 minimal)

```javascript
function identitySignature(reachableSet) {
  return {
    size: reachableSet.size,
    boundary: computeBoundary(reachableSet),
    connectivity: computeComponents(reachableSet)
  };
}
```

### Output

- Track signature over time
- Detect significant shifts (threshold-based)
- Visual indicator when identity transitions occur

### Theoretical Connection

This is the operational implementation of:

```
ℐ(x₀) := [Reach⁺(x₀, t)]_∼
```

where ∼ is the equivalence relation preserving size, connectivity, and boundary structure. v2 provides a computable approximation of this invariant.

-----

## v2 Deliverable 6 — Adversarial Mode

### Goal

Test system robustness under perturbation. This is the **only** module permitted to use randomness, and only when explicitly toggled by the user.

### Modes

|Perturbation Type    |Description                                |
|:-------------------:|:-----------------------------------------:|
|Ω drift              |Ω location shifts slightly over time       |
|State perturbation   |Random state injections at low rate        |
|Dynamics modification|Transition function modified mid-simulation|

### Metrics

Track and report:

- **Failure rate** — number of timesteps where Ω was reached
- **Containment rate** — fraction of trajectories successfully prevented
- **Recovery time** — steps required to restore safety after perturbation

### Critical Constraint

Adversarial mode must be a clearly labelled separate mode. Core deterministic behaviour must remain available at all times. Adversarial randomness must be seeded and reproducible.

-----

## v2 Deliverable 7 — Safe Manifold Visualisation

### Goal

Explicitly visualise the safe region of the state space — the set of states that are reachable AND will remain reachable under the constraint regime.

### Definition

```
SafeManifold(t) = ℛ(t) ∩ complement(Ω_expanded(N))
```

Where Ω_expanded(N) is the backward-reachable unsafe set within N steps.

### Output

Three distinct visual layers:

- **Reachable** (active mode colour)
- **Safe manifold** (highlighted with bright green border)
- **Unsafe approach zone** (states adjacent to but not in Ω_expanded)

### Significance

The safe manifold is the formal answer to the question: “Where can the system safely operate, and where can it remain safely?” It is the geometric definition of the operational envelope.

-----

## v2 Deliverable 8 — Comparison Dashboard (Extended)

### Goal

Provide clear, side-by-side validation metrics across all modes.

### Required Columns

|Metric            |Free      |One-step  |Multi-step|Adversarial|
|:----------------:|:--------:|:--------:|:--------:|:---------:|
||ℛ(t)      |          |Live      |Live       |
|Ω reached?        |Yes/No    |Yes/No    |Yes/No    |Yes/No     |
|C(t) load         |—         |Live      |Live      |Live       |
|Irreversible?     |Yes/No/t=X|Yes/No/t=X|Yes/No/t=X|Yes/No/t=X |
|Identity signature|Hash      |Hash      |Hash      |Hash       |
|Failure rate      |—         |—         |—         |Live       |

### Requirement

All values must be:

- Computed live
- Reproducible across sessions (with seed for adversarial)
- Exportable to CSV for analysis

-----

## Final Constraints (Non-Negotiable)

1. **No randomness in core system.** Only adversarial mode toggle permits randomness, and it must be seeded.
1. **All metrics must be:**
- Computable in real time
- Observable through the UI
- Reproducible given identical inputs
1. **No abstract labels** unless tied to measurable quantities. Every UI element must trace to a formal definition in this specification.
1. **Modular implementation.** Each deliverable should be independently testable. Failure of one module must not break the others.
1. **Backwards compatibility.** v2 must preserve all v1 functionality. Users should be able to switch between v1 and v2 modes.

-----

## Outcome

After v2 implementation, the system must support:

```
════════════════════════════════════════════════════════════
  1. STRUCTURAL SAFETY VALIDATION
     Not just filtering — actual reachability analysis

  2. TRAJECTORY-LEVEL REASONING
     Path-aware safety, not just state-aware

  3. MEASURABLE DEFORMATION OVER TIME
     C(t), identity signatures, set recovery

  4. DETERMINISTIC GOVERNANCE CONTROL
     Continuous Λ with reproducible behaviour

  5. CLEAR COMPARISON AGAINST BASELINE
     Side-by-side evaluation across all regimes
════════════════════════════════════════════════════════════
```

-----

## Development Phases

|Phase  |Deliverables                                         |Estimated Duration|
|:-----:|:---------------------------------------------------:|:----------------:|
|Phase A|Continuous Λ + extended dashboard + trajectory memory|6 weeks           |
|Phase B|Irreversibility + identity signature + safe manifold |6 weeks           |
|Phase C|Adversarial mode + benchmarking + CSV export         |4 weeks           |
|Phase D|Integration testing + documentation + research paper |4 weeks           |

Total: ~5 months for full v2 implementation by a part-time research engineer.

-----

## Why This Document Exists

This specification is intentionally separated from the v1 simulation. The v1 simulation is a demonstrator suitable for the Adrian Hindes evaluation call (April 13, 2026), grant applications (SFF, EA Funds), and public presentations. It must remain stable and demo-safe.

The v2 system is a research instrument. It requires a proper development environment, version control, automated testing, and dedicated engineering time. Building v2 features into the v1 artifact would compromise both: the demo would become fragile, and the research instrument would lack the rigour required for validation.

This document is the brief for the research engineer position funded under the Phase 1 grant. It defines exactly what they will build, in what order, and against what acceptance criteria.

-----

Morrison Reachability Simulation v2 · Morrison Framework™ · Geometric Control Theory of Cognition

GB2600765.8 · GB2602013.1 · GB2602072.7 · GB2602332.5

© 2026 Davarn Morrison — All Rights Reserved
