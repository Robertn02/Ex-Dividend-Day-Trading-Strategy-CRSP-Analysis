# Dividend Capture on Ex-Dividend Day: Evidence from CRSP

A compact, reproducible study of the classic **dividend capture** strategy: buy at the close before the ex-dividend day, sell after the ex-day (close or open), and test profitability with and without transaction costs. We follow the FBE-551 HW3 brief and turn it into a standalone project.

---

## Overview

- **Goal:** Evaluate whether ex-dividend price behavior supports a profitable trading strategy.
- **Core questions:**
  1) How large is the price drop relative to the dividend ((P_{t-1}−P_t)/D)?
  2) What are the annualized returns, volatility, and Sharpe of a close→close strategy?
  3) What are **abnormal returns** vs. CAPM (using lagged annual betas)?
  4) What if we trade **close→open** instead of holding the day?
  5) Do **round-trip costs** (bid–ask) wipe out profits?
  6) Can **sorting** (e.g., by dividend yield) improve performance?

---

## Data

Place these files in the project root:

- `CRSP_Dividends.csv` — Ex-day events (common shares only; includes `PRC`, `RET`, `RETX`, `OPENPRC`, `BID`, `ASK`, `vwretd`, `VOL`, etc.)
- `Yearly Betas.csv` — Stock-level CAPM betas measured at year-end (pre-event, to avoid look-ahead)
- `^IRX.csv` — 3-Month T-Bill (^IRX) daily series to build daily **rf**
  
> **Note:** SHROUT is in thousands in CRSP; we convert `mkt_cap = PRC × SHROUT × 1000`.

---

## Methods

1. **Clean & Filters**
   - Make `PRC = |PRC|` (CRSP convention for non-trading days).
   - Drop tiny dividends `< $0.01`.
   - Filter out microcaps: `mkt_cap ≥ $50M`.
   - Create `vol_flag = 1` if `VOL > 0` else `0` (we keep both, analyze both).

2. **Prior Close & Ex-Day Drop**
   - Compute `P_{t-1} = Pt / (1 + RET)` (use **total** return for close→close).
   - Ratio: `(P_{t-1} − P_t) / DIVAMT`; describe mean/median/range.
   - Strategy (close→close): annualized return, stdev, Sharpe (daily rf from ^IRX).

3. **Abnormal Returns vs. CAPM**
   - Merge **lagged** betas (year = event year − 1).
   - Expected daily: `E[r] = rf + β (Rm − rf)`, with `Rm = vwretd`.
   - Abnormal: `RET − E[r]`; report mean + t-stat; annualize mean.

4. **Close→Open Variant**
   - `P_{t-1}` from **RETX** for ex-dividend logic.
   - `r_{c→o} = (OPENPRC + DIVAMT − P_{t-1}) / P_{t-1}`.
   - Annualized return, stdev, Sharpe.

5. **Transaction Costs**
   - Round-trip cost: `(ASK − BID)/((ASK + BID)/2)` when both quotes exist.
   - Net close→close: `RET − cost`; recompute stats.

6. **Cross-Sectional Sort**
   - Example: sort by **dividend yield** `DIVAMT / P_{t-1}` into deciles.
   - Compute returns per decile; assess monotonicity and where performance concentrates.

Each analysis is run **twice**:
- **Full Sample** (keep `VOL=0` rows)
