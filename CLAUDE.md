# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow

Commit and push to GitHub regularly as work progresses — after each meaningful change, completed task, or working milestone. Never let significant work sit uncommitted. Use clear, descriptive commit messages that capture what changed and why.

```
git add <files>
git commit -m "descriptive message"
git push
```

## Project Context

FIT5226 university project implementing Q-learning agents in a grid world. Stage 1 (complete) is a single-agent solution. Stage 2 (complete) is a multi-agent coordination solution covering all four assignment tasks (Pass through HD level). Assignment spec PDFs are in the repo root. Deadline: May 15, 2026.

## Running the Code

This is a Jupyter notebook project. Launch with:
```
jupyter notebook
```
Or run a notebook non-interactively:
```
jupyter nbconvert --to notebook --execute "Stage 1 Solution.ipynb"
jupyter nbconvert --to notebook --execute "Stage 2 Solution.ipynb" --ExecutePreprocessor.timeout=600
```
Stage 2 takes ~5 minutes to execute (100k + 100k + 5×50k + 4×50k training episodes).

## Architecture

### Stage 1 (`Stage 1 Solution.ipynb`)

Two classes work together:

**`GridWorldEnvironment`** — owns all world state:
- 5×5 grid, agent starts at `(0,0)`, nest fixed at `(4,4)`, food source randomised per episode
- State tuple: `(agent_row, agent_col, food_row, food_col, reached_A)` — food location is included so the agent can navigate toward it
- `reached_A` flag gates the reward logic: +10 on reaching food, +50 on returning to nest, -1 per step otherwise

**`QTableAgent`** — owns the Q-table:
- Shape `(5, 5, 5, 5, 2, 4)` — one dimension per state component, last dimension is the 4 actions
- Standard ε-greedy selection and Q-learning (off-policy TD) update
- Trained Q-table saved to `qTable single agent fast.dat`

### Stage 2 (`Stage 2 Solution.ipynb`)

#### Grid layout
```
     col0  col1  col2  col3  col4
row0  [  ]  [  ]  [ Y]  [  ]  [  ]
row1  [  ]  [  ]  [  ]  [  ]  [  ]
row2  [ X]  [  ]  [LK]  [  ]  [ U]
row3  [  ]  [  ]  [  ]  [  ]  [  ]
row4  [  ]  [  ]  [ V]  [  ]  [  ]
```
X=(2,0) A start/return, Y=(0,2) B start/return, U=(2,4) A sample, V=(4,2) B sample, LAKE=(2,2).

#### Key classes

**`MultiAgentEnvironment`** — owns all world state:
- `lake_state` (0=dry, 1=flooded), flips with `LAKE_FLIP_PROB=0.3` each timestep (step 6 of spec sequence)
- `step(action_a, action_b, phase)` — simultaneous execution; collision detected when both land on LAKE; Phase 1 only: A penalised -20 for entering/staying in flooded lake
- `get_state_a/b()` returns `(row, col, carrying, lake_state)`

**`QTableAgent2`** — one instance per agent type:
- Q-table shape `(5, 5, 2, 2, 5)` = row × col × carrying × lake_state × action
- `update()` uses expected-lake Q-update: averages future Q over both possible next lake states weighted by `LAKE_FLIP_PROB` (spec Hint 4)
- Critical: snapshot `was_done = env.done` **before** `env.step()` to correctly update the terminal delivery step

#### Training
- `train(agent_a, agent_b, env, ..., phase=1|2)` — joint loop, 100k episodes, lr=0.3, γ=0.95, ε: 1.0→0.05
- Phase 1: water damage penalty active for A; Phase 2: no water penalty
- Saved artefacts: `qTable_A_phase1.dat`, `qTable_B_phase1.dat`, `qTable_A_phase2.dat`, `qTable_B_phase2.dat`

#### Helper functions
- `visualise_policy(agent, 'A'|'B')` — prints greedy action at approach cell and lake cell for all (carrying, lake_state) combinations
- `trace_episode(agent_a, agent_b, env, phase)` — runs one greedy episode and prints step-by-step path
- `plot_results(...)` — 2×2 subplot: reward curves, collision rate, success rate

#### Task structure
| Task | Content |
|---|---|
| Task 1 | Phase 1 training + plots + policy visualisation |
| Task 2 | Phase 2 training + comparison + seed-sensitivity experiment + step-penalty sweep |
| Task 3 | Hawk-Dove payoff matrices (V=10, C=40), `replicator_dynamics()`, phase portraits |
| Task 4 | Theoretical explanation markdown (148 words, limit 150) |

#### Known behaviour
Phase 1 agents learn **path specialisation** (A detours via row 3, B crosses through lake) rather than the canonical traffic-light solution. This is valid — the water penalty drives the asymmetry as required by the spec — but is suboptimal for A. Phase 2 collision rate (~0.089) is ~2× Phase 1 (~0.047), demonstrating the harder coordination problem.
