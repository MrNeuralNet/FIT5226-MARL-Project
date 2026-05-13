# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

FIT5226 university project implementing Q-learning agents in a grid world. Stage 1 (complete) is a single-agent solution. Stage 2 (in progress) extends this to multi-agent coordination. The assignment spec PDFs are in the repo root.

## Running the Code

This is a Jupyter notebook project. Launch with:
```
jupyter notebook
```
Or run a notebook non-interactively:
```
jupyter nbconvert --to notebook --execute "Stage 1 Solution.ipynb"
```

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

### Stage 2 (to be built)

Extends Stage 1 with:
- **Two agent types** (A and B) with separate Q-tables
- **Fixed locations**: X (west), Y (north), U (east), V (south), lake at the intersection of shortest paths
- **5 actions**: NSEW + wait
- **Lake state**: binary dry/flooded, flips probabilistically after each step
- **State per agent**: `(row, col, carrying_sample, lake_state)`
- **Simultaneous stepping**: all agents act at the same time; collisions detected post-move
- **Rewards**: step=-5, wait=-3, collision=-20, Type A entering flooded lake=-20, sample pickup=+10, delivery=+50

Key implementation note from the spec: when updating Q-values, account for **both possible next lake states** (it could flood or dry at any point).
