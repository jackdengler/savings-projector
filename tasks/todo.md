# Life-plan upgrade: buy the house, scenarios, levers, life events

Picked from the rethink: **#1 buy the house**, **#3 compare scenarios**,
**#4 levers that matter**, **#5 life events**. Branch `claude/life-plan`
builds on `claude/fix-years-crash-her-401k-cap`. It stays one file with no
build step.

## Guiding decisions (defaults; flag any you disagree with)

- **Headline becomes net worth.** Net worth = retirement + liquid + home
  equity, where home equity = value − loan balance − 6% selling cost. Counting
  equity net of selling cost keeps rent-vs-buy fair in the early years.
- **Rent comes out of fixed costs.** A new "of which rent" slider (default
  $3,000 of the $5,000 fixed) marks the part that stops when you buy. If you
  never buy, the math doesn't change.
- **Itemized deductions split 50/50** between the two single filers. Each
  person takes the larger of itemized or standard, federal and CA separately.
- **Storage stays backward compatible.** New fields are flat keys with defaults
  (`{...DEFAULTS, ...saved}` already merges them). Scenarios live under a
  separate key, `savings_scenarios_v1`, so `savings_inputs_v2` keeps its shape.
- **Everything already in the app keeps its numbers.** With no house, no events
  and no scenarios, year 1 is still $143,284 and the year-by-year table matches
  `main` exactly (after the two bug fixes).

## Phase 0: Pull the math out so it can be tested (no behavior change)
- [ ] Move the tax engine, `computeYear`, `levelPath` and the projection loop
      into a plain `<script>` block that exposes `window.SPEngine = { project, … }`.
      `project(inputs)` returns `{ rows, summary }`, and all the math lives there.
- [ ] The React component calls `project(s)`, so the render stays render-only.
- [ ] `tests/engine.test.js` (node:test, no dependencies) loads the engine block
      from `index.html` through `vm`. Snapshot tests: default inputs, the
      her-401(k) cap, years = 0, and today's-dollars mode.
- [ ] Browser check: the default page and table match `main` cell for cell.

## Phase 1: Life events (#5)
New **Life events** section below Raises & overrides. Raises keep working
exactly as today. Three kinds of event:
- [ ] **Recurring cost**: $/mo starting in year A for N years (childcare, a car
      payment). It grows with expense growth.
- [ ] **One-time**: +/− $ in year A (car, wedding, windfall, inheritance). It
      hits liquid savings.
- [ ] **Income change**: a person at X% of income for years A to B (sabbatical,
      part-time, job loss). Temporary: the income path resumes afterward, unlike
      a raise pin, which re-bases income.
- [ ] Taxes follow the lower income. The 401(k) and employer match scale with
      salary.
- [ ] Event years are marked in the table (the same • that pins use), and the
      tooltip lists them.
- [ ] Tests: each kind, overlapping events, an event past the horizon (ignored).

## Phase 2: Actually buy the house (#1)
The **Mortgage capacity** section becomes **Home**: capacity stays as the
helper, and a purchase can be switched on.
- [ ] Inputs: buy on/off, buy year, price (0 = the most you can afford that
      year, from the existing solver), down % (default 20%; the solver sets it
      in max mode), home appreciation (default 3.5%), maintenance (default
      1%/yr of value), rent portion (see decisions above).
- [ ] Buy year: liquid savings pay the down payment and closing costs, and the
      loan is price − down. If cash above the emergency fund falls short, or
      the payment breaks the lender ratio or your payment cap, show a clear
      warning and still model it, so you see the damage.
- [ ] After buying: fixed P&I on a 30-year schedule, with the loan balance
      tracked. Property tax follows Prop 13: the rate applies to the purchase
      price, and the assessed value grows at most 2%/yr. Insurance grows with
      expense growth. Maintenance is a % of current value. PMI applies until
      the balance reaches 78% of the original price. Rent stops.
- [ ] Taxes: mortgage interest (federal limit $750k of loan, CA $1M) and
      property tax become itemized deductions. Federal SALT cap: *verify the
      current post-2025 figures and phase-down before coding.* Each filer takes
      max(itemized, standard).
- [ ] Chart: a home-equity layer on top of retirement + liquid. When a purchase
      is on, a dashed **"keep renting"** line shows the same plan with no
      purchase, so rent vs buy is visible at a glance.
- [ ] Table: new Home group (value, loan, equity, housing cost/mo) and a Net
      worth column. The saved image gets the same columns.
- [ ] Tests: amortization matches a known schedule, PMI drop-off year, the
      Prop 13 cap, rent removal, an itemized-vs-standard crossover, and
      buy-off matching Phase 0 exactly.

## Phase 3: What moves the needle (#4)
New section right under the chart.
- [ ] Re-run `project()` with one lever nudged at a time, then rank by the
      change in horizon net worth. Levers: spending −$500/mo, return +1%, your
      base +10%, her salary +10%, income growth +1%, expense growth −1%, big
      purchases −$5k/yr, maxing her 401(k), and buy one year later/earlier
      (when buying).
- [ ] Horizontal bars in the existing tokens, with the change in net worth. When
      buying, also show "house affordable N yrs sooner".
- [ ] Respects today's-dollars mode. Performance: about 12 projections of 30
      years each, which is trivial.

## Phase 4: Compare scenarios (#3)
- [ ] **Save as scenario** stores a named snapshot of the current inputs (up to
      5). Each saved scenario is a chip you can show/hide on the chart, load
      (with confirm) or delete.
- [ ] Visible scenarios overlay as labeled dashed net-worth lines, with at most
      3 shown at once for legibility.
- [ ] Compare strip: each scenario's horizon net worth, buy year / affordable
      price, and year-1 savings rate, next to the current plan.
- [ ] Persisted under `savings_scenarios_v1` (localStorage fallback same as now).

## Phase 5: Finish
- [ ] Update the README and the footer notes for the new model and its limits.
- [ ] Verify light and dark themes and phone width (375px) with no horizontal
      page scroll.
- [ ] Browser check with Playwright of each feature end to end, plus engine tests.
- [ ] One commit per phase, one PR.

## Not in scope
Monte Carlo, goal-seek, actuals tracking, a filing-status switch, HSA/health
premiums. Separate follow-ups if wanted.

## Review
_(filled in after implementation)_
