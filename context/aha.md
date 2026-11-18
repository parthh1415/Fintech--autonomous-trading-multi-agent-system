# Branch `aha`: audit engine

**Last updated:** 2026-11-17 (plan created, no code yet)

## Last worked on
- Nothing on this branch yet. The branch was created from main with the plan.

## Status of my slice (PLAN.md §5)
- [ ] Input normalisation, "set fund to X%" helper, focus selection, input validation
- [ ] Objective U(w), cost model, helps/neutral/hurts classification
- [ ] Base case vs doing nothing
- [ ] Four closed-form break-evens (return, cost, risk aversion, horizon) + plug-back tests
- [ ] Window test, return curve, cost test
- [ ] 60-scenario grid, smaller-adjustment rows, recommended step, verdict + action
- [ ] 2-D assumption map + test that map cells equal point evaluations
- [ ] Demo case reproduces the shape of the PLAN.md §4 numbers

## Next up
- Day 1 kickoff, then the objective + base case (day 2).

## Blocked on
- Nothing. Use Sunny's estimation module once it lands; until then, stub μ and Σ from a quick sample covariance.

## Needs from others
- @Sunny: the estimation interface (window → μ, Σ) by day 2.

## Log
- 2026-11-17: Repo planned and branches created. No code.
