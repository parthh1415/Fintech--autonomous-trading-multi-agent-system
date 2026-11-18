# HEDGEWISE: Concept (revised)

## One line

HEDGEWISE audits an investment recommendation **before** someone acts on it.

## The question it answers

> "Does this proposed change still make sense if the assumptions behind it are slightly wrong?"

We started with a broad "portfolio defense dashboard" idea. We narrowed it on purpose: the product is the *reliability of one specific decision*, not another risk score.

## How a user experiences it

1. They enter their **current portfolio** (manual entry, from a small universe of diversified funds).
2. They enter a **proposed change**, e.g. *"An investing tool suggested increasing my technology allocation from 20% to 40%."*
3. HEDGEWISE tests the change against **doing nothing** and tells them:
   - **What needs to be true** for the change to help (e.g. "tech must return at least 13.8% a year").
   - **Which assumptions reverse it** (estimation window, expected return, trading cost, risk tolerance, horizon).
   - **Whether the benefit survives trading costs.**
   - **Whether a smaller adjustment gets most of the benefit** with less concentration and turnover.
   - **Whether the evidence is too uncertain** to justify changing anything.
4. They get a **decision report** they can inspect and challenge. It is reproducible: the same inputs and data always give the same report ID and the same numbers.

## The feature we build around: find where the recommendation flips

- Sliders for the focus asset's expected return, trading costs, risk aversion, horizon and the historical period.
- A live sentence: *"Below 13.8%/yr for XLK, increasing your allocation no longer improves the objective."*
- A 2-D **assumption map** that shows where the change helps, hurts, or makes little difference, labelled with the exact model and assumptions behind it.

This is the live demo: a judge challenges the recommendation and watches the verdict and the explanation update.

## The audit workflow (what's actually different)

| Part of the audit | What the user learns |
|---|---|
| Compare the proposed action with doing nothing | Whether changing anything adds enough modelled value |
| Find where the recommendation reverses | Which assumptions the action depends on |
| Test smaller adjustments | Whether a less disruptive choice gets similar results |
| Produce a reproducible report | Exactly what data, objective, costs and assumptions produced the conclusion |

## Illustrative example (shape of the output, not final numbers)

| Test | What HEDGEWISE finds |
|---|---|
| Use the most recent historical period | Increasing tech looks beneficial |
| Include an earlier difficult period | The estimated benefit disappears |
| Reduce the assumed tech return slightly | The proposed increase becomes unattractive |
| Include trading costs | The advantage shrinks further |
| Test a smaller increase | Captures much of the benefit with less concentration |

> "The proposed increase depends heavily on strong technology returns continuing. Its advantage disappears under several tested assumptions. A smaller increase has less sensitivity, while keeping your current allocation avoids trading costs."

## What we will and won't claim

- **We audit the proposal with our own disclosed model.** We can't reconstruct another platform's proprietary reasoning, and we won't say we do.
- **We are not inventing a new financial method.** Sensitivity analysis and robust portfolio optimisation already exist. Our claim is a better way for ordinary users to *inspect* a decision, backed by our implementation and user testing.
- **Competitors:** PortfolioPilot already lets users switch between model assumptions (its own forecasts, market expectations, a blend). Changing assumptions alone isn't enough. Our edge is the complete audit workflow above: do-nothing baseline, reversal points, smaller steps, reproducible report.
- **No ML for its own sake.** Use machine learning only if it improves a measured outcome. An LLM may *rephrase* verified results; it never produces numbers.
- **Stability isn't success.** A strategy that never trades is perfectly stable. Validation always reports return, risk, drawdown, turnover and costs together.
- Outputs are analytical estimates, not forecasts, and not personal financial advice.

## Pitch

> HEDGEWISE helps investors challenge a proposed portfolio change. It reveals the assumptions that support it, finds where it stops making sense, and compares smaller adjustments with doing nothing.
