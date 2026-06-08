# Bitcoin Fear & Greed vs Hyperliquid Trader Performance
## Quantitative Analysis Project

---

## Project Overview

This project analyzes the relationship between **Bitcoin market sentiment** (Fear & Greed Index) and **trader performance** on the Hyperliquid decentralized perpetuals exchange. The goal is to determine whether — and how — market sentiment regimes affect trading outcomes including PnL, win rate, trade frequency, and risk-taking behavior.

This analysis was completed as a hiring assignment for a Web3 trading company and follows professional data science standards suitable for institutional review.

---

## Dataset Description

### Dataset 1: Bitcoin Fear & Greed Index (`fear_greed_index.csv`)
| Column | Description |
|---|---|
| timestamp | Unix epoch timestamp |
| value | Daily index score (0–100) |
| classification | Extreme Fear / Fear / Neutral / Greed / Extreme Greed |
| date | Calendar date |

- **Source:** Alternative.me
- **Coverage:** February 2018 – May 2025 (2,644 daily records)

### Dataset 2: Hyperliquid Historical Trader Data (`historical_data.csv`)
| Column | Description |
|---|---|
| Account | Trader wallet address |
| Coin | Trading instrument |
| Execution Price | Fill price in USD |
| Size Tokens | Position in base token |
| Size USD | Position notional in USD |
| Side | BUY or SELL |
| Timestamp IST | Datetime in IST (DD-MM-YYYY HH:MM) |
| Start Position | Pre-trade position |
| Direction | Open/Close Long/Short |
| Closed PnL | Realized PnL (0 for opens) |
| Transaction Hash | On-chain identifier |
| Fee | Trading fee in USD |
| Trade ID | Exchange trade reference |

- **Source:** Hyperliquid on-chain data
- **Raw Records:** 211,224 | **Analyzed Records:** 104,266 (close trades with PnL)
- **Coverage:** December 2023 – May 2025

---

## Setup Instructions

### Prerequisites
```bash
Python 3.8+
pip install pandas numpy matplotlib seaborn scipy
```

### Running the Analysis
```bash
# Clone / download project files
cd project_directory/

# Place datasets in working directory:
#   historical_data.csv
#   fear_greed_index.csv

# Run the Jupyter notebook
jupyter notebook Bitcoin_FearGreed_Analysis.ipynb

# Or run the Python script directly
python analysis.py
```

### Output Files
All outputs are saved to the `outputs/` directory:
- `fig1_sentiment_distribution.png` — Sentiment frequency, share, and time trend
- `fig2_trader_performance.png` — Overall PnL, win rate, coin breakdown
- `fig3_sentiment_vs_performance.png` — Avg/median/total PnL and win rate by sentiment
- `fig4_pnl_boxplot_sentiment.png` — PnL distribution box and violin plots
- `fig5_risk_size_analysis.png` — Position size vs returns and risk analysis
- `fig6_trader_segmentation.png` — Top/mid/poor performer profiles
- `fig7_symbol_analysis.png` — Best/worst coins and symbol scatter
- `fig8_correlation_stats.png` — Correlation heatmap and statistical tests
- `fig9_timeseries.png` — Cumulative PnL and sentiment time series
- `fig10_advanced_patterns.png` — Hour/day patterns and sentiment transitions
- `analysis_report.md` — Full written report
- `Bitcoin_FearGreed_Analysis.ipynb` — Complete Jupyter notebook
- `interview_prep.md` — Interview questions and answers

---

## Analysis Summary

### Methodology
1. **Data Cleaning:** Removed zero-PnL open trades; parsed IST timestamps; standardized sentiment categories.
2. **Integration:** Left-joined trading records to daily Fear & Greed values on trade date.
3. **EDA:** Full statistical profiling of PnL distribution, win rates, trade counts, and volume across sentiment regimes.
4. **Segmentation:** Traders segmented into Top/Average/Poor performers by total PnL.
5. **Statistical Testing:** One-way ANOVA (F=7.79, p<0.001) and Mann-Whitney U test.
6. **Advanced Analysis:** Hour-of-day patterns, day-of-week effects, sentiment transition matrices.

### Key Results

| Finding | Evidence |
|---|---|
| Sentiment significantly affects PnL | ANOVA p = 0.000003 |
| Best performing regime | Extreme Greed ($130.27 avg PnL, 89.2% win rate) |
| Fear is a buy signal | Fear avg PnL $112.07 — 2nd highest |
| Size drives PnL more than accuracy | XLarge positions: $330/trade; Small: $3.86/trade |
| Meme coins destroy value | TRUMP + FARTCOIN = -$452K combined |
| Platform native token = edge | HYPE: $1.95M PnL, 88.2% win rate |
| Short trades outperform per trade | $101.91 avg PnL vs $74.49 for longs |
| Overall win rate | 83.2% across 104,266 trades |
| Total realized PnL | $10,197,005 |

---

## Project Structure

```
project/
├── README.md                          ← This file
├── Bitcoin_FearGreed_Analysis.ipynb   ← Complete Jupyter Notebook
├── analysis_report.md                 ← Full analysis report
├── interview_prep.md                  ← Interview Q&A
├── data/
│   ├── historical_data.csv
│   └── fear_greed_index.csv
└── outputs/
    ├── fig1_sentiment_distribution.png
    ├── fig2_trader_performance.png
    ├── fig3_sentiment_vs_performance.png
    ├── fig4_pnl_boxplot_sentiment.png
    ├── fig5_risk_size_analysis.png
    ├── fig6_trader_segmentation.png
    ├── fig7_symbol_analysis.png
    ├── fig8_correlation_stats.png
    ├── fig9_timeseries.png
    └── fig10_advanced_patterns.png
```

---

## Technologies Used
- **Python 3.12**
- **pandas** — Data manipulation and merging
- **NumPy** — Numerical computing
- **Matplotlib / Seaborn** — Visualizations
- **SciPy** — Statistical testing (ANOVA, Mann-Whitney U)

---

*Prepared by: Senior Data Scientist & Quantitative Trading Analyst*
*Assignment: Web3 Trading Company Hiring Project — June 2025*
