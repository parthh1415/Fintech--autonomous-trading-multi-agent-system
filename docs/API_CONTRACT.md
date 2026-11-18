# API contract v1 (freeze on day 1)

Owner: **Parth** (implements it) · Consumer: **JP** · Engine behind it: **Aha** + **Sunny**

Change this file only by agreement in the group chat. Bump the version at the top, and note the change in your `context/` file.

## Units (the #1 source of integration bugs)
- **Inputs** use human units:
  - trading cost in **bps**
  - return shift in **percentage points**
  - neutral band in **% per year**
  - target weight in **%**
  - holdings in **dollars or weights** (the server normalises them to 100%)
- **Outputs** use decimals:
  - every return, volatility and benefit is an **annual decimal** (0.01 = 1% a year)
  - every weight is a **fraction** (0.25 = 25%)
- Outcome labels are always exactly `helps`, `neutral` or `hurts`.
- Errors return HTTP **422** with `detail`. That's a readable string for business errors (unknown ticker, current = proposed, bad window) or FastAPI's validation list for schema errors. The frontend must handle both.

## Shared request body ("audit request")

| Field | Type | Notes |
|---|---|---|
| current | map ticker → number | required; dollars or weights |
| proposed | map ticker → number | send **exactly one** of `proposed` / `change` |
| change | object: ticker, target_weight (%) | "set this fund to X%"; the other funds scale proportionally |
| focus | ticker, optional | asset whose return gets stress-tested; default = biggest weight increase |
| assumptions | object, optional | fields below; any missing field uses its default |

**assumptions fields:**

| Field | Default | Range |
|---|---|---|
| mu_mode | blend | historical, assumptions, blend |
| blend_weight | 0.5 | 0–1 (weight on history) |
| window | recent_5y | one of the window ids from `/api/universe` |
| risk_aversion | 3 | > 0, ≤ 50 |
| cost_bps | 10 | 0–1000 |
| horizon_years | 3 | 0.25–30 |
| return_shift | 0 | −20 to 20 (pp, applied to the focus asset) |
| neutral_band | 0.10 | 0–5 (% per year) |

## Endpoints

| Method + path | Purpose | Speed target |
|---|---|---|
| GET /api/health | ok, version, last data date | instant |
| GET /api/universe | assets (ticker, name, group, expected_return in %), assumption source note, windows (id, label, start, end), default assumptions, map axes (id, label, default values), data fingerprint, demo (current, change, story) | instant |
| POST /api/audit | full decision report (below) | < 0.5 s |
| POST /api/evaluate | single point for live sliders: focus, assumptions, result (same fields as `base` below), thresholds | < 50 ms |
| POST /api/map | audit request + x_axis, y_axis (two different of: return_shift, cost_bps, risk_aversion, horizon_years, blend_weight, window) + optional x_values / y_values; max 2,500 cells | < 0.3 s |
| POST /api/validate | current, assumptions, lookback_years (3), rebalance_months (12), max_weight (0.6) | < 5 s, run from a button |
| POST /api/report.md | audit request → Markdown report as text | < 0.5 s |

## `/api/audit` response, top-level fields

| Field | Contents |
|---|---|
| focus | ticker |
| assumptions | echo of the assumptions actually used |
| base | delta_expected_return, delta_volatility, delta_risk_penalty, gross_benefit, turnover, one_off_cost, cost_drag, net_benefit, outcome, focus_expected_return, utility_current, utility_proposed, current {expected_return, volatility, variance}, proposed {same} |
| portfolios | current / proposed / recommended, each: weights, expected_return, volatility, risk_adjusted_return, largest_holding, largest_weight, effective_holdings, group_weights |
| thresholds | focus, focus_weight_change, return_breakeven {required_focus_return, assumed_focus_return, shift_to_breakeven, direction: above/below} or null, cost_breakeven_bps or null, risk_aversion_breakeven {value, helps_when: below/above} or null, min_horizon_years or null |
| windows | list: id, label, start, end, is_base, net_benefit, delta_expected_return, delta_volatility, outcome, focus_historical_return, focus_expected_return |
| return_curve | list: shift, focus_expected_return, net_benefit, outcome (shifts −5 to +5 pp in 0.25 steps) |
| costs | list: cost_bps, is_base, one_off_cost, net_benefit, outcome (0, 5, 10, 25, 50, 100, 200 + the base) |
| smaller_steps | rows (alpha, weights, base_net_benefit, share_of_full_benefit, turnover, largest_weight, hit_rate, hurt_rate, p25_net_benefit, median_net_benefit, worst_net_benefit), recommended_alpha, smallest_alpha_for_80pct, scenario_count |
| verdict | code (HURTS, NO_MEANINGFUL_BENEFIT, ROBUST, FRAGILE, TOO_UNCERTAIN), headline, hit_rate, hurt_rate |
| action | kind (keep_current, smaller_change, proposed_change), alpha, weights |
| explanation | summary, findings: list of {id, title, text, evidence}. ids: base_case, what_needs_to_be_true, windows, robustness, smaller_change, limits |
| reproducibility | report_id, engine_version, generated_at, inputs, data {prices_sha256, first_date, last_date, trading_days, tickers, source, assumption_source}, model {formulas + rules as text}, units |

## `/api/map` response
x {axis, label, values}, y {same}, net_benefit (rows = y, columns = x), outcome (same shape), base_point {x, y}, counts {helps, neutral, hurts}, model (text), assumptions.

## `/api/validate` response
- **period** {start, end}, **rebalances**
- **strategies**: do_nothing, follow, audited. Each has cagr, volatility, return_per_risk, max_drawdown, total_turnover, total_cost, trades, allocation_instability, final_value, and curve (monthly date + value).
- **decisions**: one per rebalance date: date, recommended weights, audited_alpha.
- **recommender** (description text) and **notes**.

## Sample responses
Parth commits real example responses for the demo request to `docs/api-samples/` by **day 2**. JP builds against those until the live API is up.
