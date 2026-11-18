# Working in the HEDGEWISE repo (humans and coding agents)

Read [README.md](README.md), [docs/CONCEPT.md](docs/CONCEPT.md), [docs/PLAN.md](docs/PLAN.md) and [docs/API_CONTRACT.md](docs/API_CONTRACT.md) before doing anything.

## Branches
- Four working branches, one per person: `parth`, `aha`, `sunny`, `jp`. Work **only** on the branch for the person you're working for, and only in the areas PLAN.md §5 gives them.
- `main` is the integration branch. Only Parth merges branches into main. The one exception is the context update below, which everyone does directly on main.
- Don't change [docs/API_CONTRACT.md](docs/API_CONTRACT.md) or the decisions in PLAN.md §3 on your own. Propose the change in your context file under "Needs from others" instead.

## The context protocol (main always knows what every branch last did)

`context/<branch>.md` on **main** is the single source of truth for what that branch last worked on. Each person (or their agent) edits **only their own file**, so these updates never conflict.

**At the start of every session**
1. Fetch, then read all four files as they are on main (for example with `git fetch origin` and `git show origin/main:context/aha.md`). That way you know what the others did, what they need from you, and whether the contract changed.
2. Check "Needs from others" in the other three files for anything addressed to you.

**At the end of every session (or every ~2 hours of work)**
1. Commit your work on your own branch.
2. Switch to main and pull it. Edit **only** `context/<your-branch>.md`:
   - **Last updated:** date + time.
   - **Last worked on:** 2–5 bullets of what you actually did this session, with the branch commit hash.
   - **Status of my slice:** keep the task checklist current.
   - **Next up**, **Blocked on**, **Needs from others** (tag the person).
   - Add one dated line to the **Log** at the bottom. Never delete old log lines.
3. Commit on main with the message `context(<branch>): <one-line summary>`. Push main **only** if your person has said pushing is OK. Then switch back to your branch.

If you're a coding agent: never end a session without doing step 2. If you can't (merge trouble, no permission to push), say so clearly in your final message.

## Engineering rules
- Units follow the contract: human units in, annual decimals and weight fractions out.
- Every number the user sees must be computed, testable and reproducible. Never hard-code results in the UI or in explanation text.
- An LLM may only rephrase verified findings. It never produces numbers.
- No look-ahead in validation: a decision at date *t* only sees data before *t*.
- Write comments like a teammate would: short and specific, only where they help. No boilerplate.
- Never commit secrets, API keys or `.env` files.
- Present results honestly, including the ones that don't flatter us (see PLAN.md §4).
