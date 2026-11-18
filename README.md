# HEDGEWISE

**HEDGEWISE helps investors challenge a proposed portfolio change. It shows the assumptions that support it, finds where it stops making sense, and compares smaller adjustments with doing nothing.**

The core question: *"Does this proposed change still make sense if the assumptions behind it are slightly wrong?"*

This repo holds the plan only so far. Nobody has written code yet. Start here:

| File | What it's for |
|---|---|
| [docs/CONCEPT.md](docs/CONCEPT.md) | The product: what it does, what makes it different, what it won't claim |
| [docs/PLAN.md](docs/PLAN.md) | Architecture, the math we agreed on, who owns what, the 2-week timeline, the demo, risks |
| [docs/API_CONTRACT.md](docs/API_CONTRACT.md) | The shared contract between backend and frontend. Agree on it before building |
| [CLAUDE.md](CLAUDE.md) | Rules for every coding agent (and human) working in this repo, including the context protocol |
| [context/](context/) | **What each branch last worked on.** One file per person, kept up to date on `main` |

## Branches

| Branch | Owner | Area |
|---|---|---|
| `main` | everyone | Integration branch. Holds the plan and the `context/` files |
| `parth` | Parth | Data layer, API, decision report, explanations, integration |
| `aha` | Aha | Audit engine: the objective, the five tests, break-even points, verdict |
| `sunny` | Sunny | Historical validation (walk-forward), estimation windows, sourcing return assumptions, evaluation write-up |
| `jp` | JP | Frontend: setup flow, live "breaking point" sliders, assumption map, report view |

Each person works on their own branch. Before you stop, update `context/<your-branch>.md` **on main**; [CLAUDE.md](CLAUDE.md) explains how. Anyone (or any agent) can then read `main` and see where all four branches stand.

_HEDGEWISE is a decision-support project. Its outputs are analytical estimates under disclosed assumptions, not forecasts or personal financial advice._
