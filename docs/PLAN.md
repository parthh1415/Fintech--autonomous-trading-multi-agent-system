# HEDGEWISE: 2-week build plan

**Team:** Parth, Aha, Sunny, JP · **Window:** Wed 18 Nov to Tue 1 Dec 2026 · **Read first:** [CONCEPT.md](CONCEPT.md)

> Heads-up: **Thursday 26 Nov is Thanksgiving.** The plan treats 25 to 27 Nov as light days. Feature freeze lands *before* it.

---

## 1. Scope

### In the MVP
- Manual entry of a current portfolio from a fixed universe of 11 diversified ETFs.
- A proposed change, entered either as "set one fund to X%" (the rest scale proportionally) or as a full proposed allocation.
- One objective (below), applied the same way everywhere.
- The audit:
  1. base case vs doing nothing
  2. estimation-window test
  3. expected-return break-even
  4. trading-cost test
  5. smaller-adjustment test

  Plus a verdict and a recommended action.
- **Breaking-point sliders** and the **assumption map** (the demo centrepiece).
- Walk-forward **historical validation** comparing *follow the recommendation*, *audited version* and *do nothing*.
- **Reproducible decision report** (on screen + Markdown/JSON download).
- Plain-language explanations generated from templates filled with computed numbers.

### Out of scope (say so if asked)
Brokerage import, individual stocks, taxes, live data, user accounts, ML models, LLM-generated numbers. "Explain with an LLM" is a stretch goal only, and only as a rephrasing layer (see §7).

---

## 2. Architecture

| Layer | Tech | Owner |
|---|---|---|
| Price data | Daily adjusted closes from Yahoo via the `yfinance` library (**use v0.2.60 or newer**; 0.2.51 returns empty data now). Snapshot saved to a committed CSV, so every run is reproducible offline | Parth |
| Fund universe + long-run return assumptions | A committed JSON file (ticker, name, group, expected return) | Sunny sources the numbers, Parth owns the file format |
| Audit engine | Python: numpy, pandas, scipy | Aha |
| Historical validation | Python, same engine | Sunny |
| Explanations + report | Python, template-based | Parth |
| API | FastAPI | Parth |
| Frontend | Next.js (App Router, TypeScript), Tailwind, Recharts | JP |

Repo layout (folders we'll create; none exist yet):
- `backend/`: engine package, API, data folder, tests
- `frontend/`: Next.js app (one component per page section so nobody collides)
- `docs/`: this plan, the concept, the API contract, sample API responses
- `context/`: per-branch status (see [CLAUDE.md](../CLAUDE.md))

---

## 3. Decisions we already made (don't relitigate without the group)

### Universe (11 funds, all trading since July 2007, so the 2008 crisis is in the data)
SPY (US large cap), XLK (US tech), IWM (US small cap), XLV (US health care), VEA (international developed), VWO (emerging markets), VNQ (REITs), GLD (gold), AGG (US aggregate bonds), TLT (long Treasuries), SHY (short Treasuries, cash-like).
VEA is the youngest fund (starts 26 Jul 2007), so it sets the common start date.

### The objective (one, everywhere)
- Risk-adjusted expected return (mean-variance certainty equivalent): **U(w) = w·μ − (γ/2)·wᵀΣw**, annual.
- **Net benefit** of a change = U(proposed) − U(current) − (cost in bps ÷ 10,000) × turnover ÷ horizon in years.
- **Turnover** = sum of |proposed weight − current weight| (buys + sells).
- **Neutral band** (default 0.10% a year): differences smaller than this count as "makes little difference". Every result is classified **helps / neutral / hurts**.
- **Σ:** sample covariance of daily returns in the chosen window, × 252.
- **μ:** one of *historical* (window mean × 252), *assumptions* (long-run numbers in the universe file), or *blend* (default 50/50).
- **Defaults:** blend 50/50, window = last 5 years, γ = 3, cost = 10 bps, horizon = 3 years, return shift = 0.

Why this objective: it's standard, explainable in one sentence, and every break-even below has a **closed form**, so any number on screen can be checked by hand.

### The "focus" asset
The fund whose expected return we stress-test. By default it's the fund with the biggest weight increase; the user can override it.

### Break-even points ("what needs to be true")
All four are closed form. Each holds everything else at the base assumptions:
- **Return:** net benefit is linear in the focus asset's return, with slope = its weight change. Required return = assumed return − net ÷ (weight change). If the focus weight goes *up*, it must beat this bar; if it goes *down*, it must stay below it.
- **Cost:** costs break even at bps = gross benefit × horizon ÷ turnover × 10,000 (only when the gross benefit is positive).
- **Risk aversion:** γ\* = 2 × (Δ expected return − cost drag) ÷ Δ variance. If the change adds variance, it only helps investors below γ\*; if it removes variance, only investors above it.
- **Horizon:** minimum years to earn back the one-off cost = one-off cost ÷ gross benefit.

### Estimation windows (6)
Last 3 years · last 5 years · last 10 years · full history (since 2007, includes 2008) · everything except the last 3 years · 2007–2015 (crisis and recovery). Drop any window with fewer than ~120 trading days.

### Robustness scenario grid (disclosed in every report)
6 windows × focus-return shifts of −2, −1, 0, +1, +2 percentage points × cost ∈ {base, max(2 × base, base + 10 bps)} = **60 scenarios**.

### Smaller adjustments
Test fractions α = 0, 5%, 10% … 100% of the proposed change. For each α report:
- base net benefit
- share of the full change's benefit
- hit rate (share of scenarios where it helps)
- 25th-percentile net benefit
- turnover
- largest weight

**Recommended step** = the α with the best 25th-percentile net benefit, with ties going to the smaller move. Doing nothing scores exactly 0, so a change has to win in at least three quarters of the tested scenarios to be recommended.

### Verdict rules
| Code | Rule |
|---|---|
| HURTS | base net benefit < −band |
| NO_MEANINGFUL_BENEFIT | base net benefit within ±band |
| ROBUST | helps in ≥ 75% of scenarios |
| FRAGILE | helps in 40–75% |
| TOO_UNCERTAIN | base case helps but < 40% of scenarios do |

The **action** is separate: keep current (α = 0), smaller change (0 < α < 1), or proposed change (α = 1).

### Walk-forward validation
- Start from the user's portfolio and rebalance every 12 months, using **only data before each rebalance date**.
- The "recommender" is a disclosed stand-in for an investing tool: a long-only mean-variance optimiser on trailing 3-year returns, max 60% per fund.
- Three strategies:
  - *do nothing* (buy and hold)
  - *follow* (trade to the recommendation every time)
  - *audited* (move only as far as the step rule allows)
- Report CAGR, volatility, max drawdown, return per unit of risk, turnover, costs, trade count and allocation instability, all side by side.

### Reproducibility
Every report carries:
- the normalised inputs and all assumptions
- the scenario grid definition and the model formulas
- engine version
- data date range and **SHA-256 of the price CSV**
- a **report ID** = hash of (inputs + data hash + engine version)

Same inputs + same data = same ID.

---

## 4. What we learned from a throwaway prototype (re-verify, don't trust blindly)

We built a quick prototype on 17 Nov to sanity-check this plan, then deleted it so the team builds from scratch. Use these numbers only as a sanity check for your own implementation:

- **The demo case is a good story.** Current $20k = SPY 40%, XLK 20%, VEA 10%, AGG 25%, GLD 5%. Proposed: XLK → 40%. At default assumptions:
  - net ≈ **−0.30%/yr** (hurts)
  - XLK would need ≈ **13.8%/yr** versus ≈ 12.3% assumed
  - risk aversion would need to be < ≈ **2.3**
  - helps in **1 of 6** windows (only the last 10 years)
  - helps in ≈ **27%** of the 60 scenarios
  - action: **keep current**

  Moving the return slider up about 1.5 points flips it. That's exactly the live moment we want.
- **Performance is not a concern:**
  - a full audit took about 0.3 s
  - a 2-D map took about 0.05 s, so sliders can call the server live
  - walk-forward took about 2 s, so put it behind a button
- **Honest risk for the pitch:** in the prototype's walk-forward (Jul 2010 to Nov 2026):

  | Strategy | CAGR | Volatility | Turnover |
  |---|---|---|---|
  | Audited | ≈ 9.5% | ≈ 12.9% | ≈ 3.5 |
  | Do nothing | ≈ 13.6% | ≈ 16.0% | 0 |
  | Follow | ≈ 13.9% | ≈ 17.2% | ≈ 12.6 |

  The audited version was much calmer and traded far less, but returned less. Sunny should investigate this (window choice, γ, the recommender, the step rule) and the deck must present it honestly, not hide it.
- **The data source works:** daily adjusted closes since Jan 2007 for all 11 funds, about 600 KB as CSV.
- **The expected-return numbers we used were placeholders** (SPY 6.5, XLK 7.0, IWM 7.0, XLV 6.5, VEA 7.0, VWO 7.5, VNQ 6.5, GLD 3.5, AGG 4.5, TLT 4.5, SHY 3.8 %/yr). **They must be replaced with a cited published source** before the demo (Sunny).

---

## 5. Who owns what

Each person owns a vertical slice and its tests. If someone's skills fit another slice better, swap on day 1 and update this table and the `context/` files.

### Parth (`parth`): data, API, report, explanations, integration lead
- Price fetch script + committed snapshot + data fingerprint.
- Universe file format, loaded by everyone.
- FastAPI app implementing [API_CONTRACT.md](API_CONTRACT.md); 422 errors with readable messages; CORS for the frontend.
- **By day 2:** publish sample JSON responses in `docs/api-samples/` so JP can build against them before the engine exists.
- Template explanations, where every sentence is filled from computed numbers. Includes a grounding test: the numbers in the text must match the report.
- Decision report (JSON + Markdown) with the reproducibility block and report ID.
- Owns merges into `main` and the integration checkpoints.
- **Done when:** every endpoint in the contract works end to end on the demo case, the report ID is stable across runs, and the explanations pass the grounding test.

### Aha (`aha`): audit engine
- Input normalisation (dollars or weights → weights), the "set fund to X%" helper, focus selection, input validation.
- The objective, cost model and helps/neutral/hurts classification.
- The five tests, the four closed-form break-evens, the 60-scenario grid, smaller-adjustment rows, the recommended step, the verdict.
- The 2-D assumption map over any two of: return shift, cost, risk aversion, horizon, blend weight, window.
- **Tests that prove the maths:** plug each break-even back in and the net benefit must be 0; costs can only ever lower net benefit; map cells must equal single-point evaluations; the recommended step must really be the best 25th percentile.
- **Done when:** those tests pass and the demo case reproduces the shape of the §4 numbers.

### Sunny (`sunny`): estimation windows, validation, evidence
- Estimation module: the 6 windows (also buildable "as of" a past date for validation), μ modes, Σ.
- **Source the long-run expected returns** from a published capital-market-assumptions document and cite it in the universe file and the report.
- Walk-forward validation with no look-ahead: the recommender, the three strategies, all metrics, monthly equity curves.
- Investigate the audited-vs-do-nothing gap from §4. Try a few disclosed variants and report all of them, not just the best.
- Write `docs/EVALUATION.md`: method, results table, limitations. This is the "how do we know it works" slide.
- Run **user testing** with 3–5 people in week 2: can they say what the verdict means and what would flip it?
- **Done when:** validation runs from the API, there's a no-look-ahead test, EVALUATION.md is written, and the user-testing notes are in.

### JP (`jp`): frontend
Single-page flow, one component per section:
1. **Setup:** holdings editor, proposed change, prefilled demo.
2. **Verdict:** headline, summary, action, current / proposed / recommended weights.
3. **Find the breaking point** (hero):
   - sliders shown in human terms (e.g. "XLK expected return 12.3%/yr"), with the break-even marked on the track
   - live calls to the evaluate endpoint (debounced, stale requests cancelled)
   - the live "Below X…" sentence
4. **Assumption map:** heatmap with axis pickers, base point marked, model + assumptions printed under it.
5. **The tests:** windows chart, return curve with break-even, cost table, smaller-steps chart.
6. **History check:** run button, equity curves, metrics table, notes.
7. **Report:** findings, reproducibility block, Markdown/JSON download.

- Colourblind-safe helps/neutral/hurts colours; works at laptop and phone widths; loading, error and empty states everywhere.
- Build against `docs/api-samples/` until the real API is up. Configure the API URL with an environment variable.
- **Done when:** the full demo script (§8) runs in the browser against the real backend without touching devtools.

---

## 6. Timeline

| Date | Day | Milestone |
|---|---|---|
| Wed 18 Nov | 1 | Kickoff: walk through this plan, confirm owners, freeze [API_CONTRACT.md](API_CONTRACT.md) v1, everyone creates their folders on their branch |
| Thu 19 Nov | 2 | Parth: data snapshot + sample API JSON on main. Aha: objective + base case. Sunny: windows + estimation, start sourcing return assumptions. JP: app shell + setup section on mock data |
| Fri 20 Nov | 3 | Aha: break-evens + cost test + tests. Sunny: return assumptions cited. JP: verdict + sliders on mock data. Parth: API skeleton serving the engine |
| Sat 21 – Sun 22 Nov | 4–5 | **Integration #1 (Sun):** real end-to-end audit of the demo case through the UI. Merge all branches to main |
| Mon 23 Nov | 6 | Aha: smaller steps, scenario grid, verdict, map. Sunny: walk-forward. JP: map + tests sections. Parth: explanations + report |
| Tue 24 Nov | 7 | **Integration #2 + feature freeze.** Every section works on real data. Merge to main |
| Wed 25 – Fri 27 Nov | 8–10 | Thanksgiving, light days. Bug fixes only; Sunny runs user testing where people are available |
| Sat 28 Nov | 11 | Fix what user testing found. Sunny: EVALUATION.md. Parth: report polish. JP: visual polish + mobile check |
| Sun 29 Nov | 12 | Demo script locked, deck drafted, first full rehearsal |
| Mon 30 Nov | 13 | Second rehearsal, record a backup demo video, final merge to main, tag a release |
| Tue 1 Dec | 14 | Submit / present |

**Rules:** merge to main at least at each integration point. Only Parth merges into main (to avoid four people resolving conflicts four ways). Every push to main must keep the demo working.

---

## 7. Stretch goals (only after feature freeze is green)
1. An LLM rephrasing layer for the explanations. It may only reword the template findings, and the grounding test must still pass (no numbers that aren't in the report).
2. A third map axis via small multiples.
3. A tax-drag assumption in the cost model.
4. Additional funds in the universe.

---

## 8. Demo script (≈ 3 minutes)
1. **Story:** "An investing tool told me to raise tech from 20% to 40%." The demo portfolio is prefilled; click Run.
2. **Verdict:** it hurts under our base assumptions and only helps in about a quarter of the tested scenarios. Read the one-line summary aloud.
3. **Hand the judge the slider:** "What do you think tech will return?" They drag it up. Past the break-even the verdict flips live, and the sentence updates.
4. **Map:** show the whole region where the change helps vs hurts, with the model printed underneath.
5. **Smaller step:** show whether a partial move holds up better than the full one or doing nothing.
6. **History check:** show the three strategies honestly, including where the audited version gave up return.
7. **Report:** download it and point at the report ID + data hash: "Run it again tomorrow, same answer."
8. **Close with the pitch line and the limits:** our model, not theirs; not advice; not a new financial method.

---

## 9. Risks

| Risk | Mitigation |
|---|---|
| Yahoo or yfinance breaks during the 2 weeks | We use the committed CSV snapshot; refetch is optional |
| Return assumptions look made up | Sunny cites a published source; the report shows it |
| Validation shows audited < do nothing | Present it honestly; frame the product as decision transparency, and show the vol/turnover/cost trade-off |
| "Isn't this just PortfolioPilot?" | Point to the do-nothing baseline, reversal points, smaller steps, reproducible report ([CONCEPT.md](CONCEPT.md)) |
| Integration crunch | Contract frozen on day 1, sample JSON on day 2, two integration days, freeze before Thanksgiving |
| Someone gets blocked | Write it under "Blocked on" in your `context/` file on main; Parth checks those every morning |
