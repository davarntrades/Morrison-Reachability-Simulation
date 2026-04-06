<div align="center">

# Morrison Reachability Simulation™

## Interactive Demonstration of Structural Safety via Reachability Constraints

![Type](https://img.shields.io/badge/Type-Interactive_Simulation-0075ca?style=flat-square)
![Framework](https://img.shields.io/badge/Framework-Morrison_Framework™-0075ca?style=flat-square)
![Safety](https://img.shields.io/badge/Safety-Reachability_Invariant-2d6a2e?style=flat-square)
![Language](https://img.shields.io/badge/Language-React_·_Vite-0075ca?style=flat-square)
![©](https://img.shields.io/badge/©_Davarn_Morrison-555555?style=flat-square)

-----

*“Safety is not what a system says — it is what it can reach.”*

*— Davarn Morrison, 2026*



-----

## Overview

This interactive simulation demonstrates the Morrison Safety Invariant in a simple 20×20 toy state space.

Instead of shaping outputs after they are generated, safety is enforced **structurally** by constraining the reachable set ℛ(t) so that unsafe states in Ω become unreachable.

**Core Invariant**  
ℛ(t) ∩ Ω = ∅ ∀ t ≥ 0

When this holds, failure is not merely unlikely — it is geometrically impossible.

---

## System Parameters

| Parameter           | Value                          |
|---------------------|--------------------------------|
| State space         | 20 × 20 grid                   |
| Initial state       | x₀ = (3, 14)                   |
| Transitions         | 8-directional                  |
| Forbidden region    | Ω = 5×5 block (12–16, 5–9)     |
| Time horizon        | t = 0 to 12                    |

---

## Two Modes of Operation

### ❌ No Constraints  
- All transitions permitted  
- Reachable set expands freely  
- ℛ(t) inevitably intersects Ω  

**Outcome:** Unsafe states are reached.

### ✅ One-Step Constraint (A_safe(x))  
- Transitions into Ω are blocked at the action level  
- Reachable set deforms around the forbidden region  
- ℛ(t) ∩ Ω = ∅ is maintained  

**Outcome:** Unsafe states remain structurally unreachable.

---

## Key Insight

The simulation shows that **safety is not output filtering** — it is a property of the system’s dynamics.

By enforcing the constraint at the transition level, the reachable set is reshaped before unsafe states can ever be entered.

> Governance does not suppress behaviour.  
> It makes dangerous futures unreachable by construction.

---

## How to Run Locally

```bash
npm install
npm run dev

Deployment (Vercel — Recommended)
	1	Push this repository to GitHub
	2	Go to vercel.com
	3	Import the project
	4	Click Deploy

File Structure
morrison-reachability-simulation/
├── package.json
├── vite.config.js
├── index.html
└── src/
    ├── main.jsx
    └── App.jsx          ← Core simulation logic

Author
Davarn Morrison Morrison Framework™ · Resurrection Tech Ltd
© 2026 Davarn Morrison — All Rights Reserved

ℛ(t) ∩ Ω = ∅
If unsafe states are unreachable, failure is structurally impossible.
```
