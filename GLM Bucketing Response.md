Design a sophisticated, algorithmically implementable framework for classifying NSE-listed stocks with market capitalization above ₹10,000 crore into five distinct momentum buckets ranging from "Extremely High Momentum" to "Extremely Negative Trend." The classification system must reflect the expertise of a seasoned techno-fundamental analyst with 20+ years of experience managing large AUM/PMS portfolios. Specify exact quantitative criteria, multi-timeframe parameters, and composite scoring mechanisms that can be directly coded into trading algorithms without ambiguity.

Structure your response to cover:

**Multi-Timeframe Price Momentum Engine**
- Define specific lookback periods (e.g., 20-day, 63-day, 126-day, 252-day) with differentiated weightings
- Specify normalized return calculations (excess returns vs. Nifty 50, sectoral indices, and equal-weighted large-cap universe)
- Include volatility-adjusted momentum metrics (Sharpe-like ratios, not raw returns)

**Trend Structure & Market Microstructure Criteria**
- Detail precise moving average alignment rules (multiple MA hierarchy with slope confirmation, not simple price-vs-MA crossovers)
- Specify volume-profile confirmation requirements (volume-weighted price progression, institutional participation signatures)
- Include drawdown recovery patterns and time-underwater metrics with exact thresholds

**Relative Strength & Sector Rotation Dynamics**
- Define sector-relative momentum deciles with rebalancing frequency
- Specify industry group momentum rankings and peer percentile cutoffs
- Include regime detection for sector leadership transitions

**Fundamental Momentum Overlay**
- Quantify earnings revision velocity (rate of change in consensus EPS estimates, not just direction)
- Specify revenue/earnings surprise momentum with exact beat/miss thresholds
- Include institutional ownership trajectory and promoter activity patterns

**Risk-Adjusted Composite Scoring**
- Construct weighted multi-factor score (exact weights and component interactions)
- Define dynamic threshold boundaries for each of the five buckets (not static percentiles)
- Include volatility regime adjustments that shift boundaries based on market-wide VIX-equivalent levels

**Algorithmic Implementation Specifications**
- Provide exact formulas, parameter values, and rebalancing triggers
- Specify data frequency requirements and lookback window adjustments for different market conditions
- Detail edge cases: IPOs within classification window, corporate actions, suspension periods, and thin trading days

Avoid generic technical indicators (simple RSI thresholds, basic MACD crossovers, naive 200-DMA rules). Instead, emphasize institutional-grade methodologies: adaptive volatility targeting, regime-switching models, cross-asset confirmation signals, and behavioral anomaly detection that sophisticated quantitative systems employ for smallcap-midcap Indian equities. 

Thought Process
This framework is designed for institutional deployment within a PMS/AUM environment, targeting NSE-listed equities with market capitalization exceeding ₹10,000 crore. It synthesizes prudence-based fundamental momentum with rigorous, multi-timeframe quantitative trend structures, explicitly avoiding naive heuristics.

The system classifies the universe into five distinct buckets: Tier 1 (Extremely High Momentum), Tier 2 (High Momentum), Tier 3 (Neutral / Consolidating), Tier 4 (Negative Trend), and Tier 5 (Extremely Negative Trend).

1. Multi-Timeframe Price Momentum Engine
To neutralize market beta and isolate idiosyncratic alpha, raw returns are strictly prohibited. The engine uses a skip-1-month methodology (Jagadeesh-Titman lag) to mitigate short-term mean-reversion noise.

Lookback Parameters & Weightings:

T 
1
​
 
 = 63 trading days (Quarterly), Weight 
W 
1
​
 =20%
 * 
T 
2
​
 
 = 126 trading days (Half-Yearly), Weight 
W 
2
​
 =35%
 * 
T 
3
​
 
 = 252 trading days (Yearly), Weight 
W 
3
​
 =45%
 * Lag: 
t−21
 to 
t−1
 (skip recent 1 month)
Normalized Excess Return Calculation:
For each lookback 
T 
i
​
 
, calculate the excess return over three benchmarks: Nifty 50 (
N 
50
​
 
), respective Sectoral Index (
SI
), and an Equal-Weight Large-Cap Universe (
EWU
).
ER 
i
​
 =[∏ 
j=1
T 
i
​
 
​
 (1+R 
j,stock
​
 )−∏ 
j=1
T 
i
​
 
​
 (1+R 
j,bench
​
 )]× 
3
1
​
  (averaged across 3 benchmarks)

Volatility-Adjusted Momentum (M-Score):
Instead of raw returns, we use a Sharpe-like ratio of the excess returns.
M 
i
​
 = 
σ(ER 
i,daily
​
 )× 
252
​
 
ER 
i
​
 
​
 
 Aggregate Momentum Z-Score:
Z 
mom
​
 =∑(W 
i
​
 ×M 
i
​
 ) for i=1,2,3
2. Trend Structure & Market Microstructure Criteria
Trend confirmation requires a hierarchical moving average alignment with strict slope validation, filtering out "false breakouts" in low-liquidity setups.

Moving Average Hierarchy & Slope Confirmation:

MAs: 20-day EMA (
E 
20
​
 
), 50-day EMA (
E 
50
​
 
), 100-day EMA (
E 
100
​
 
), 200-day SMA (
S 
200
​
 
).
Tier 1 Alignment: 
Price>E 
20
​
 >E 
50
​
 >E 
100
​
 >S 
200
​
 
.
Slope Confirmation: The 20-day linear regression slope of 
E 
50
​
 
 must be 
>0
, AND the angle of ascent must be in the 80th percentile of its own 252-day slope history (prevents flat-base crossovers).
Volume-Profile Confirmation (Institutional Signatures):

VWAP Progression: The 20-day rolling VWAP must be 
>
 the 100-day rolling VWAP.
Accumulation/Up-Volume Ratio: On up-days, volume must exceed 10-day average volume by 1.2x; on down-days, volume must be 
<0.85×
 10-day average.
Volume-Weighted ATR (V-ATR): Calculated as 
ATR 
14
​
 × 
SMA(Volume,50)
Volume 
today
​
 
​
 
. Used to detect abnormal distribution.
Drawdown & Time-Underwater Metrics:

Drawdown Recovery: Stock must be within 10% of its 252-day High for Tier 1/2.
Time Underwater: Number of days since the stock made a 20-day closing high. For Tier 1, this must be 
<15
 days. A 252-day rolling drawdown must be less than 15% for Tier 1 categorization.
3. Relative Strength & Sector Rotation Dynamics
Sector leadership shifts are modeled via Mansfield Relative Strength (RS), adapted for dynamic Indian market regimes.

Mansfield RS Calculation:
RS 
raw
​
 = 
Price 
sector
​
 
Price 
stock
​
 
​
 
 
RS_{Mansfield} = \left( \frac{RS_{raw, today}}{SMA(RS_{raw}, 252})} - 1 \right) \times 100
 (Note: If 
RS 
Mansfield
​
 
 is rising but 
<0
, the stock is outperforming recently but still in a long-term relative downtrend—disqualified from Tier 1).

Peer Percentile & Sector Regime Filters:

Peer Decile: Calculate 
Z 
mom
​
 
 for all stocks in the sector. Stock must be in the top 20th percentile of its sector for Tier 1, top 40th for Tier 2.
Sector Regime Detection: Calculate the 63-day rate of change of the Sectoral Index. If Sector ROC 
<
 Nifty 50 ROC, the sector is in a "fading regime." Stocks in fading regimes cannot achieve Tier 1 status regardless of idiosyncratic strength; they are capped at Tier 2.
4. Fundamental Momentum Overlay
Fundamental data is factored via velocity rather than absolute valuation to capture earnings inflection points.

Earnings Revision Velocity (ERV):
Using rolling consensus EPS estimates (
EPS 
est
​
 
):
ERV= 
σ(ΔEPS 
est,90_day_window
​
 )
EPS 
est,today
​
 −EPS 
est,30_days_ago
​
 
​
 
 Threshold: ERV 
>1.5
 for Tier 1 (positive revisions accelerating beyond historical noise).

Surprise Momentum (SUE - Standardized Unexpected Earnings):
SUE= 
σ(Surprise 
last_8_quarters
​
 )
Actual_EPS−Consensus_EPS
​
 
 Threshold: SUE 
>2.0
 for Tier 1. Furthermore, the company must have beaten estimates in at least 2 of the last 3 quarters.

Institutional & Promoter Activity:

FII/DII Trajectory: QoQ change in institutional holding must be 
>+0.5%
 for Tier 1.
Promoter Activity: No pledge reduction or pledging increase. Creeping acquisition (promoter buying from open market) acts as a 
+0.5
 multiplier to the final composite score.
5. Risk-Adjusted Composite Scoring & Dynamic Thresholds
All components are normalized to Z-scores (
Z 
price
​
 
, 
Z 
trend
​
 
, 
Z 
relstr
​
 
, 
Z 
fund
​
 
) cross-sectionally across the eligible universe.

Composite Score (
CS
) Formula:
CS=(0.40×Z 
price
​
 )+(0.25×Z 
trend
​
 )+(0.15×Z 
relstr
​
 )+(0.20×Z 
fund
​
 )

Volatility Regime Adjustments (India VIX):
Static thresholds fail across regimes. We adjust the boundary thresholds based on the 252-day percentile rank of the India VIX.
Let 
VIX 
rank
​
 =Percentile(VIX 
today
​
 ,252)
.

High Volatility Regime (
VIX 
rank
​
 >0.80
): Momentum tends to mean-revert violently. Tighten thresholds to exclude all but the most extreme outperformers. Tier 1 requires 
CS>1.65
; Tier 5 requires 
CS<−1.65
.
Normal Regime (
0.20≤VIX 
rank
​
 ≤0.80
): Standard boundaries apply. Tier 1 requires 
CS>1.35
; Tier 5 requires 
CS<−1.35
.
**Low Volatility Regime (
VIX 
rank
​
 <0.20
):** Trends stretch further. Loosen thresholds. Tier 1 requires 
CS>1.10
; Tier 5 requires 
CS<−1.10
.
Five-Bucket Classification:

Extremely High Momentum: 
CS≥
 Dynamic Threshold, Trend Structure = Tier 1 Alignment, Sector not in fading regime.
High Momentum: 
CS≥
 (Dynamic Threshold 
×0.75
), Trend Structure = 
E 
20
​
 >E 
50
​
 >E 
100
​
 
.
Neutral / Consolidating: 
CS
 between positive and negative thresholds, OR Trend Structure is mixed (e.g., 
E 
20
​
 >E 
50
​
 
 but 
Price<S 
200
​
 
).
Negative Trend: 
CS≤
 (Dynamic Threshold 
×−0.75
), Trend Structure = 
Price<E 
20
​
 <E 
50
​
 
.
Extremely Negative Trend: 
CS≤
 (Dynamic Threshold 
×−1
), 
RS 
Mansfield
​
 
 declining for 
>63
 days, ERV 
<−1.0
.
6. Algorithmic Implementation Specifications
Data Frequency & Lookback Adjustments:

Price Data: End-of-Day (EOD) OHLCV. Intraday is unnecessary for this timeframe.
Fundamental Data: Polled daily; algorithms trigger on delta changes.
Rebalancing: Monthly execution on the 1st trading day (using 
t−1
 data).
Quantile Estimation: To avoid look-ahead bias, rolling quantiles for Z-scores must be computed using an expanding window ending at 
t−21
.
Edge Case Handling:

**IPOs < 252 Days:** 
Z 
mom
​
 
 and 
Z 
trend
​
 
 cannot be accurately computed. The framework implements a truncated lookback (uses available data 
>63
 days). If 
<63
 days, stock is forced to Tier 3 (Neutral) regardless of price action to prevent IPO euphoria skewing PMS risk.
Corporate Actions (Splits, Bonuses, Special Dividends): Prices MUST be adjusted using pure adjusted close algorithms (back-adjusted). Volume is inversely adjusted to preserve turnover value.
Suspensions / Trading Halts: If a stock is suspended for 
>5
 consecutive days, it is dropped from the universe until it trades for 21 continuous days. If suspension 
<5
 days, the 
Z
-score calculation rolls forward using 
t−x
 pricing.
Thin Trading Days (Circuit Filters): If a stock hits the upper/lower circuit for 
>3
 consecutive days, the 
ATR
 is artificially depressed. The algorithm substitutes the V-ATR with the median 14-day V-ATR prior to the circuit. If a stock is stuck in a Tier 5 lower circuit, the PMS algorithm must flag it for immediate manual liquidation review, as algorithmic exit is impossible.
Index Exclusions: If a stock drops below ₹10,000 Cr market cap due to a market crash, it remains in the universe for 63 days to allow for mean-reversion recovery before being purged (prevents selling at the absolute bottom).
Can you give me this response in a downloadable MD file? 

Thought Process
Here is the complete framework formatted as a Markdown file. Since I cannot directly send file attachments, you can easily save this as a .md file by copying the code block below, pasting it into a plain text editor (like Notepad, TextEdit, or VS Code), and saving it as NSE_Momentum_Framework.md.

Institutional-Grade Momentum Classification Framework for NSE Large/Mid-Cap Equities
This framework is designed for institutional deployment within a PMS/AUM environment, targeting NSE-listed equities with market capitalization exceeding ₹10,000 crore. It synthesizes prudence-based fundamental momentum with rigorous, multi-timeframe quantitative trend structures, explicitly avoiding naive heuristics.

The system classifies the universe into five distinct buckets: Tier 1 (Extremely High Momentum), Tier 2 (High Momentum), Tier 3 (Neutral / Consolidating), Tier 4 (Negative Trend), and Tier 5 (Extremely Negative Trend).

1. Multi-Timeframe Price Momentum Engine
To neutralize market beta and isolate idiosyncratic alpha, raw returns are strictly prohibited. The engine uses a skip-1-month methodology (Jagadeesh-Titman lag) to mitigate short-term mean-reversion noise.

Lookback Parameters & Weightings
T 
1
​
  = 63 trading days (Quarterly), Weight W 
1
​
 =20% * T 
2
​
  = 126 trading days (Half-Yearly), Weight W 
2
​
 =35% * T 
3
​
  = 252 trading days (Yearly), Weight W 
3
​
 =45% * Lag: t−21 to t−1 (skip recent 1 month)
Normalized Excess Return Calculation
For each lookback T 
i
​
 , calculate the excess return over three benchmarks: Nifty 50 (N 
50
​
 ), respective Sectoral Index (SI), and an Equal-Weight Large-Cap Universe (EWU).

ER 
i
​
 =[ 
j=1
∏
T 
i
​
 
​
 (1+R 
j,stock
​
 )− 
j=1
∏
T 
i
​
 
​
 (1+R 
j,bench
​
 )]× 
3
1
​
  (averaged across 3 benchmarks)

Volatility-Adjusted Momentum (M-Score)
Instead of raw returns, we use a Sharpe-like ratio of the excess returns.

M 
i
​
 = 
σ(ER 
i,daily
​
 )× 
252
​
 
ER 
i
​
 
​
 
Aggregate Momentum Z-Score:
Z 
mom
​
 =∑(W 
i
​
 ×M 
i
​
 ) for i=1,2,3
2. Trend Structure & Market Microstructure Criteria
Trend confirmation requires a hierarchical moving average alignment with strict slope validation, filtering out "false breakouts" in low-liquidity setups.

Moving Average Hierarchy & Slope Confirmation
MAs: 20-day EMA (E 
20
​
 ), 50-day EMA (E 
50
​
 ), 100-day EMA (E 
100
​
 ), 200-day SMA (S 
200
​
 ).
Tier 1 Alignment: Price>E 
20
​
 >E 
50
​
 >E 
100
​
 >S 
200
​
 .
Slope Confirmation: The 20-day linear regression slope of E 
50
​
  must be >0, AND the angle of ascent must be in the 80th percentile of its own 252-day slope history (prevents flat-base crossovers).
Volume-Profile Confirmation (Institutional Signatures)
VWAP Progression: The 20-day rolling VWAP must be > the 100-day rolling VWAP.
Accumulation/Up-Volume Ratio: On up-days, volume must exceed 10-day average volume by 1.2x; on down-days, volume must be <0.85× 10-day average.
Volume-Weighted ATR (V-ATR): Calculated as ATR 
14
​
 × 
SMA(Volume,50)
Volume 
today
​
 
​
 . Used to detect abnormal distribution.
Drawdown & Time-Underwater Metrics
Drawdown Recovery: Stock must be within 10% of its 252-day High for Tier 1/2.
Time Underwater: Number of days since the stock made a 20-day closing high. For Tier 1, this must be <15 days. A 252-day rolling drawdown must be less than 15% for Tier 1 categorization.
3. Relative Strength & Sector Rotation Dynamics
Sector leadership shifts are modeled via Mansfield Relative Strength (RS), adapted for dynamic Indian market regimes.

Mansfield RS Calculation
RS 
raw
​
 = 
Price 
sector
​
 
Price 
stock
​
 
​
 
RS_{Mansfield} = \left( \frac{RS_{raw, today}}{SMA(RS_{raw}, 252})} - 1 \right) \times 100 (Note: If RS 
Mansfield
​
  is rising but <0, the stock is outperforming recently but still in a long-term relative downtrend—disqualified from Tier 1).

Peer Percentile & Sector Regime Filters
Peer Decile: Calculate Z 
mom
​
  for all stocks in the sector. Stock must be in the top 20th percentile of its sector for Tier 1, top 40th for Tier 2.
Sector Regime Detection: Calculate the 63-day rate of change of the Sectoral Index. If Sector ROC < Nifty 50 ROC, the sector is in a "fading regime." Stocks in fading regimes cannot achieve Tier 1 status regardless of idiosyncratic strength; they are capped at Tier 2.
4. Fundamental Momentum Overlay
Fundamental data is factored via velocity rather than absolute valuation to capture earnings inflection points.

Earnings Revision Velocity (ERV)
Using rolling consensus EPS estimates (EPS 
est
​
 ):
ERV= 
σ(ΔEPS 
est,90_day_window
​
 )
EPS 
est,today
​
 −EPS 
est,30_days_ago
​
 
​
 
Threshold: ERV >1.5 for Tier 1 (positive revisions accelerating beyond historical noise).

Surprise Momentum (SUE - Standardized Unexpected Earnings)
SUE= 
σ(Surprise 
last_8_quarters
​
 )
Actual_EPS−Consensus_EPS
​
 
Threshold: SUE >2.0 for Tier 1. Furthermore, the company must have beaten estimates in at least 2 of the last 3 quarters.

Institutional & Promoter Activity
FII/DII Trajectory: QoQ change in institutional holding must be >+0.5% for Tier 1.
Promoter Activity: No pledge reduction or pledging increase. Creeping acquisition (promoter buying from open market) acts as a +0.5 multiplier to the final composite score.
5. Risk-Adjusted Composite Scoring & Dynamic Thresholds
All components are normalized to Z-scores (Z 
price
​
 , Z 
trend
​
 , Z 
relstr
​
 , Z 
fund
​
 ) cross-sectionally across the eligible universe.

Composite Score (CS) Formula
CS=(0.40×Z 
price
​
 )+(0.25×Z 
trend
​
 )+(0.15×Z 
relstr
​
 )+(0.20×Z 
fund
​
 )

Volatility Regime Adjustments (India VIX)
Static thresholds fail across regimes. We adjust the boundary thresholds based on the 252-day percentile rank of the India VIX.Let VIX 
rank
​
 =Percentile(VIX 
today
​
 ,252).

High Volatility Regime (VIX 
rank
​
 >0.80): Momentum tends to mean-revert violently. Tighten thresholds to exclude all but the most extreme outperformers. Tier 1 requires CS > 1.65$; Tier 5 requires $CS < -1.65.
Normal Regime (0.20≤VIX 
rank
​
 ≤0.80): Standard boundaries apply. Tier 1 requires CS > 1.35$; Tier 5 requires $CS < -1.35.
**Low Volatility Regime (VIX_{rank} < 0.20$):** Trends stretch further. Loosen thresholds. Tier 1 requires $CS > 1.10$; Tier 5 requires $CS < -1.10.
Five-Bucket Classification
Extremely High Momentum: CS≥ Dynamic Threshold, Trend Structure = Tier 1 Alignment, Sector not in fading regime.
High Momentum: CS≥ (Dynamic Threshold \times 0.75$), Trend Structure = $E_{20} > E_{50} > E_{100}.
Neutral / Consolidating: CS between positive and negative thresholds, OR Trend Structure is mixed (e.g., E 
20
​
 >E 
50
​
  but Price<S 
200
​
 ).
Negative Trend: CS≤ (Dynamic Threshold \times -0.75$), Trend Structure = $Price < E_{20} < E_{50}.
Extremely Negative Trend: CS≤ (Dynamic Threshold \times -1$), $RS_{Mansfield} declining for >63 days, ERV <−1.0.
6. Algorithmic Implementation Specifications
Data Frequency & Lookback Adjustments
Price Data: End-of-Day (EOD) OHLCV. Intraday is unnecessary for this timeframe.
Fundamental Data: Polled daily; algorithms trigger on delta changes.
Rebalancing: Monthly execution on the 1st trading day (using t−1 data).
Quantile Estimation: To avoid look-ahead bias, rolling quantiles for Z-scores must be computed using an expanding window ending at t−21.
Edge Case Handling
**IPOs < 252 Days:** Z 
mom
​
  and Z 
trend
​
  cannot be accurately computed. The framework implements a truncated lookback (uses available data >63 days). If <63 days, stock is forced to Tier 3 (Neutral) regardless of price action to prevent IPO euphoria skewing PMS risk.
Corporate Actions (Splits, Bonuses, Special Dividends): Prices MUST be adjusted using pure adjusted close algorithms (back-adjusted). Volume is inversely adjusted to preserve turnover value.
Suspensions / Trading Halts: If a stock is suspended for >5 consecutive days, it is dropped from the universe until it trades for 21 continuous days. If suspension <5 days, the Z$-score calculation rolls forward using $t-x pricing.
Thin Trading Days (Circuit Filters): If a stock hits the upper/lower circuit for >3 consecutive days, the ATR is artificially depressed. The algorithm substitutes the V-ATR with the median 14-day V-ATR prior to the circuit. If a stock is stuck in a Tier 5 lower circuit, the PMS algorithm must flag it for immediate manual liquidation review, as algorithmic exit is impossible.
Index Exclusions: If a stock drops below ₹10,000 Cr market cap due to a market crash, it remains in the universe for 63 days to allow for mean-reversion recovery before being purged (prevents selling at the absolute bottom).
Can you give me those two responses in a downloadable MD file? 

Thought Process
Here is the complete framework combining both the conversational explanation and the structured methodology, formatted within a single Markdown code block.

You can save this as a .md file by clicking the "Copy" button in the top right corner of the code block below, pasting it into a plain text editor (like Notepad, TextEdit, or VS Code), and saving it as NSE_Momentum_Framework.md.

Institutional-Grade Momentum Classification Framework for NSE Large/Mid-Cap Equities
This framework is designed for institutional deployment within a PMS/AUM environment, targeting NSE-listed equities with market capitalization exceeding ₹10,000 crore. It synthesizes prudence-based fundamental momentum with rigorous, multi-timeframe quantitative trend structures, explicitly avoiding naive heuristics.

The system classifies the universe into five distinct buckets: Tier 1 (Extremely High Momentum), Tier 2 (High Momentum), Tier 3 (Neutral / Consolidating), Tier 4 (Negative Trend), and Tier 5 (Extremely Negative Trend).

1. Multi-Timeframe Price Momentum Engine
To neutralize market beta and isolate idiosyncratic alpha, raw returns are strictly prohibited. The engine uses a skip-1-month methodology (Jagadeesh-Titman lag) to mitigate short-term mean-reversion noise.

Lookback Parameters & Weightings
T 
1
​
  = 63 trading days (Quarterly), Weight W 
1
​
 =20% * T 
2
​
  = 126 trading days (Half-Yearly), Weight W 
2
​
 =35% * T 
3
​
  = 252 trading days (Yearly), Weight W 
3
​
 =45% * Lag: t−21 to t−1 (skip recent 1 month)
Normalized Excess Return Calculation
For each lookback T 
i
​
 , calculate the excess return over three benchmarks: Nifty 50 (N 
50
​
 ), respective Sectoral Index (SI), and an Equal-Weight Large-Cap Universe (EWU).

ER 
i
​
 =[ 
j=1
∏
T 
i
​
 
​
 (1+R 
j,stock
​
 )− 
j=1
∏
T 
i
​
 
​
 (1+R 
j,bench
​
 )]× 
3
1
​
  (averaged across 3 benchmarks)

Volatility-Adjusted Momentum (M-Score)
Instead of raw returns, we use a Sharpe-like ratio of the excess returns.

M 
i
​
 = 
σ(ER 
i,daily
​
 )× 
252
​
 
ER 
i
​
 
​
 
Aggregate Momentum Z-Score:
Z 
mom
​
 =∑(W 
i
​
 ×M 
i
​
 ) for i=1,2,3
2. Trend Structure & Market Microstructure Criteria
Trend confirmation requires a hierarchical moving average alignment with strict slope validation, filtering out "false breakouts" in low-liquidity setups.

Moving Average Hierarchy & Slope Confirmation
MAs: 20-day EMA (E 
20
​
 ), 50-day EMA (E 
50
​
 ), 100-day EMA (E 
100
​
 ), 200-day SMA (S 
200
​
 ).
Tier 1 Alignment: Price>E 
20
​
 >E 
50
​
 >E 
100
​
 >S 
200
​
 .
Slope Confirmation: The 20-day linear regression slope of E 
50
​
  must be >0, AND the angle of ascent must be in the 80th percentile of its own 252-day slope history (prevents flat-base crossovers).
Volume-Profile Confirmation (Institutional Signatures)
VWAP Progression: The 20-day rolling VWAP must be > the 100-day rolling VWAP.
Accumulation/Up-Volume Ratio: On up-days, volume must exceed 10-day average volume by 1.2x; on down-days, volume must be <0.85× 10-day average.
Volume-Weighted ATR (V-ATR): Calculated as ATR 
14
​
 × 
SMA(Volume,50)
Volume 
today
​
 
​
 . Used to detect abnormal distribution.
Drawdown & Time-Underwater Metrics
Drawdown Recovery: Stock must be within 10% of its 252-day High for Tier 1/2.
Time Underwater: Number of days since the stock made a 20-day closing high. For Tier 1, this must be <15 days. A 252-day rolling drawdown must be less than 15% for Tier 1 categorization.
3. Relative Strength & Sector Rotation Dynamics
Sector leadership shifts are modeled via Mansfield Relative Strength (RS), adapted for dynamic Indian market regimes.

Mansfield RS Calculation
RS 
raw
​
 = 
Price 
sector
​
 
Price 
stock
​
 
​
 
RS 
Mansfield
​
 =( 
SMA(RS 
raw,252
​
 )
RS 
raw,today
​
 
​
 −1)×100
(Note: If RS 
Mansfield
​
  is rising but <0, the stock is outperforming recently but still in a long-term relative downtrend—disqualified from Tier 1).

Peer Percentile & Sector Regime Filters
Peer Decile: Calculate Z 
mom
​
  for all stocks in the sector. Stock must be in the top 20th percentile of its sector for Tier 1, top 40th for Tier 2.
Sector Regime Detection: Calculate the 63-day rate of change of the Sectoral Index. If Sector ROC < Nifty 50 ROC, the sector is in a "fading regime." Stocks in fading regimes cannot achieve Tier 1 status regardless of idiosyncratic strength; they are capped at Tier 2.
4. Fundamental Momentum Overlay
Fundamental data is factored via velocity rather than absolute valuation to capture earnings inflection points.

Earnings Revision Velocity (ERV)
Using rolling consensus EPS estimates (EPS 
est
​
 ):
ERV= 
σ(ΔEPS 
est,90_day_window
​
 )
EPS 
est,today
​
 −EPS 
est,30_days_ago
​
 
​
 
Threshold: ERV >1.5 for Tier 1 (positive revisions accelerating beyond historical noise).

Surprise Momentum (SUE - Standardized Unexpected Earnings)
SUE= 
σ(Surprise 
last_8_quarters
​
 )
Actual_EPS−Consensus_EPS
​
 
Threshold: SUE >2.0 for Tier 1. Furthermore, the company must have beaten estimates in at least 2 of the last 3 quarters.

Institutional & Promoter Activity
FII/DII Trajectory: QoQ change in institutional holding must be >+0.5% for Tier 1.
Promoter Activity: No pledge reduction or pledging increase. Creeping acquisition (promoter buying from open market) acts as a +0.5 multiplier to the final composite score.
5. Risk-Adjusted Composite Scoring & Dynamic Thresholds
All components are normalized to Z-scores (Z 
price
​
 , Z 
trend
​
 , Z 
relstr
​
 , Z 
fund
​
 ) cross-sectionally across the eligible universe.

Composite Score (CS) Formula
CS=(0.40×Z 
price
​
 )+(0.25×Z 
trend
​
 )+(0.15×Z 
relstr
​
 )+(0.20×Z 
fund
​
 )

Volatility Regime Adjustments (India VIX)
Static thresholds fail across regimes. We adjust the boundary thresholds based on the 252-day percentile rank of the India VIX.Let VIX 
rank
​
 =Percentile(VIX 
today
​
 ,252).

High Volatility Regime (VIX 
rank
​
 >0.80): Momentum tends to mean-revert violently. Tighten thresholds to exclude all but the most extreme outperformers. Tier 1 requires CS > 1.65$; Tier 5 requires $CS < -1.65.
Normal Regime (0.20≤VIX 
rank
​
 ≤0.80): Standard boundaries apply. Tier 1 requires CS > 1.35$; Tier 5 requires $CS < -1.35.
**Low Volatility Regime (VIX_{rank} < 0.20$):** Trends stretch further. Loosen thresholds. Tier 1 requires $CS > 1.10$; Tier 5 requires $CS < -1.10.
Five-Bucket Classification
Extremely High Momentum: CS≥ Dynamic Threshold, Trend Structure = Tier 1 Alignment, Sector not in fading regime.
High Momentum: CS≥ (Dynamic Threshold \times 0.75$), Trend Structure = $E_{20} > E_{50} > E_{100}.
Neutral / Consolidating: CS between positive and negative thresholds, OR Trend Structure is mixed (e.g., E 
20
​
 >E 
50
​
  but Price<S 
200
​
 ).
Negative Trend: CS≤ (Dynamic Threshold \times -0.75$), Trend Structure = $Price < E_{20} < E_{50}.
Extremely Negative Trend: CS≤ (Dynamic Threshold \times -1$), $RS_{Mansfield} declining for >63 days, ERV <−1.0.
6. Algorithmic Implementation Specifications
Data Frequency & Lookback Adjustments
Price Data: End-of-Day (EOD) OHLCV. Intraday is unnecessary for this timeframe.
Fundamental Data: Polled daily; algorithms trigger on delta changes.
Rebalancing: Monthly execution on the 1st trading day (using t−1 data).
Quantile Estimation: To avoid look-ahead bias, rolling quantiles for Z-scores must be computed using an expanding window ending at t−21.
Edge Case Handling
**IPOs < 252 Days:** Z 
mom
​
  and Z 
trend
​
  cannot be accurately computed. The framework implements a truncated lookback (uses available data >63 days). If <63 days, stock is forced to Tier 3 (Neutral) regardless of price action to prevent IPO euphoria skewing PMS risk.
Corporate Actions (Splits, Bonuses, Special Dividends): Prices MUST be adjusted using pure adjusted close algorithms (back-adjusted). Volume is inversely adjusted to preserve turnover value.
Suspensions / Trading Halts: If a stock is suspended for >5 consecutive days, it is dropped from the universe until it trades for 21 continuous days. If suspension <5 days, the Z$-score calculation rolls forward using $t-x pricing.
Thin Trading Days (Circuit Filters): If a stock hits the upper/lower circuit for >3 consecutive days, the ATR is artificially depressed. The algorithm substitutes the V-ATR with the median 14-day V-ATR prior to the circuit. If a stock is stuck in a Tier 5 lower circuit, the PMS algorithm must flag it for immediate manual liquidation review, as algorithmic exit is impossible.
Index Exclusions: If a stock drops below ₹10,000 Cr market cap due to a market crash, it remains in the universe for 63 days to allow for mean-reversion recovery before being purged (prevents selling at the absolute bottom).
I need to save these responses in an MD File? Please share an MD file recording these responses. 

Thought Process
As an AI, I cannot directly generate and download a file to your computer. However, I have packaged the complete, combined framework into a single, clean Markdown code block below.

To save this as an MD file:

Click the "Copy" button in the top right corner of the code block below.
Open a plain text editor (like Notepad on Windows, TextEdit on Mac, or VS Code).
Paste the copied text.
Save the file as NSE_Momentum_Framework.md.
Institutional-Grade Momentum Classification Framework for NSE Equities
This framework is designed for institutional deployment within a PMS/AUM environment, targeting NSE-listed equities with market capitalization exceeding ₹10,000 crore. It synthesizes prudence-based fundamental momentum with rigorous, multi-timeframe quantitative trend structures, explicitly avoiding naive heuristics.

The system classifies the universe into five distinct buckets: Tier 1 (Extremely High Momentum), Tier 2 (High Momentum), Tier 3 (Neutral / Consolidating), Tier 4 (Negative Trend), and Tier 5 (Extremely Negative Trend).

1. Multi-Timeframe Price Momentum Engine
To neutralize market beta and isolate idiosyncratic alpha, raw returns are strictly prohibited. The engine uses a skip-1-month methodology (Jagadeesh-Titman lag) to mitigate short-term mean-reversion noise.

Lookback Parameters & Weightings
T 
1
​
  = 63 trading days (Quarterly), Weight W 
1
​
 =20% * T 
2
​
  = 126 trading days (Half-Yearly), Weight W 
2
​
 =35% * T 
3
​
  = 252 trading days (Yearly), Weight W 
3
​
 =45% * Lag: t−21 to t−1 (skip recent 1 month)
Normalized Excess Return Calculation
For each lookback T 
i
​
 , calculate the excess return over three benchmarks: Nifty 50 (N 
50
​
 ), respective Sectoral Index (SI), and an Equal-Weight Large-Cap Universe (EWU).

ER 
i
​
 =[ 
j=1
∏
T 
i
​
 
​
 (1+R 
j,stock
​
 )− 
j=1
∏
T 
i
​
 
​
 (1+R 
j,bench
​
 )]× 
3
1
​
  (averaged across 3 benchmarks)

Volatility-Adjusted Momentum (M-Score)
Instead of raw returns, we use a Sharpe-like ratio of the excess returns.

M 
i
​
 = 
σ(ER 
i,daily
​
 )× 
252
​
 
ER 
i
​
 
​
 
Aggregate Momentum Z-Score:
Z 
mom
​
 =∑(W 
i
​
 ×M 
i
​
 ) for i=1,2,3
2. Trend Structure & Market Microstructure Criteria
Trend confirmation requires a hierarchical moving average alignment with strict slope validation, filtering out "false breakouts" in low-liquidity setups.

Moving Average Hierarchy & Slope Confirmation
MAs: 20-day EMA (E 
20
​
 ), 50-day EMA (E 
50
​
 ), 100-day EMA (E 
100
​
 ), 200-day SMA (S 
200
​
 ).
Tier 1 Alignment: Price>E 
20
​
 >E 
50
​
 >E 
100
​
 >S 
200
​
 .
Slope Confirmation: The 20-day linear regression slope of E 
50
​
  must be >0, AND the angle of ascent must be in the 80th percentile of its own 252-day slope history (prevents flat-base crossovers).
Volume-Profile Confirmation (Institutional Signatures)
VWAP Progression: The 20-day rolling VWAP must be > the 100-day rolling VWAP.
Accumulation/Up-Volume Ratio: On up-days, volume must exceed 10-day average volume by 1.2x; on down-days, volume must be <0.85× 10-day average.
Volume-Weighted ATR (V-ATR): Calculated as ATR 
14
​
 × 
SMA(Volume,50)
Volume 
today
​
 
​
 . Used to detect abnormal distribution.
Drawdown & Time-Underwater Metrics
Drawdown Recovery: Stock must be within 10% of its 252-day High for Tier 1/2.
Time Underwater: Number of days since the stock made a 20-day closing high. For Tier 1, this must be <15 days. A 252-day rolling drawdown must be less than 15% for Tier 1 categorization.
3. Relative Strength & Sector Rotation Dynamics
Sector leadership shifts are modeled via Mansfield Relative Strength (RS), adapted for dynamic Indian market regimes.

Mansfield RS Calculation
RS 
raw
​
 = 
Price 
sector
​
 
Price 
stock
​
 
​
 
RS 
Mansfield
​
 =( 
SMA(RS 
raw,252
​
 )
RS 
raw,today
​
 
​
 −1)×100
(Note: If RS 
Mansfield
​
  is rising but <0, the stock is outperforming recently but still in a long-term relative downtrend—disqualified from Tier 1).

Peer Percentile & Sector Regime Filters
Peer Decile: Calculate Z 
mom
​
  for all stocks in the sector. Stock must be in the top 20th percentile of its sector for Tier 1, top 40th for Tier 2.
Sector Regime Detection: Calculate the 63-day rate of change of the Sectoral Index. If Sector ROC < Nifty 50 ROC, the sector is in a "fading regime." Stocks in fading regimes cannot achieve Tier 1 status regardless of idiosyncratic strength; they are capped at Tier 2.
4. Fundamental Momentum Overlay
Fundamental data is factored via velocity rather than absolute valuation to capture earnings inflection points.

Earnings Revision Velocity (ERV)
Using rolling consensus EPS estimates (EPS 
est
​
 ):
ERV= 
σ(ΔEPS 
est,90_day_window
​
 )
EPS 
est,today
​
 −EPS 
est,30_days_ago
​
 
​
 
Threshold: ERV >1.5 for Tier 1 (positive revisions accelerating beyond historical noise).

Surprise Momentum (SUE - Standardized Unexpected Earnings)
SUE= 
σ(Surprise 
last_8_quarters
​
 )
Actual_EPS−Consensus_EPS
​
 
Threshold: SUE >2.0 for Tier 1. Furthermore, the company must have beaten estimates in at least 2 of the last 3 quarters.

Institutional & Promoter Activity
FII/DII Trajectory: QoQ change in institutional holding must be >+0.5% for Tier 1.
Promoter Activity: No pledge reduction or pledging increase. Creeping acquisition (promoter buying from open market) acts as a +0.5 multiplier to the final composite score.
5. Risk-Adjusted Composite Scoring & Dynamic Thresholds
All components are normalized to Z-scores (Z 
price
​
 , Z 
trend
​
 , Z 
relstr
​
 , Z 
fund
​
 ) cross-sectionally across the eligible universe.

Composite Score (CS) Formula
CS=(0.40×Z 
price
​
 )+(0.25×Z 
trend
​
 )+(0.15×Z 
relstr
​
 )+(0.20×Z 
fund
​
 )

Volatility Regime Adjustments (India VIX)
Static thresholds fail across regimes. We adjust the boundary thresholds based on the 252-day percentile rank of the India VIX.Let VIX 
rank
​
 =Percentile(VIX 
today
​
 ,252).

High Volatility Regime (VIX 
rank
​
 >0.80): Momentum tends to mean-revert violently. Tighten thresholds to exclude all but the most extreme outperformers. Tier 1 requires CS > 1.65$; Tier 5 requires $CS < -1.65.
Normal Regime (0.20≤VIX 
rank
​
 ≤0.80): Standard boundaries apply. Tier 1 requires CS > 1.35$; Tier 5 requires $CS < -1.35.
**Low Volatility Regime (VIX_{rank} < 0.20$):** Trends stretch further. Loosen thresholds. Tier 1 requires $CS > 1.10$; Tier 5 requires $CS < -1.10.
Five-Bucket Classification
Extremely High Momentum: CS≥ Dynamic Threshold, Trend Structure = Tier 1 Alignment, Sector not in fading regime.
High Momentum: CS≥ (Dynamic Threshold \times 0.75$), Trend Structure = $E_{20} > E_{50} > E_{100}.
Neutral / Consolidating: CS between positive and negative thresholds, OR Trend Structure is mixed (e.g., E 
20
​
 >E 
50
​
  but Price<S 
200
​
 ).
Negative Trend: CS≤ (Dynamic Threshold \times -0.75$), Trend Structure = $Price < E_{20} < E_{50}.
Extremely Negative Trend: CS≤ (Dynamic Threshold \times -1$), $RS_{Mansfield} declining for >63 days, ERV <−1.0.
6. Algorithmic Implementation Specifications
Data Frequency & Lookback Adjustments
Price Data: End-of-Day (EOD) OHLCV. Intraday is unnecessary for this timeframe.
Fundamental Data: Polled daily; algorithms trigger on delta changes.
Rebalancing: Monthly execution on the 1st trading day (using t−1 data).
Quantile Estimation: To avoid look-ahead bias, rolling quantiles for Z-scores must be computed using an expanding window ending at t−21.
Edge Case Handling
**IPOs < 252 Days:** Z 
mom
​
  and Z 
trend
​
  cannot be accurately computed. The framework implements a truncated lookback (uses available data >63 days). If <63 days, stock is forced to Tier 3 (Neutral) regardless of price action to prevent IPO euphoria skewing PMS risk.
Corporate Actions (Splits, Bonuses, Special Dividends): Prices MUST be adjusted using pure adjusted close algorithms (back-adjusted). Volume is inversely adjusted to preserve turnover value.
Suspensions / Trading Halts: If a stock is suspended for >5 consecutive days, it is dropped from the universe until it trades for 21 continuous days. If suspension <5 days, the Z$-score calculation rolls forward using $t-x pricing.
Thin Trading Days (Circuit Filters): If a stock hits the upper/lower circuit for >3 consecutive days, the ATR is artificially depressed. The algorithm substitutes the V-ATR with the median 14-day V-ATR prior to the circuit. If a stock is stuck in a Tier 5 lower circuit, the PMS algorithm must flag it for immediate manual liquidation review, as algorithmic exit is impossible.
Index Exclusions: If a stock drops below ₹10,000 Cr market cap due to a market crash, it remains in the universe for 63 days to allow for mean-reversion recovery before being purged (prevents selling at the absolute bottom).
Can you share NSE_Momentum_Framework.md in the downloads folder? 

Thought Process
I apologize, but as an AI, I do not have direct access to your computer's local file system. Therefore, I cannot automatically save or download a file directly into your Downloads folder.

However, you can easily create the file yourself in just a few seconds. Here is how to do it:

Copy all the text inside the black code block below.
Open a plain text editor on your computer (like Notepad on Windows or TextEdit on Mac). Note for Mac users: ensure TextEdit is set to "Make Plain Text" under the Format menu.
Paste the copied text into the editor.
Go to File > Save As.
Name the file exactly NSE_Momentum_Framework.md
Important: In the "Save as type" dropdown (if on Windows), select "All Files" so it doesn't accidentally save as NSE_Momentum_Framework.md.txt.
Choose your Downloads folder as the save location and click Save.
Here is the content to copy:

Institutional-Grade Momentum Classification Framework for NSE Equities
This framework is designed for institutional deployment within a PMS/AUM environment, targeting NSE-listed equities with market capitalization exceeding ₹10,000 crore. It synthesizes prudence-based fundamental momentum with rigorous, multi-timeframe quantitative trend structures, explicitly avoiding naive heuristics.

The system classifies the universe into five distinct buckets: Tier 1 (Extremely High Momentum), Tier 2 (High Momentum), Tier 3 (Neutral / Consolidating), Tier 4 (Negative Trend), and Tier 5 (Extremely Negative Trend).

1. Multi-Timeframe Price Momentum Engine
To neutralize market beta and isolate idiosyncratic alpha, raw returns are strictly prohibited. The engine uses a skip-1-month methodology (Jagadeesh-Titman lag) to mitigate short-term mean-reversion noise.

Lookback Parameters & Weightings
T 
1
​
  = 63 trading days (Quarterly), Weight W 
1
​
 =20% * T 
2
​
  = 126 trading days (Half-Yearly), Weight W 
2
​
 =35% * T 
3
​
  = 252 trading days (Yearly), Weight W 
3
​
 =45% * Lag: t−21 to t−1 (skip recent 1 month)
Normalized Excess Return Calculation
For each lookback T 
i
​
 , calculate the excess return over three benchmarks: Nifty 50 (N 
50
​
 ), respective Sectoral Index (SI), and an Equal-Weight Large-Cap Universe (EWU).

ER 
i
​
 =[ 
j=1
∏
T 
i
​
 
​
 (1+R 
j,stock
​
 )− 
j=1
∏
T 
i
​
 
​
 (1+R 
j,bench
​
 )]× 
3
1
​
  (averaged across 3 benchmarks)

Volatility-Adjusted Momentum (M-Score)
Instead of raw returns, we use a Sharpe-like ratio of the excess returns.

M 
i
​
 = 
σ(ER 
i,daily
​
 )× 
252
​
 
ER 
i
​
 
​
 
Aggregate Momentum Z-Score:
Z 
mom
​
 =∑(W 
i
​
 ×M 
i
​
 ) for i=1,2,3
2. Trend Structure & Market Microstructure Criteria
Trend confirmation requires a hierarchical moving average alignment with strict slope validation, filtering out "false breakouts" in low-liquidity setups.

Moving Average Hierarchy & Slope Confirmation
MAs: 20-day EMA (E 
20
​
 ), 50-day EMA (E 
50
​
 ), 100-day EMA (E 
100
​
 ), 200-day SMA (S 
200
​
 ).
Tier 1 Alignment: Price>E 
20
​
 >E 
50
​
 >E 
100
​
 >S 
200
​
 .
Slope Confirmation: The 20-day linear regression slope of E 
50
​
  must be >0, AND the angle of ascent must be in the 80th percentile of its own 252-day slope history (prevents flat-base crossovers).
Volume-Profile Confirmation (Institutional Signatures)
VWAP Progression: The 20-day rolling VWAP must be > the 100-day rolling VWAP.
Accumulation/Up-Volume Ratio: On up-days, volume must exceed 10-day average volume by 1.2x; on down-days, volume must be <0.85× 10-day average.
Volume-Weighted ATR (V-ATR): Calculated as ATR 
14
​
 × 
SMA(Volume,50)
Volume 
today
​
 
​
 . Used to detect abnormal distribution.
Drawdown & Time-Underwater Metrics
Drawdown Recovery: Stock must be within 10% of its 252-day High for Tier 1/2.
Time Underwater: Number of days since the stock made a 20-day closing high. For Tier 1, this must be <15 days. A 252-day rolling drawdown must be less than 15% for Tier 1 categorization.
3. Relative Strength & Sector Rotation Dynamics
Sector leadership shifts are modeled via Mansfield Relative Strength (RS), adapted for dynamic Indian market regimes.

Mansfield RS Calculation
RS 
raw
​
 = 
Price 
sector
​
 
Price 
stock
​
 
​
 
RS 
Mansfield
​
 =( 
SMA(RS 
raw,252
​
 )
RS 
raw,today
​
 
​
 −1)×100
(Note: If RS 
Mansfield
​
  is rising but <0, the stock is outperforming recently but still in a long-term relative downtrend—disqualified from Tier 1).

Peer Percentile & Sector Regime Filters
Peer Decile: Calculate Z 
mom
​
  for all stocks in the sector. Stock must be in the top 20th percentile of its sector for Tier 1, top 40th for Tier 2.
Sector Regime Detection: Calculate the 63-day rate of change of the Sectoral Index. If Sector ROC < Nifty 50 ROC, the sector is in a "fading regime." Stocks in fading regimes cannot achieve Tier 1 status regardless of idiosyncratic strength; they are capped at Tier 2.
4. Fundamental Momentum Overlay
Fundamental data is factored via velocity rather than absolute valuation to capture earnings inflection points.

Earnings Revision Velocity (ERV)
Using rolling consensus EPS estimates (EPS 
est
​
 ):
ERV= 
σ(ΔEPS 
est,90_day_window
​
 )
EPS 
est,today
​
 −EPS 
est,30_days_ago
​
 
​
 
Threshold: ERV >1.5 for Tier 1 (positive revisions accelerating beyond historical noise).

Surprise Momentum (SUE - Standardized Unexpected Earnings)
SUE= 
σ(Surprise 
last_8_quarters
​
 )
Actual_EPS−Consensus_EPS
​
 
Threshold: SUE >2.0 for Tier 1. Furthermore, the company must have beaten estimates in at least 2 of the last 3 quarters.

Institutional & Promoter Activity
FII/DII Trajectory: QoQ change in institutional holding must be >+0.5% for Tier 1.
Promoter Activity: No pledge reduction or pledging increase. Creeping acquisition (promoter buying from open market) acts as a +0.5 multiplier to the final composite score.
5. Risk-Adjusted Composite Scoring & Dynamic Thresholds
All components are normalized to Z-scores (Z 
price
​
 , Z 
trend
​
 , Z 
relstr
​
 , Z 
fund
​
 ) cross-sectionally across the eligible universe.

Composite Score (CS) Formula
CS=(0.40×Z 
price
​
 )+(0.25×Z 
trend
​
 )+(0.15×Z 
relstr
​
 )+(0.20×Z 
fund
​
 )

Volatility Regime Adjustments (India VIX)
Static thresholds fail across regimes. We adjust the boundary thresholds based on the 252-day percentile rank of the India VIX.Let VIX 
rank
​
 =Percentile(VIX 
today
​
 ,252).

High Volatility Regime (VIX 
rank
​
 >0.80): Momentum tends to mean-revert violently. Tighten thresholds to exclude all but the most extreme outperformers. Tier 1 requires CS > 1.65$; Tier 5 requires $CS < -1.65.
Normal Regime (0.20≤VIX 
rank
​
 ≤0.80): Standard boundaries apply. Tier 1 requires CS > 1.35$; Tier 5 requires $CS < -1.35.
**Low Volatility Regime (VIX_{rank} < 0.20$):** Trends stretch further. Loosen thresholds. Tier 1 requires $CS > 1.10$; Tier 5 requires $CS < -1.10.
Five-Bucket Classification
Extremely High Momentum: CS≥ Dynamic Threshold, Trend Structure = Tier 1 Alignment, Sector not in fading regime.
High Momentum: CS≥ (Dynamic Threshold \times 0.75$), Trend Structure = $E_{20} > E_{50} > E_{100}.
Neutral / Consolidating: CS between positive and negative thresholds, OR Trend Structure is mixed (e.g., E 
20
​
 >E 
50
​
  but Price<S 
200
​
 ).
Negative Trend: CS≤ (Dynamic Threshold \times -0.75$), Trend Structure = $Price < E_{20} < E_{50}.
Extremely Negative Trend: CS≤ (Dynamic Threshold \times -1$), $RS_{Mansfield} declining for >63 days, ERV <−1.0.
6. Algorithmic Implementation Specifications
Data Frequency & Lookback Adjustments
Price Data: End-of-Day (EOD) OHLCV. Intraday is unnecessary for this timeframe.
Fundamental Data: Polled daily; algorithms trigger on delta changes.
Rebalancing: Monthly execution on the 1st trading day (using t−1 data).
Quantile Estimation: To avoid look-ahead bias, rolling quantiles for Z-scores must be computed using an expanding window ending at t−21.
Edge Case Handling
**IPOs < 252 Days:** Z 
mom
​
  and Z 
trend
​
  cannot be accurately computed. The framework implements a truncated lookback (uses available data >63 days). If <63 days, stock is forced to Tier 3 (Neutral) regardless of price action to prevent IPO euphoria skewing PMS risk.
Corporate Actions (Splits, Bonuses, Special Dividends): Prices MUST be adjusted using pure adjusted close algorithms (back-adjusted). Volume is inversely adjusted to preserve turnover value.
Suspensions / Trading Halts: If a stock is suspended for >5 consecutive days, it is dropped from the universe until it trades for 21 continuous days. If suspension <5 days, the Z$-score calculation rolls forward using $t-x pricing.
Thin Trading Days (Circuit Filters): If a stock hits the upper/lower circuit for >3 consecutive days, the ATR is artificially depressed. The algorithm substitutes the V-ATR with the median 14-day V-ATR prior to the circuit. If a stock is stuck in a Tier 5 lower circuit, the PMS algorithm must flag it for immediate manual liquidation review, as algorithmic exit is impossible.
Index Exclusions: If a stock drops below ₹10,000 Cr market cap due to a market crash, it remains in the universe for 63 days to allow for mean-reversion recovery before being purged (prevents selling at the absolute bottom).


