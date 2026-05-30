# savings-projector

A single-file savings & growth projector for a two-income Los Angeles household.

`index.html` is the projector page that renders inside the PWA (and works
standalone from GitHub Pages too). It's a self-contained React app — React,
ReactDOM and Babel load from CDN, so there's no build step.

## What it does

- **Verified 2026 tax engine** (single filer, each spouse files separately):
  IRS Rev. Proc. 2025-32 federal brackets, $16,100 standard deduction,
  $184,500 Social Security wage base, 1.45% + 0.9% Medicare, $24,500 401(k) /
  $7,500 IRA limits, plus California's latest FTB single brackets, $5,706
  standard deduction and 1.3% SDI. Brackets and limits index by inflation
  across the projection.
- **Year-by-year projection** of retirement vs. liquid balances, savings rate,
  and total growth.
- **Mortgage capacity** — how much home the household can afford each year
  given liquid savings (above an emergency fund) and a lender front-end ratio,
  with PMI applied automatically when the down payment is under 20%.

## Storage

Inputs auto-save through `window.storage`. Inside the PWA shell the host
injects `window.storage`, which persists to the private `private-data-storage`
repository under the key `savings_inputs_v2`. When opened standalone, the page
falls back to `localStorage` so it still remembers your inputs.
