# AIGurukul PS1 — Pure Neuro-Symbolic Architecture for Automated Floor Plan Generation

## Overview
This project implements a **Neuro-Symbolic AI Pipeline** to autonomously generate a 2D house perimeter (floor plan) that complies with strict physical constraints and the Seattle Building Code (SBC). 

Rather than relying purely on a Large Language Model (LLM)—which often struggles with deterministic spatial mathematics—or purely on a hardcoded procedural algorithm, this project combines the creative, generative reasoning of LLMs with the immutable mathematical proofs of a **Z3 Symbolic Solver**.

## System Architecture: The Neuro-Symbolic Loop
The pipeline consists of a continuous, autonomous feedback loop between three distinct entities coordinating in real-time:

1. **Agent 1 (The Architect - Cloud LLM):** Ingests raw text constraints (plot size, setbacks, tree protection zones). Using a required `[PLANNING]` Chain-of-Thought scratchpad, it calculates a valid geometry and outputs a JSON array of `[X, Y]` coordinates.
2. **The Verifier (Z3 Theorem Prover):** Ingests the JSON array and translates the physical rules into algebraic boolean logic. It tests the coordinates against 7 strict constraints. If a single point violates a rule, Z3 throws an `UNSAT` (Unsatisfiable) error and halts the build.
3. **Agent 2 (The Inspector - Local LLM):** Reads the raw, mathematical `UNSAT` error string from Z3 and translates it into a conceptual "Navigator Hint." It feeds this semantic hint back to Agent 1 for the next iteration, acting as an organic heuristic guide without giving away the mathematical answer.

## The Experimental Methodology & Constraints
The AI was tasked with maximizing land utilization on an 80x88 ft plot while adhering to the following rules:
* **Plot Size:** 80ft (X) by 88ft (Y).
* **South Setback:** All walls must be ≥ 8ft from the South street line.
* **Fire Safety Entry:** A single 40ft wide entry (X=35 to X=75) must extend to the street (Y=0).
* **Protected Tree:** No building allowed in the top-right corner (X > 75 AND Y > 84).
* **Optimization Target:** Maximize land utilization (Area > 6,600 sqft).
* **Orthogonality:** All lines must be strictly horizontal or vertical.

## Challenges & Failures

### 1. The Cognitive Ceiling of Small Models
Initially, the project tested the spatial reasoning limits of small, local LLMs:
* **Llama 3 (8B):** Failed due to severe cognitive overload. It could not hold 7 overlapping geometric planes in its working memory, frequently hallucinating coordinates and failing to close polygons.
* **Qwen 2.5 Coder (7B):** Showed significant improvement by correctly routing the complex entry protrusion. However, it suffered from a constraint imbalance: when forced to fix a setback error, it would shrink the house to a tiny 320 sqft box, failing the minimum area rule.

### 2. The Trap of Data Leakage & Hardcoding
During early development, when models struggled to find the valid geometric path, the prompt was augmented with "Planning Heuristics" (e.g., *Start at the West Wall, route East to X=80, drop down to Y=84*). 

**This approach was entirely rejected.** Providing routing steps constitutes **Data Leakage** and hardcoding. If the AI is told *how* to route the lines, it ceases to be an autonomous reasoning agent and degrades into a simple string formatter. The final implementation uses a **100% Pure Prompt** with *zero* hints, *zero* starting coordinates, and *zero* routing steps.

### 3. The "Flat Roof" Loophole
Once the prompt was purified, the AI discovered a mathematical loophole. It realized that if it capped the *entire* North wall of the house at `Y = 84`, it completely bypassed the tree zone (`Y > 84`) while still achieving 6,400 sqft of area (which passed an initial 5,000 sqft threshold). 
**Resolution:** To force the AI to discover the optimal "stepped" cutout shape, the Z3 Minimum Area rule was mathematically tightened to **6,600 sqft**.

## The Breakthrough: Hybrid Cloud/Local Architecture
To overcome the cognitive limitations of 7B models without sacrificing speed or incurring massive API costs, the system was upgraded to a **Heterogeneous Agent Architecture**:
* **Agent 1 (Architect):** Upgraded to **Llama-3.3-70B-Versatile** running via the Groq Cloud API, providing the massive spatial reasoning and logic capabilities required to process the pure constraint prompt.
* **Agent 2 (Inspector):** Kept as an 8B local model (`llama3:latest`) running via Ollama on an ASUS TUF Gaming A15, as it only needed basic NLP capabilities to translate Z3 errors.

## The Role of Stochasticity: Solving the State Space (11 vs. 6 Iterations)
During testing, the exact same codebase and constraints successfully solved the puzzle in **11 iterations** on one run, and then solved it in only **6 iterations** on the next run. This perfectly demonstrates the concept of **Stochasticity (non-determinism)** in AI:

Because LLMs navigate a vast geometric state space using probabilistic token generation, their initial heuristic path varies:
* **The 11-Iteration Run (Local Minimum Trap):** In the first run, the AI's initial heuristic seed was poor. It attempted to flatten the roof entirely or drastically shrink the house, falling into a "local minimum." It required extensive, back-and-forth guidance from the Z3 verifier and Agent 2 to forcefully push it out of these invalid geometries over 11 attempts.
* **The 6-Iteration Run (Efficient Organic Search):** In the subsequent run, the AI generated a much more balanced initial hypothesis. We observed a highly realistic, efficient search pattern where the AI intelligently bounced between correcting South Setback rules and dialing in Area requirements (6020 sqft → 6320 sqft → 5680 sqft) before successfully assembling the perfect geometry on attempt #6.

Ultimately, Agent 1 successfully calculated a mathematically perfect path that extended to `Y = 88`, stepped down exactly 4x5 feet to dodge the tree, routed the 40-foot entry protrusion, and closed the loop—utilizing a maximized **6,700 sqft** of space with exactly 11 vertices.

## Final Output
`[STEP 2] Constraint Verification (Z3 Symbolic Solver)`
`✓ Rule 1: Polygon closure (SAT)`
`✓ Rule 2/3: Plot bounds (SAT)`
`✓ Rule 4: Top-Right Tree (SAT)`
`✓ Rule 5: SBC South Setback & Entry (SAT)`
`✓ Rule 6: Orthogonality (SAT)`
`✓ Rule 7: Land Utilization (SAT: 6700.0 sqft)`
`[COMPLETE SUCCESS] Saved 'valid_house_layout.dxf' safely with 11 perfect vertices!`
