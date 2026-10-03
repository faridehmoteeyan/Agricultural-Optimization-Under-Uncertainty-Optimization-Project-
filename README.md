
# Agricultural Optimization Under Uncertainty
<img width="2199" height="1650" alt="image" src="https://github.com/user-attachments/assets/ac536a0d-7baa-4359-8682-7ad2ac48aac9" />

## Implementation and Analysis of the L-Shape Multicut Algorithm

**Final Project — Stochastic Programming Course**  
Sharif University of Technology — Spring 2026  
Instructor: Dr. Majid Rafiee

---

## Project Overview

This project models the **Farmer Problem** as a two-stage stochastic programming problem and solves it using the **L-Shape Multicut** algorithm. The goal is to evaluate the efficiency of this method for large-scale problems (64 scenarios) and demonstrate its fast convergence behavior.
> **Note:** The source code and the full project report (PDF) are available in Persian in this repository.


---

## Objectives

- Model the Farmer Problem as a two-stage stochastic program
- Implement the L-Shape Multicut algorithm (multi-cut variant)
- Generate random scenarios based on specified distributions
- Evaluate the algorithm on large-scale problems
- Analyze convergence behavior across scenarios

---

## Farmer Problem Description

A farmer must decide at the beginning of the year how much land to allocate to three crops:

| Crop | Planting Cost (per hectare) |
|------|----------------------------|
| Wheat | 150 |
| Corn | 230 |
| Sugar Beet | 260 |

**Constraints and rules:**
- Total arable land: **500 hectares**
- The farmer must keep a minimum amount of wheat and corn for personal use
- Surplus crops can be sold; shortages can be purchased
- Sugar beet has a favorable price up to a quota (**6000 kg**); beyond that, it is sold at a lower price

**Sources of uncertainty:**
- Crop harvest coefficients
- Selling and purchase prices
- Minimum required wheat and corn

---

Here `y` represents purchase variables and `w` represents selling variables.

---

## L-Shape Multicut Algorithm

The multi-cut variant adds **one cut per scenario** in each iteration instead of a single cut. Thus, up to **k cuts** are added per iteration.

**Algorithm steps:**

| Step | Description |
|------|-------------|
| 0 | Initialize: `v = 0`, `r = 0`, `Sₖ = 0` |
| 1 | Solve master problem to obtain `(xᵛ, θᵛ)` |
| 2 | Check feasibility of `xᵛ` for all scenarios; if infeasible, add feasibility cut |
| 3 | For each scenario: solve subproblem and extract dual multipliers (π) |
| 4 | If `θₖᵛ < eₛₖ₊₁ − Eₛₖ₊₁·xᵛ` ⇒ add optimality cut |
| 5 | If no cuts added ⇒ **terminate**; otherwise return to Step 1 |

**Termination criterion:** No new cuts are added for any scenario (or differences are below `1e-5`).

---

## Scenario Generation

Scenarios are generated from a normal distribution using the **generalized trapezoidal rule**:

- `δ, θ, α, β, γ, ε` are sampled from N(0, 0.33)
- **8 bins** per random variable ⇒ **64 scenarios** total
- Probability of each scenario = (points in bin) / (total samples = 2000)

> ⚠️ Minimum requirements for wheat and corn are kept deterministic at 200 and 240.

---

## Code Structure

**Main code sections:**

1. **Libraries**
   - `pyomo` — optimization modeling and solving
   - `numpy` — algebraic operations and random sampling
   - `matplotlib` — plotting

2. **Parameter definitions**
   - Deterministic: total area, beet quota, planting costs
   - Stochastic: yields, prices, minimum needs

3. **Scenario generation** (`test_data_50`)
   - Normal sampling
   - Binning and probability computation

4. **Computing `t` and `h` matrices**
   - Building subproblem coefficients for each scenario
   - Storing in dictionaries for reuse

5. **Main algorithm loop**
   - Solve master problem
   - Feasibility check
   - Solve subproblem and extract duals
   - Generate optimality cuts
   - Termination check

6. **Plotting**
   - θ changes across iterations
   - w changes across iterations
   - θ vs. w comparison for a specific scenario

---

## Results

**Final output after 7 iterations:**

**Runtime:** Average **111 seconds** over three runs (97, 94, 140 s)

**Observations:**
- θ converges quickly at first, then slowly (classic Benders weakness)
<img width="624" height="362" alt="image" src="https://github.com/user-attachments/assets/52c91128-4a5d-452c-9b08-34b971c6514e" />

- w follows a similar trend and acts as a feasible bound
<img width="1347" height="762" alt="image" src="https://github.com/user-attachments/assets/eb82cb85-0c29-4b00-8707-6cf99337f209" />

- θ and w converge toward each other as upper/lower bounds per scenario
<img width="932" height="586" alt="image" src="https://github.com/user-attachments/assets/b217016b-8fa5-4e20-bdc5-2ea5627990f5" />

---

## Requirements

```bash
pip install pyomo numpy matplotlib
solver = SolverFactory('cplex')  # or 'glpk', 'gurobi'
