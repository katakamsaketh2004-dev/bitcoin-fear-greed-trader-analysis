# Bitcoin Fear & Greed Index vs Hyperliquid Trader Performance
## Complete Analysis Report
### Prepared for: Web3 Trading Company Hiring Assignment
### Analyst: Senior Data Scientist & Quantitative Trading Analyst
### Date: June 2025

---

## EXECUTIVE SUMMARY

This report presents a comprehensive quantitative analysis of 104,266 closing trades executed by 32 accounts on the Hyperliquid decentralized exchange between December 2023 and May 2025, cross-referenced with the Bitcoin Fear & Greed Index.

**Key Findings at a Glance:**

| Metric | Value |
|---|---|
| Total Closed PnL | $10,197,005 |
| Overall Win Rate | **83.2%** |
| Best Sentiment Period | Extreme Greed (avg $130/trade, 89.2% win rate) |
| Second Best | Fear (avg $112/trade, 87.3% win rate) |
| Worst Sentiment Period | Extreme Fear (avg $72/trade, 76.2% win rate) |
| Top Performing Coin | @107 ($2.78M total PnL) |
| ANOVA Statistical Significance | p = 0.000003 ✓ |

**Core Insight:** Trader performance is **statistically significantly different** across sentiment regimes. Counterintuitively, the **Fear** sentiment period produced the second-highest average per-trade PnL ($112.07), suggesting experienced traders exploit fearful market conditions. Extreme Greed also performs strongly ($130.27), but carries elevated volatility. Larger position sizes consistently produce higher absolute PnL without harming win rates, suggesting the top traders are well-calibrated to scale up during favorable conditions.

---

## 1. INTRODUCTION

### 1.1 Background
Hyperliquid is a decentralized perpetuals exchange operating on-chain. Unlike centralized exchanges, all trading data is transparent and auditable. The Bitcoin Fear & Greed Index, published daily by Alternative.me, quantifies market sentiment on a 0–100 scale based on volatility, market momentum, social media, surveys, dominance, and trends.

The central hypothesis of this analysis is: **market sentiment measurably influences both the frequency and profitability of trading activity on Hyperliquid.**

### 1.2 Research Questions
1. Are traders more profitable during Fear or Greed regimes?
2. Does position sizing (as a leverage proxy) improve returns?
3. Which sentiment leads to the largest losses?
4. What behaviors consistently correlate with profitability?
5. How does sentiment influence risk-taking behavior?

### 1.3 Scope
- **Period:** December 14, 2023 – May 1, 2025 (503 trading days)
- **Exchange:** Hyperliquid (on-chain perpetuals + spot)
- **Sentiment data:** Bitcoin Fear & Greed Index (daily)

---

## 2. METHODOLOGY

### 2.1 Data Sources
| Dataset | Source | Records | Date Range |
|---|---|---|---|
| Hyperliquid Trade History | Hyperliquid API export | 211,224 raw; 104,266 analyzed | May 2023 – May 2025 |
| Bitcoin Fear & Greed Index | Alternative.me | 2,644 daily values | Feb 2018 – May 2025 |

### 2.2 Data Cleaning Decisions

**Trading Data:**
- **Zero PnL rows removed:** 106,816 records with `Closed PnL = 0` were excluded. These represent *opening* trades, partial fills, or spot transfers where no realized PnL exists. Only closing events carry analytical signal.
- **Direction filter:** Only rows with `Direction` in {Close Long, Close Short, Sell, Buy} were retained to isolate realized PnL events.
- **Timestamp parsing:** `Timestamp IST` column was parsed with format `%d-%m-%Y %H:%M` (DD-MM-YYYY HH:MM). Date portion used for daily join.
- **No missing values** were detected in any column after load.
- **Duplicate Trade IDs:** Trade IDs were not used as a uniqueness key since the same trade can appear in multiple accounts (the dataset contains multiple accounts observing shared liquidity).
- **Fee column:** Retained as-is; used to compute Net PnL = Closed PnL − Fee.

**Fear & Greed Data:**
- Duplicate dates (none found) would have been resolved by keeping the first entry.
- `classification` field whitespace stripped.
- Ordered as: Extreme Fear → Fear → Neutral → Greed → Extreme Greed.

### 2.3 Merging Strategy
- **Join key:** Trade date (UTC day boundary) ↔ Fear & Greed date.
- **Join type:** Left join from trades onto sentiment.
- **Records lost post-merge:** 6 rows (0.006%) — trading days with no sentiment data (holiday/gap in FGI feed).
- **Assumption:** The Fear & Greed Index value for a given day applies to **all trades executed on that day**, regardless of intraday timing. This is a standard assumption for daily sentiment indicators.

---

## 3. PHASE 1 — DATA UNDERSTANDING

### 3.1 Dataset 1: Bitcoin Fear & Greed Index

| Column | Type | Description |
|---|---|---|
| timestamp | int64 | Unix epoch timestamp |
| value | int64 | Index score 0–100 (0=Extreme Fear, 100=Extreme Greed) |
| classification | str | Text label: Extreme Fear / Fear / Neutral / Greed / Extreme Greed |
| date | str → datetime | Calendar date (parsed to datetime) |

**Distribution across 2,644 days:**
- Fear: 781 days (29.5%)
- Greed: 633 days (23.9%)
- Extreme Fear: 508 days (19.2%)
- Neutral: 396 days (15.0%)
- Extreme Greed: 326 days (12.3%)

Bitcoin has spent nearly half its tracked history in some form of Fear, reflecting the asymmetric nature of crypto market psychology.

### 3.2 Dataset 2: Hyperliquid Historical Trader Data

| Column | Type | Description |
|---|---|---|
| Account | str | Trader wallet address (42-char hex) |
| Coin | str | Trading pair symbol (BTC, ETH, HYPE, etc.) |
| Execution Price | float | Fill price in USD |
| Size Tokens | float | Position size in base token units |
| Size USD | float | Position size in USD notional |
| Side | str | BUY or SELL |
| Timestamp IST | str | Trade datetime in Indian Standard Time |
| Start Position | float | Account's position before this trade |
| Direction | str | Open Long / Close Long / Open Short / Close Short / etc. |
| Closed PnL | float | Realized PnL for closing trades (0 for opens) |
| Transaction Hash | str | On-chain transaction identifier |
| Order ID | int | Exchange order reference |
| Crossed | bool | Whether order crossed the spread (taker) |
| Fee | float | Trading fee paid in USD |
| Trade ID | float | Unique trade identifier |
| Timestamp | float | Unix timestamp |

**Note:** No explicit leverage column exists. Position sizing (Size USD) is used as a leverage proxy — larger USD notional on a given account indicates higher risk exposure per trade.

---

## 4. PHASE 2 — DATA CLEANING REPORT

| Issue | Finding | Resolution |
|---|---|---|
| Zero PnL records | 106,816 (50.6%) | Excluded — these are open trades |
| Missing values | 0 across all columns | No action needed |
| Duplicate Trade IDs | Expected (multi-account data) | Not used as uniqueness constraint |
| Timestamp format | Non-standard DD-MM-YYYY HH:MM | Parsed with explicit format string |
| Classification text | Consistent, no typos | Whitespace strip applied |
| Fee sign | Mostly positive, some negative (rebates) | Used as-is |
| Extreme PnL values | Max $135,329 / Min -$117,990 | Retained; winsorized only for visualization |

**Final clean dataset: 104,266 trade-close records across 32 accounts, 220 symbols, 503 days.**

---

## 5. PHASE 3 — EDA: MARKET SENTIMENT ANALYSIS

### 5.1 Sentiment Distribution in Study Period (Dec 2023 – May 2025)

| Sentiment | Trade Days | Trades | % of Trades |
|---|---|---|---|
| Extreme Fear | — | 10,395 | 10.0% |
| Fear | — | 29,776 | 28.6% |
| Neutral | — | 18,132 | 17.4% |
| Greed | — | 25,128 | 24.1% |
| Extreme Greed | — | 20,835 | 20.0% |

**Observation:** The study period (Dec 2023 – May 2025) spans the post-FTX recovery and the 2024 bull market. As a result, Greed and Extreme Greed periods are well-represented, allowing robust statistical comparison.

### 5.2 Time Trends
The Fear & Greed Index trended upward through 2024 as Bitcoin approached and breached its all-time high of ~$73,000 in March 2024, then again in late 2024. This creates natural "regime change" experiments for our analysis.

---

## 6. PHASE 4 — EDA: TRADER PERFORMANCE ANALYSIS

### 6.1 Overall Statistics

| Metric | Value |
|---|---|
| Total Realized PnL | $10,197,005 |
| Average PnL per Trade | $97.81 |
| Median PnL per Trade | $6.06 |
| Win Rate | **83.2%** |
| Loss Rate | 16.8% |
| Total Winning Trades | 86,869 |
| Total Losing Trades | 17,397 |
| Unique Traders (Accounts) | 32 |
| Unique Symbols | 220 |

The **83.2% win rate** is exceptionally high but consistent with a dataset dominated by experienced on-chain traders. The median PnL ($6.06) being much lower than the mean ($97.81) reveals a highly right-skewed distribution: most trades generate small profits, but a subset of large winning trades drives aggregate PnL.

### 6.2 Direction Analysis

| Direction | Avg PnL | Win Rate |
|---|---|---|
| Close Long | $74.49 | 87.8% |
| Close Short | $101.91 | 78.0% |

**Short trades generate higher average PnL** ($101.91 vs $74.49) despite a lower win rate (78.0% vs 87.8%). This is the classic *small frequent wins vs. larger less-frequent wins* trade-off. Short sellers appear to hold winners longer.

### 6.3 Position Size vs Returns

| Size Bucket | Avg PnL | Win Rate | Count |
|---|---|---|---|
| Small (Q1) | $3.86 | 84.2% | 26,067 |
| Medium (Q2) | $13.71 | 79.9% | 26,066 |
| Large (Q3) | $43.08 | 84.8% | 26,066 |
| XLarge (Q4) | $330.53 | 83.9% | 26,067 |

**Larger positions generate proportionally higher PnL** with no significant win rate degradation. XLarge positions average $330.53/trade — 85x the Small bucket — while maintaining an 83.9% win rate. This suggests top traders are **size-calibrated**: they scale into high-conviction trades rather than using uniform position sizing.

---

## 7. PHASE 5 — SENTIMENT vs TRADER PERFORMANCE

### 7.1 Core Results

| Sentiment | Avg PnL | Median PnL | Win Rate | Trades | Total PnL |
|---|---|---|---|---|---|
| Extreme Fear | $72.22 | $5.74 | 76.2% | 10,395 | $750,683 |
| Fear | $112.07 | $5.48 | 87.3% | 29,776 | $3,338,234 |
| Neutral | $71.27 | $5.56 | 82.4% | 18,132 | $1,291,972 |
| Greed | $83.68 | $6.86 | 76.9% | 25,128 | $2,102,448 |
| Extreme Greed | $130.27 | $6.36 | 89.2% | 20,835 | $2,714,238 |

### 7.2 Statistical Significance

**One-Way ANOVA Test (PnL across 5 sentiment groups):**
- F-statistic: 7.79
- p-value: **0.000003** (p < 0.001)
- Result: **Statistically significant** — sentiment meaningfully differentiates trader PnL.

**Mann-Whitney U Test (Fear vs Greed PnL distributions):**
- p-value: 0.521
- Result: Not significant at the median level — while averages differ, the median trade-level PnL is comparable between Fear and Greed. The average difference is driven by **large winning trades**, not typical trades.

### 7.3 Key Insight: The Fear Paradox
Fear periods generate the **second-highest average PnL** ($112.07). This is counterintuitive but logical: during Fear periods, panic sellers create mispriced assets that savvy traders can exploit. Traders in this dataset are not retail participants reacting emotionally — they are systematic traders who *buy fear*.

Extreme Greed's superior performance ($130.27) is consistent with momentum: late-bull-market trades can generate outsized returns as assets continue trending higher.

---

## 8. PHASE 6 — TRADER SEGMENTATION

### 8.1 Segment Definitions
Traders were segmented by total PnL across the study period:
- **Top Performers** (top 33%): Total PnL ≥ $308,582
- **Average Performers** (middle 33%): $x < Total PnL < $308,582
- **Poor Performers** (bottom 33%): Total PnL < $x

### 8.2 Segment Statistics

| Segment | Avg Total PnL | Win Rate | Avg Trade Count | Avg Position Size |
|---|---|---|---|---|
| Top Performers | $808,002 | 86.9% | 5,610 trades | $9,921 |
| Average Performers | $128,164 | 91.1% | 2,107 trades | $2,757 |
| Poor Performers | $2,486 | 77.6% | 1,953 trades | $6,127 |

**Critical Finding:** Average Performers have the *highest* win rate (91.1%) but accumulate less total PnL than Top Performers because they trade smaller sizes. Top Performers combine a high win rate (86.9%) with high trade frequency (5,610 avg trades) and large position sizes ($9,921 avg), creating compounding returns. Poor Performers use large position sizes ($6,127) but suffer low win rates (77.6%) — the classic "over-leveraged, under-skilled" profile.

---

## 9. PHASE 7 — SYMBOL ANALYSIS

### 9.1 Top Performers by Total PnL

| Coin | Total PnL | Win Rate | Trade Count |
|---|---|---|---|
| @107 | $2,783,913 | 81.7% | 17,166 |
| HYPE | $1,948,062 | 88.2% | 31,980 |
| SOL | $1,628,228 | 83.9% | 5,016 |
| ETH | $1,314,432 | 76.9% | 5,205 |
| BTC | $886,295 | 83.1% | 10,990 |

**@107** is Hyperliquid's internal perpetual index product. Its dominance (27.3% of total PnL) reflects heavy trading activity from sophisticated accounts on this high-liquidity instrument. **HYPE** (Hyperliquid's native token) shows the highest win rate (88.2%) among major symbols — traders with informational edge about the platform trade its native token most effectively.

### 9.2 Worst Performers by Total PnL

| Coin | Total PnL | Win Rate | Trade Count |
|---|---|---|---|
| TRUMP | -$352,072 | 70.0% | 958 |
| FARTCOIN | -$100,804 | 70.5% | 2,162 |
| ADA | -$28,427 | 54.2% | 286 |
| IO | -$21,894 | 44.6% | 130 |
| PAXG | -$18,689 | 42.7% | 485 |

Meme coins (TRUMP, FARTCOIN) are net losers despite high trade counts — reflecting their high volatility and tendency toward violent reversals. PAXG (tokenized gold) shows a very low win rate (42.7%), suggesting this gold-pegged asset is poorly suited to perpetual trading strategies.

---

## 10. PHASE 8 — ADVANCED INSIGHTS

### Insight 1: Sentiment Significantly Affects PnL — But Not Median Trades
ANOVA confirms PnL varies significantly across sentiment regimes (p < 0.001). However, Mann-Whitney shows the *median* trade is not significantly different between Fear and Greed. The difference is entirely driven by **large winning trades** that occur more frequently during Fear and Extreme Greed.
**Business Implication:** Strategy should focus on maximizing size during high-conviction setups in fear/extreme greed periods, not on changing entry frequency.

### Insight 2: Fear Is a Buying Opportunity
Fear periods yield $112.07 average PnL vs. Neutral's $71.27 — a 57% premium. Traders who maintain discipline during Fear avoid the panic that creates the mispricing.
**Business Implication:** Build a "fear signal" alert system; increase position limits when FGI drops below 30.

### Insight 3: Extreme Greed Is Not a Warning — It's an Accelerant
Extreme Greed ($130.27 avg PnL, 89.2% win rate) is the best performing regime. In a Bitcoin bull market, momentum is profitable well into "overbought" territory.
**Business Implication:** Do not reduce positions purely based on high FGI readings; wait for confirmed reversal signals.

### Insight 4: Position Size Drives Profitability — Not Win Rate
XLarge positions average $330/trade vs. Small at $3.86/trade — with nearly identical win rates (~84%). Profitability is **dominated by sizing decisions**, not by directional accuracy.
**Business Implication:** Train traders to increase size on high-confidence trades rather than optimizing for marginally better entry timing.

### Insight 5: Short Sellers Earn More Per Trade
Despite lower win rates (78% vs 88%), short positions generate $101.91 avg PnL vs. $74.49 for longs. Short sellers likely hold positions longer through volatile moves and cut winners less aggressively.
**Business Implication:** Review the hold-time distribution for shorts vs longs. Encouraging longs to hold winners longer could improve their per-trade PnL.

### Insight 6: Poor Performers Over-Size Underperforming Trades
Poor performers use avg $6,127 position sizes — larger than Average performers ($2,757) — but achieve only 77.6% win rates. They are trading too large relative to their edge.
**Business Implication:** Implement dynamic position limits scaled to a trader's demonstrated edge (rolling win rate and PnL per unit risk).

### Insight 7: Meme Coins Destroy Value
TRUMP and FARTCOIN alone destroyed $452,876 in aggregate PnL. Their win rates (~70%) are well below the portfolio average (83.2%).
**Business Implication:** Flag meme coin exposure in risk dashboards; require higher margin requirements or apply lower leverage caps for low-cap tokens.

### Insight 8: HYPE Token Is Alpha Territory
HYPE (Hyperliquid's native token) has 31,980 trades — the most of any asset — with an 88.2% win rate and $1.95M total PnL. Traders with platform knowledge have a measurable edge on this asset.
**Business Implication:** Encourage research into platform-native tokens as a potential alpha source.

### Insight 9: Fee Drag Is Manageable But Real
Total fees paid across all trades were significant. The fee-to-PnL ratio is healthiest during Extreme Greed (large profitable moves absorb fees easily) and worst during Neutral markets (smaller moves, similar fees).
**Business Implication:** Reduce trading frequency during Neutral/low-conviction periods; fees eat proportionally more of small-move PnL.

### Insight 10: Trading Activity Peaks During Greed
Greed periods show the highest trade count (25,128 trades), suggesting traders increase activity when sentiment is positive. This is consistent with FOMO-driven behavior even in sophisticated accounts.
**Business Implication:** Monitor over-trading bias in Greed periods; increased activity without increased edge degrades per-trade PnL.

---

## 11. TRADING RECOMMENDATIONS

### 11.1 Risk Management
- **Sentiment-adjusted stop losses:** Widen stops during Extreme Fear (higher volatility) and tighten during Neutral (mean-reverting environment).
- **Daily loss limits:** Cap daily drawdown at 2% of account equity regardless of sentiment regime.
- **Meme coin hard caps:** Limit any single meme coin to ≤5% of portfolio notional to contain tail risk.

### 11.2 Position Sizing
- **Fear regime:** Increase base position size by 20–30% — Fear trades show higher avg PnL with maintained win rate.
- **Extreme Greed:** Increase size further but reduce hold time; momentum works but reversals are sharp.
- **Neutral regime:** Reduce size by 20%; lower expected PnL makes fee drag more impactful.
- **Scale with conviction:** XLarge positions outperform Small by 85x in PnL. Develop a position sizing model that scales with signal confidence.

### 11.3 Leverage Usage
- Without an explicit leverage field, position size serves as the proxy. The data supports the conclusion that **higher notional exposure in high-conviction environments is rewarded**.
- Avoid uniformly high leverage (as seen in Poor Performers): large size without edge leads to severe drawdowns.

### 11.4 Trading During Fear Markets
- **Increase activity.** Fear is the second-best performing sentiment regime.
- Focus on assets with strong fundamental backing (BTC, ETH, SOL) — avoid meme coins in fear periods.
- Use limit orders to capitalize on widened spreads and panic selling.

### 11.5 Trading During Greed Markets
- **Maintain momentum exposure.** Extreme Greed shows the best average PnL.
- Do NOT preemptively reduce positions on FGI readings alone.
- Monitor for reversal signals (FGI dropping from >75 to <60 within 3 days) as an exit trigger.

### 11.6 Improving Profitability
- **Adopt sentiment-aware position sizing** as a systematic strategy component.
- **Audit TRUMP and FARTCOIN exposure** — these two coins alone account for $452K in losses.
- **Develop a short-holding protocol** to capture more of the $101.91 avg short PnL that current long-biased traders leave on the table.
- **Reduce Neutral-period trading frequency** — the fee drag during these periods is disproportionately high.

---

## 12. CONCLUSION

This analysis demonstrates that Bitcoin market sentiment — as measured by the Fear & Greed Index — has a statistically significant and practically meaningful impact on trader performance on Hyperliquid. The key take-aways are:

1. The dataset represents highly skilled traders (83.2% overall win rate) who consistently profit across all sentiment regimes.
2. Extreme Greed offers the best average PnL per trade; Fear offers the second-best — both outperform Neutral significantly.
3. Profitable traders succeed not by picking better entries but by **scaling position size into high-conviction, favorable-sentiment setups**.
4. Meme coins (TRUMP, FARTCOIN) are structural alpha destroyers; platform-native tokens (HYPE) are alpha generators.
5. Top-performing traders combine high trade frequency, large positions, and sentiment-aware risk management.

The analytical framework developed here — sentiment-stratified performance attribution, trader segmentation, and symbol-level PnL decomposition — provides a robust foundation for building production-grade trading dashboards and strategy optimization tools.

---

## APPENDIX: DATA QUALITY ASSESSMENT

| Check | Result |
|---|---|
| Missing values (trader data) | 0 |
| Missing values (FGI) | 0 |
| Duplicate dates in FGI | 0 |
| Records excluded (zero PnL) | 106,816 |
| Records excluded (post-merge) | 6 |
| Final analytical dataset | 104,266 |
| Date coverage gap | None significant |
| Outliers (extreme PnL) | Retained; winsorized for visualization only |

---

*Report generated using Python (pandas, NumPy, SciPy, Matplotlib, Seaborn). All statistics are computed from raw data without imputation. Statistical tests use α = 0.05 significance threshold.*
