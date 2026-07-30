# NSE Momentum Classification Framework
## Institutional-Grade 5-Bucket System | Stocks ≥ ₹10,000 Cr Market Cap

---

## UNIVERSE DEFINITION

**Eligible securities:**
- NSE-listed equities only (exclude ETFs, REITs, InvITs, preference shares)
- Market cap ≥ ₹10,000 Cr on classification date (weekly check)
- Minimum 63 continuous trading days post-listing
- Not under F&O ban for >10 of prior 21 trading days
- Free-float ≥ 15% (low free-float distorts volume signals)

**Market cap maintenance rule:**
- Sub-threshold stocks retained 21 trading days post-breach before exclusion
- Re-entry: 21 trading days above threshold continuously

**Price series:**
- Use **dividend-adjusted close** (total return basis) throughout
- Source: NSE Bhav Copy + corporate action table, cross-validated against Adjusted Close from primary data vendor
- All prices in INR, not rebased

---

## SECTION 1 — MULTI-TIMEFRAME PRICE MOMENTUM ENGINE

### 1.1 Lookback Windows (NSE trading days, not calendar days)

| Symbol | NSE Trading Days | Calendar Approx | Role |
|--------|-----------------|-----------------|------|
| T1 | 21 | 1 month | Short-term signal / reversal proxy |
| T2 | 63 | 3 months | Primary tactical window |
| T3 | 126 | 6 months | Intermediate trend |
| T4 | 231 | ~11 months | Long-term momentum (skip T1: use P(t-21) as numerator) |

**T4 design intent:** Classic Jegadeesh-Titman (1993) skip-period.
Uses `P(t−21)` as current price to eliminate 1-month reversal contamination.

### 1.2 Raw and Excess Return Calculations

```
R_raw(i, t, n)  = P_adj(i, t) / P_adj(i, t-n) - 1

R_nifty(t, n)   = P_nifty(t) / P_nifty(t-n) - 1        # Nifty 50 TRI

R_sector(i,t,n) = cap-weighted return of stock i's NSE sector over n days
                  (NSE-AMFI GICS equivalent, 13 categories)

R_ewu(t, n)     = equal-weighted avg of all eligible universe stocks over n days

ER_nifty(i,t,n)  = R_raw(i,t,n) - R_nifty(t,n)
ER_sector(i,t,n) = R_raw(i,t,n) - R_sector(i,t,n)
ER_peer(i,t,n)   = R_raw(i,t,n) - R_ewu(t,n)
```

**Composite excess return (CER):**
```
CER(i,t,n) = 0.40 * ER_nifty(i,t,n)
           + 0.35 * ER_sector(i,t,n)
           + 0.25 * ER_peer(i,t,n)
```

### 1.3 Volatility-Adjusted Momentum (Sharpe-Like Ratios)

**Realized volatility — 63-day rolling:**
```
sigma_63(i,t) = std( ln(P(i,τ)/P(i,τ-1)), τ ∈ [t-62, t] ) * sqrt(252)
```
Use log returns for vol estimation regardless of return metric.

**Sharpe-momentum ratio per window:**
```
SMR(i,t,n) = CER(i,t,n) / sigma_63(i,t)

where sigma_63 is annualized; CER(i,t,n) is NOT annualized
→ result is not strictly Sharpe but preserves cross-sectional rank
```

**Floor on sigma:** `max(sigma_63, 0.10)` — prevents division-by-near-zero for suspended/illiquid periods.

### 1.4 Multi-Timeframe Score (MTS) — Weighted Composite

```
MTS(i,t) = w1 * SMR(i,t,T1)
          + w2 * SMR(i,t,T2)
          + w3 * SMR(i,t,T3)
          + w4 * SMR(i,t,T4)

Weights:
  w1 = 0.10   (T1 = 21d; low weight, acts as reversal check)
  w2 = 0.25   (T2 = 63d)
  w3 = 0.35   (T3 = 126d; primary signal, highest weight)
  w4 = 0.30   (T4 = 231d skip-period)
```

**Weight redistribution for stocks with <252d history:**
```
if listing_days < 252 and listing_days >= 126:
    w2_adj = 0.30, w3_adj = 0.40, w4_adj = 0.00, w1_adj = 0.10  (T4 not computed, weight to T2/T3)

if listing_days < 126 and listing_days >= 63:
    w2_adj = 0.55, w3_adj = 0.00, w4_adj = 0.00, w1_adj = 0.15  (T3, T4 not computed)
    apply 20% confidence haircut to final MTS
```

### 1.5 Cross-Sectional Normalization

**Z-score MTS within universe (weekly cross-section):**
```
Z_MTS(i,t) = ( MTS(i,t) - mean(MTS, universe_t) ) / std(MTS, universe_t)
```

Winsorize Z_MTS at ±3.0 before use in composite.

---

## SECTION 2 — TREND STRUCTURE & MARKET MICROSTRUCTURE

### 2.1 Moving Average Hierarchy

**MA types:** Exponential Moving Averages (EMA), not SMA.
EMA decay: standard — alpha = 2/(n+1).

**MA set:** EMA20, EMA50, EMA100, EMA200

**Full bull stack condition:**
```
EMA20(t) > EMA50(t) > EMA100(t) > EMA200(t)   → TRUE/FALSE
```

**MA slope confirmation:**
```
Slope(EMA_n, t, k) = ( EMA_n(t) - EMA_n(t-k) ) / EMA_n(t-k)

Lookback k per MA:
  EMA20  → k = 10
  EMA50  → k = 21
  EMA100 → k = 42
  EMA200 → k = 63
```

**Slope threshold:** `> 0.0` = rising, `< 0.0` = falling.
Use `> +0.5%` and `< -0.5%` as meaningful signal (not flat noise).

**MA Structure Score (MASS) — 0 to 8:**
```
MASS = Σ (alignment_points + slope_points)

Alignment (each pair scores 1 if in correct order):
  EMA20 > EMA50   → 1 pt
  EMA50 > EMA100  → 1 pt
  EMA100 > EMA200 → 1 pt
  Price > EMA20   → 1 pt    [current close above fastest MA]

Slope (each MA scores 1 if slope > +0.5%):
  Slope(EMA20, 10)  > 0.5% → 1 pt
  Slope(EMA50, 21)  > 0.5% → 1 pt
  Slope(EMA100, 42) > 0.5% → 1 pt
  Slope(EMA200, 63) > 0.5% → 1 pt

MASS range: 0–8
```

**Normalize:** `Z_MASS = MASS / 4.0 - 1.0` → maps to [-1.0, +1.0]

### 2.2 Volume-Profile Confirmation

**On-Balance Volume Trend:**
```
OBV(t) = OBV(t-1) + volume(t) * sign(close(t) - close(t-1))

OBV_slope(t) = slope of OBV linear regression over 63 days
             = (OBV(t) - OBV(t-63)) / 63     [simplified; use OLS for precision]
```

**Up-Volume Ratio (UVR) — 21-day:**
```
UVR(t) = Σ(volume(τ) for τ where close(τ) > close(τ-1), last 21d) /
          Σ(volume(τ), last 21d)

UVR > 0.60 → accumulation (institutional buying signature)
UVR < 0.40 → distribution
0.40–0.60  → neutral
```

**Volume Expansion Ratio (VER):**
```
VER(t) = MA(volume, 20d) / MA(volume, 63d)

VER > 1.15 → expanding participation (confirms breakouts)
VER < 0.85 → shrinking participation (weakening trend)
```

**Price-OBV Divergence Flag:**
```
price_new_63d_high = (close(t) == max(close, last 63d))
obv_new_63d_high   = (OBV(t) == max(OBV, last 63d))

confirmed_breakout = price_new_63d_high AND obv_new_63d_high      → +1
bearish_divergence  = price_new_63d_high AND NOT obv_new_63d_high → -1
neutral             = otherwise                                    →  0
```

**Volume Score (VS) — normalized to [-1, +1]:**
```
VS = 0.40 * normalize(UVR, 0.40, 0.60)
   + 0.35 * normalize(VER, 0.85, 1.15)
   + 0.25 * divergence_flag     [values: -1, 0, +1]

normalize(x, lo, hi): linear map → x≤lo: -1, x≥hi: +1, else linear interp
```

### 2.3 Drawdown Recovery & Time-Underwater Metrics

**Drawdown from 252-day high:**
```
ATH_252(i,t)  = max(close(i, τ), τ ∈ [t-251, t])
DD(i,t)       = (close(i,t) - ATH_252(i,t)) / ATH_252(i,t)   [always ≤ 0]
```

**Recovery factor:**
```
RF(i,t) = -DD(i,t) / sigma_63(i,t)
         [ratio of % drawdown to annualized vol; ≈ sigma-multiples below ATH]
```

**Time-Underwater (TUW):**
```
TUW(i,t) = number of consecutive trading days where close < ATH_252
```

**Drawdown Score (DS):**
```
DD_score = f(DD):
    DD > -0.05              → +1.0    (within 5% of ATH; strong trend)
   -0.05 >= DD > -0.10      → +0.5
   -0.10 >= DD > -0.20      →  0.0
   -0.20 >= DD > -0.30      → -0.5
    DD <= -0.30             → -1.0    (broken trend)

TUW_score = g(TUW):
    TUW < 21                → +0.5
    21 <= TUW < 63          →  0.0
    63 <= TUW < 126         → -0.5
    TUW >= 126              → -1.0

DS = 0.60 * DD_score + 0.40 * TUW_score    [range ≈ -1 to +1]
```

### 2.4 Composite Trend Score (Z_TREND)

```
TREND_raw = 0.45 * Z_MASS + 0.35 * VS + 0.20 * DS

Z_TREND = ( TREND_raw - mean(TREND_raw, universe) ) / std(TREND_raw, universe)
```
Winsorize at ±3.0.

---

## SECTION 3 — RELATIVE STRENGTH & SECTOR ROTATION

### 3.1 RS Score Construction

**Raw RS ratio:**
```
RS_ratio(i,t,n) = (1 + R_raw(i,t,n)) / (1 + R_nifty(t,n))
```

**Percentile rank within universe:**
```
RS_pct(i,t,n) = percentile_rank(RS_ratio(i,t,n), all eligible stocks) ∈ [0, 100]
```

**Composite RS (CRS):**
```
CRS(i,t) = 0.20 * RS_pct(i,t,T1)
          + 0.40 * RS_pct(i,t,T2)
          + 0.40 * RS_pct(i,t,T3)
```
Note: CRS intentionally excludes T4 — RS is a tactical signal, not structural.

**Normalize:**
```
Z_RS(i,t) = ( CRS(i,t) - 50 ) / std(CRS, universe)
```

### 3.2 Sector Momentum & Leadership

**Sector Momentum Score (SMS):**
```
SMS(s,t,n) = cap-weighted avg return of all eligible stocks in sector s over n days

SMS_rank(s,t) = percentile_rank(SMS(s,t,63), all sectors)    [13 NSE sectors]
```

**Stock-level sector adjustment:**
```
sector_bonus(i,t) = f(SMS_rank(sector_of(i), t)):
    SMS_rank >= 75   → +0.30   (stock benefits from sector tailwind)
    SMS_rank 50–75   → +0.10
    SMS_rank 25–50   → -0.10
    SMS_rank < 25    → -0.30   (sector headwind)
```

### 3.3 Sector Leadership Transition Detection

**Regime signal:**
```
SMS_accel(s,t) = SMS(s,t,63) - SMS(s,t,126)/2
                [3m return minus half of 6m → positive if accelerating]

Emerging leadership:  SMS_accel > +3% AND SMS_rank(63d) > SMS_rank(126d)+15pct
Deteriorating:        SMS_accel < -3% AND SMS_rank(63d) < SMS_rank(126d)-15pct
Stable:               otherwise
```

**Penalty/boost adjustment to Z_RS:**
```
if emerging leadership sector:
    Z_RS(i,t) += 0.20
if deteriorating sector:
    Z_RS(i,t) -= 0.20
```

### 3.4 Industry Group Peer Decile

**Peer percentile within same NSE sector:**
```
peer_pct(i,t) = percentile_rank(CRS(i,t), stocks in same sector) ∈ [0, 100]

peer_score(i,t) = f(peer_pct):
    >= 90  → +0.40
    75–90  → +0.20
    25–75  →  0.00
    10–25  → -0.20
    < 10   → -0.40
```

**Final RS component:**
```
Z_RS_final(i,t) = Z_RS(i,t) + peer_score(i,t) + sector_bonus(i,t)
```
Re-winsorize at ±3.0 after adjustments.

**Rebalancing frequency:** Weekly (Friday close data, scores computed Saturday pre-market).

---

## SECTION 4 — FUNDAMENTAL MOMENTUM OVERLAY

*Data source assumptions: Capitaline / Ace Equity / Bloomberg consensus (quarterly actual; monthly consensus revision)*

### 4.1 EPS Revision Velocity (ERV)

**Definition:** Annualized rate of change in 12m-forward consensus EPS.

```
ERV(i,t) = [ EPS_consensus_FY1(i,t) - EPS_consensus_FY1(i,t-63) ]
           / |EPS_consensus_FY1(i,t-63)|
           * (252/63)

where FY1 = next full fiscal year consensus
```

**Edge cases:**
- If EPS_consensus crosses zero (loss to profit or vice versa): use absolute change divided by revenue per share as denominator
- If consensus estimate unavailable (rare for ≥₹10K Cr stocks): use trailing EPS proxy with zero-confidence weight

**ERV scoring:**
```
ERV_score = f(ERV):
    > +20%      → +2.0
    +10% to 20% → +1.0
    +2% to 10%  → +0.5
    -2% to +2%  →  0.0   (flat; not directional)
    -10% to -2% → -0.5
    -20% to -10%→ -1.0
    < -20%      → -2.0
```

### 4.2 Revenue & Earnings Surprise Momentum

**Quarter-on-quarter beat/miss:**
```
Revenue_surprise(i, Qk) = (Revenue_actual(i,Qk) - Revenue_estimate_pre(i,Qk))
                         / |Revenue_estimate_pre(i,Qk)| * 100

where estimate_pre = consensus as of 5 trading days before result announcement
```

**Trailing weighted surprise score:**
```
RSS(i,t) = 0.50 * surprise(Q0) + 0.30 * surprise(Q-1) + 0.20 * surprise(Q-2)

EPS_surprise (same formula, for PAT or adjusted EPS)
ESP(i,t) = 0.50 * EPS_surprise(Q0) + 0.30 * EPS_surprise(Q-1) + 0.20 * EPS_surprise(Q-2)
```

**Thresholds:**
```
RSS_score = ESP_score = f(score):
    > +15%      → +2.0   (consistent strong beat cadence)
    +5% to +15% → +1.0
    -5% to +5%  →  0.0
    -15% to -5% → -1.0
    < -15%      → -2.0   (consistent miss; estimate credibility gap)
```

### 4.3 Institutional Ownership Trajectory

**Data:** NSE shareholding pattern filings (quarterly; lag ~21 days after quarter end)

```
FII_delta(i,t) = FII_pct(i,Q0) - FII_pct(i,Q-4)      [year-over-year change]
DII_delta(i,t) = DII_pct(i,Q0) - DII_pct(i,Q-4)
IO_delta(i,t)  = FII_delta + DII_delta
```

**Scoring:**
```
IO_score = f(IO_delta):
    > +5%        → +2.0
    +2% to +5%   → +1.0
    -2% to +2%   →  0.0
    -5% to -2%   → -1.0
    < -5%        → -2.0
```

**Recency weighting:** If latest shareholding data is >90 days old (delayed filer), apply 50% confidence weight.

### 4.4 Promoter Activity Score

```
Promoter signals (additive):

pledge_pct(i,t) current:
    < 10%  → +0.5
    10-30% →  0.0
    > 30%  → -1.0    [material risk flag]

promoter_delta (QoQ %):
    increasing pledges > 2%   → -0.5
    decreasing pledges > 2%   → +0.5
    stable                    →  0.0

corporate_actions trailing 12m:
    buyback at market price   → +0.5
    QIP at market (>95% mkt)  →  0.0
    QIP/PE at discount        → -0.3
    rights issue at >20% disc → -0.5
    ESOP overhang > 5% dilution → -0.3

Promoter_score = sum of above, clipped to [-2.0, +2.0]
```

### 4.5 Fundamental Score (FS) Composite

```
FS(i,t) = 0.35 * ERV_score
         + 0.25 * RSS_score
         + 0.25 * ESP_score
         + 0.10 * IO_score
         + 0.05 * Promoter_score

Range: [-2.0, +2.0]
```

**Z-score for composite:**
```
Z_FS(i,t) = ( FS(i,t) - mean(FS, universe) ) / std(FS, universe)
```
Winsorize at ±2.5 (fundamentals have fatter tails than price; tighter winsorizing).

---

## SECTION 5 — RISK-ADJUSTED COMPOSITE SCORING

### 5.1 Master Composite Score (MCS)

```
MCS(i,t) = 0.35 * Z_MTS(i,t)
          + 0.25 * Z_TREND(i,t)
          + 0.20 * Z_RS_final(i,t)
          + 0.20 * Z_FS(i,t)
```

**Weight rationale:**
- Z_MTS (35%): Price momentum primary driver; 1-3m predictive power established empirically
- Z_TREND (25%): Trend health prevents entry into decaying moves
- Z_RS (20%): Context — momentum must be relative, not absolute
- Z_FS (20%): Prevents value traps; prevents buying broken fundamentals with good recent price

### 5.2 Volatility Regime Detection (India VIX)

```
IVIX(t) = India VIX closing value (NSE published)
IVIX_30d_ma = 30-day moving average of IVIX

Regime assignments:
    IVIX < 13                          → REGIME_1 (Low volatility)
    13 <= IVIX < 20                    → REGIME_2 (Normal)
    20 <= IVIX < 28                    → REGIME_3 (Elevated)
    IVIX >= 28                         → REGIME_4 (High vol / crisis)

Regime confirmation: require 3 consecutive days in new regime before switching
(prevents daily regime flip)
```

### 5.3 Dynamic Bucket Boundaries

**5 Buckets:** EHM, HM, NT, NG, ENT

| Regime | EHM | HM | NT | NG | ENT |
|--------|-----|-----|-----|-----|-----|
| REGIME_1 (IVIX < 13) | MCS > 1.50 | 0.75 < MCS ≤ 1.50 | -0.50 < MCS ≤ 0.75 | -1.50 < MCS ≤ -0.50 | MCS ≤ -1.50 |
| REGIME_2 (13–20) | MCS > 1.25 | 0.50 < MCS ≤ 1.25 | -0.50 < MCS ≤ 0.50 | -1.25 < MCS ≤ -0.50 | MCS ≤ -1.25 |
| REGIME_3 (20–28) | MCS > 1.00 | 0.30 < MCS ≤ 1.00 | -0.30 < MCS ≤ 0.30 | -1.00 < MCS ≤ -0.30 | MCS ≤ -1.00 |
| REGIME_4 (IVIX ≥ 28) | MCS > 0.75 | 0.20 < MCS ≤ 0.75 | -0.20 < MCS ≤ 0.20 | -0.75 < MCS ≤ -0.20 | MCS ≤ -0.75 |

**Economic rationale:** In high-vol regimes, cross-sectional MCS dispersion widens — narrower boundaries ensure genuine outliers are captured without inflating EHM/ENT counts.

### 5.4 Override Rules (Hard Guards)

```
Override 1 — Drawdown cap on EHM:
    if bucket == EHM and DD(i,t) < -0.20:
        bucket = HM   [stock cannot be EHM if 20%+ below 52w high]

Override 2 — Fundamental floor:
    if Z_FS(i,t) < -1.75 and bucket in [EHM, HM]:
        bucket = NT   [severe fundamental deterioration caps price momentum upside]

Override 3 — Bucket persistence (anti-whipsaw):
    min_days_in_bucket = 10 trading days
    if days_in_current_bucket < 10:
        allow upgrade only if MCS exceeds current bucket upper bound by > 0.30
        allow downgrade only if MCS falls below current bucket lower bound by > 0.30

Override 4 — TUW hard cap on EHM:
    if TUW(i,t) >= 126:
        max_bucket = HM   [stock 6+ months below 52w high cannot be EHM]

Override 5 — Promoter pledge crisis:
    if pledge_pct > 50% and quarter-over-quarter pledge increasing:
        max_bucket = NT   [systemic forced-selling risk regardless of price]
```

### 5.5 Expected Bucket Distribution (Empirical Calibration Target)

Under REGIME_2 (normal vol), target distribution:
```
EHM:  ~10-12% of universe
HM:   ~18-20%
NT:   ~35-40%
NG:   ~18-20%
ENT:  ~10-12%
```

If actual distribution deviates >5% from targets: re-examine z-score computation or check for data quality issues in cross-sectional normalization.

---

## SECTION 6 — ALGORITHMIC IMPLEMENTATION SPECIFICATIONS

### 6.1 Data Frequency & Feed Requirements

```
Daily feed required:
├── OHLCV for all NSE equities (Bhav Copy, post-close ~6 PM IST)
├── India VIX closing value
├── Nifty 50 TRI (Total Return Index)
├── Sector indices (Nifty Auto, Bank, IT, Pharma, FMCG, Metal, Realty, Energy, Infra, etc.)
├── Market cap (shares outstanding * close; update daily)
├── Shareholding pattern (quarterly, pull on filing date)
└── Consensus EPS/Revenue estimates (monthly or on revision trigger)

Corporate action feed (event-driven):
├── Ex-dividend dates + dividend amount
├── Record dates for splits, bonuses, rights
├── Merger/amalgamation effective dates
└── Suspension/delisting notices from NSE
```

### 6.2 Computation Schedule

```
T = Friday close (end of trading week)

Step 1 [Friday 15:30–16:00 IST]: Fetch raw Bhav Copy
Step 2 [Friday 16:00–17:00]:     Apply corporate action adjustments
Step 3 [Friday 17:00–18:00]:     Compute all price-based signals (MTS, TREND, RS)
Step 4 [Friday 18:00–19:00]:     Fetch updated consensus estimates (if monthly update)
Step 5 [Friday 19:00–20:00]:     Compute FS, Z_FS
Step 6 [Friday 20:00–21:00]:     Compute MCS, apply overrides, assign buckets
Step 7 [Saturday morning]:       Generate change report: bucket upgrades/downgrades
Step 8 [Monday pre-market]:      Output active for trading decisions
```

**Intra-week reclassification trigger:**
```
if |MCS(i,t) - MCS(i,t_last_classification)| > 0.50:
    flag for mid-week review
    if override conditions also breached:
        immediate reclassification regardless of 10-day persistence rule
```

### 6.3 Corporate Action Handling

**Splits and bonuses:**
```
P_adj(i, τ < ex_date) = P_raw(i, τ) * adjustment_factor
adjustment_factor = post_split_shares / pre_split_shares (for split)
                  = 1 / (1 + bonus_ratio) (for bonus)

All historical prices and volumes retroactively adjusted.
OBV recalculated on adjusted price series.
```

**Dividends:**
```
Use total return index prices throughout — dividend auto-incorporated.
If only raw close available:
    P_adj(i, τ) = P_raw(i, τ) + PV_dividends_paid_since(τ, t)
```

**Rights issue:**
```
TERP (Theoretical Ex-Rights Price) adjustment applied on ex-date.
TERP = (P_pre * existing_shares + issue_price * new_shares) / (existing_shares + new_shares)
adjustment_factor = TERP / P_pre (applied to all pre-ex dates)
```

**Mergers / scheme of arrangement:**
```
Surviving entity: treated as continuation, adjustment applied on effective date.
Absorbed entity: removed from universe on effective date.
New combined entity: IPO treatment (63-day exclusion period from merger effective date).
```

### 6.4 IPO & Short-History Adjustments

```
def can_classify(stock, t):
    days_listed = (t - listing_date(stock)).trading_days
    if days_listed < 63:   return False, "Too new"
    if days_listed < 126:  return True, use_T1_T2_only
    if days_listed < 252:  return True, use_T1_T2_T3_only
    return True, full_computation

Confidence multiplier:
    63–126d:   0.70 (30% confidence haircut on MTS)
    126–252d:  0.85 (15% confidence haircut on MTS)
    ≥252d:     1.00
```

### 6.5 Suspension & Thin Trading Adjustments

```
Thin day definition:
    turnover(i,t) < ₹5 Crore   OR   volume(i,t) < 10,000 shares

thin_days_21d = count(thin days in last 21 trading days)
thin_days_63d = count(thin days in last 63 trading days)

if thin_days_21d > 5:
    skip OBV divergence computation (unreliable)
    use median volume of non-thin days for VER

if thin_days_63d > 15:
    apply confidence multiplier = 0.75 to Z_MTS

Suspension:
    if suspended_consecutive_days > 3:
        freeze bucket at last valid computation
    if suspended_consecutive_days > 21:
        exclude entirely; re-enter with 21-day post-resumption embargo
```

### 6.6 Circuit Breaker Handling

```
circuit_hit(i,t) = (close == upper_circuit) OR (close == lower_circuit)

if circuit_hit:
    use T-1 close for all return calculations on day T
    flag stock; do NOT update MCS on circuit day
    resume normal computation next non-circuit day
    if circuit_hit for 3+ consecutive days:
        freeze bucket classification; flag for manual review
```

### 6.7 Data Validation Pre-Scoring

```python
def validate_stock(ticker, t, prices, fundamentals):
    checks = {
        'min_history':        len(prices) >= 270,
        'market_cap':         market_cap(ticker, t) >= 10_000_crore,
        'not_suspended':      suspension_days_recent_21(ticker, t) <= 5,
        'price_continuity':   max_gap(prices, 63d) <= 10_trading_days,
        'no_pending_delist':  delist_notice(ticker) == False,
        'free_float':         free_float_pct(ticker) >= 15.0,
        'not_shell':          avg_daily_turnover_63d >= 1_crore,
    }
    confidence = 1.0
    thin_days = count_thin_days(ticker, 63)
    confidence -= (thin_days / 63) * 0.25

    failed = [k for k, v in checks.items() if not v]
    if any(c in failed for c in ['min_history', 'market_cap', 'not_suspended']):
        return False, 0.0, failed
    return True, confidence, failed
```

### 6.8 Exact Formula Reference (Machine-Readable Summary)

```
Z_MTS   = zscore(winsorise(Σwₙ * (CER(n)/σ₆₃), ±3))   w=[0.10,0.25,0.35,0.30]
Z_TREND = zscore(winsorise(0.45*Z_MASS + 0.35*VS + 0.20*DS, ±3))
Z_RS    = zscore(winsorise(CRS, ±3)) + peer_score + sector_bonus, clip ±3
Z_FS    = zscore(winsorise(0.35*ERV + 0.25*RSS + 0.25*ESP + 0.10*IO + 0.05*PROMO, ±2.5))

MCS     = 0.35*Z_MTS + 0.25*Z_TREND + 0.20*Z_RS + 0.20*Z_FS

Regime  = f(IVIX_3d_confirmed): {<13: R1, 13-20: R2, 20-28: R3, ≥28: R4}
Bucket  = f(MCS, Regime) → apply overrides → final classification
```

---

## APPENDIX: SECTOR CLASSIFICATION MAPPING (NSE)

| Code | Sector | NSE Index Reference |
|------|--------|-------------------|
| BFSI | Banking & Financial Services | Nifty Bank + Nifty Financial Services |
| IT | Information Technology | Nifty IT |
| PHARMA | Pharmaceuticals & Healthcare | Nifty Pharma |
| AUTO | Automobiles & Components | Nifty Auto |
| FMCG | Consumer Staples | Nifty FMCG |
| CONS | Consumer Discretionary | Nifty India Consumption |
| METAL | Metals & Mining | Nifty Metal |
| ENERGY | Oil, Gas & Power | Nifty Energy |
| INFRA | Infrastructure & Capital Goods | Nifty Infrastructure |
| REALTY | Real Estate | Nifty Realty |
| CHEM | Chemicals & Agrochemicals | Custom equal-weight |
| TELE | Telecom & Media | Custom equal-weight |
| MISC | Diversified Conglomerates | Custom equal-weight |

*Use AMFI/NSE official sector assignments as primary; custom mapping only for unclassified stocks.*

---

## KNOWN LIMITATIONS & MODEL RISKS

- **Look-ahead bias:** Consensus estimate revisions must use point-in-time data; survivor bias eliminated by including current delistings in historical backtests
- **Corporate governance:** Pledge data has 21-day filing lag; real-time deterioration undetectable
- **Thin market liquidity:** Mid-caps in universe may have 2-3 day impact cost for large AUM; MCS does not adjust for AUM-specific liquidity constraints
- **Regime model:** India VIX is options-derived; understates realized vol in illiquid midcap names; consider supplementing with realized vol cross-section in R3/R4
- **Fundamental data:** Capitaline/Ace consensus pools are thin (<5 analysts) for many midcaps; low analyst coverage = high ERV noise; apply coverage-count confidence weight: `FS_confidence = min(1.0, analyst_count/5)`
