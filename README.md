# Defiant-Fed-Policy-Investment-Strategy

Quantitative Methodology: Regime-Based Factor Rotation
This document outlines the formal logic used to categorize economic environments, evaluate stock performance, and manage risk within the S&P 500 Regime Strategy.

## Mathematical Description of Assumptions

### 1. Economic Regime Classification
The strategy identifies market cycles by comparing current macroeconomic indicators against their historical averages (Simple Moving Averages).

**Indicator Definitions**
- IR: Effective Federal Funds Rate (monthly data)
- BS: Federal Reserve Total Assets (Balance Sheet) (weekly data)
- SMA_260: 260-week (5-year) Simple Moving Average for IR (approx 260 weeks, min periods 50)
- SMA_52: 52-week (1-year) Simple Moving Average for BS (min periods 20)

**Assumptions:**
- Historical data is fetched for 10 years to ensure sufficient data for moving averages.
- Data is aligned via merge_asof with backward fill for IR onto BS dates.
- Missing or invalid values (e.g., '.' in FRED data) are filtered out.

**Mathematical Formulation:**
Let:
- \( \text{IR}_{\text{curr}} \) = current IR value
- \( \text{IR}_{\text{avg}} \) = SMA_260(IR) over historical data
- \( \text{BS}_{\text{curr}} \) = current BS value
- \( \text{BS}_{\text{avg}} \) = SMA_52(BS) over historical data

Define boolean conditions:
- High Rate: \( H_R = (\text{IR}_{\text{curr}} > \text{IR}_{\text{avg}}) \)
- High Liquidity: \( H_L = (\text{BS}_{\text{curr}} > \text{BS}_{\text{avg}}) \)

Regime classification:
- Regime A: \( \neg H_R \land H_L \) (Low Rate, High Liquidity)
- Regime B: \( \neg H_R \land \neg H_L \) (Low Rate, Low Liquidity)
- Regime C: \( H_R \land H_L \) (High Rate, High Liquidity)
- Regime D: \( H_R \land \neg H_L \) (High Rate, Low Liquidity)

**Regime Matrix**
| Regime | Rate Trend | Liquidity Trend | Economic Context | Factor Focus |
|---|---|---|---|---|
| A | LOW | EXPANSION | Easy Money / High Liquidity | Growth (Z_g) |
| B | LOW | CONTRACTION | Transition / Neutral | Balanced |
| C | HIGH | EXPANSION | Inflationary / Overheating | Balanced |
| D | HIGH | CONTRACTION | Tightening / Risk-Off | Profitability (Z_p) |
### 2. Factor Normalization (Z-Scores)
To compare different metrics (like % growth vs profit margins), we use cross-sectional Z-scores. This measures how many standard deviations a stock is from the S&P 500 average.

**Mathematical Formulation:**
For a given metric \( X \) (e.g., Growth or ProfitScore) across all stocks in the universe:
- \( \mu_X \) = mean of \( X \) across all stocks
- \( \sigma_X \) = standard deviation of \( X \) across all stocks
- For each stock \( i \) with value \( x_i \):
  \[ Z_{X,i} = \frac{x_i - \mu_X}{\sigma_X} \]

**Assumptions:**
- Data is cleaned by dropping rows with missing values in PE, Margin, or Growth.
- Only stocks with PE > 0 are included to avoid division by zero or negative earnings.
- Metrics defined:
  - Growth: Revenue Growth Quarterly YoY (as percentage)
  - ProfitScore: Operating Margin * (1 / PE), where Operating Margin and PE are TTM values.

**Z-scores computed:**
- \( Z_g \): Z-score for Growth
- \( Z_p \): Z-score for ProfitScore

### 3. Factor Scoring Engine
The final score for each stock determines its rank in the portfolio.

**Mathematical Formulation:**
Let:
- \( w_g \) = weight for growth factor
- \( w_p \) = weight for profitability factor
- Final score for stock \( i \):
  \[ S_i = w_g \cdot Z_{g,i} + w_p \cdot Z_{p,i} \]

**Regime Weights (Assumptions):**
The strategy rotates factor weights based on the regime:
- Regime A: \( w_g = 1.0, w_p = 0.0 \) (Growth focus)
- Regime D: \( w_g = 0.0, w_p = 1.0 \) (Profit focus)
- Regimes B & C: \( w_g = 0.5, w_p = 0.5 \) (Balanced focus)

**Assumption:** This weighting scheme is predefined and not dynamically optimized.

### 4. Portfolio Construction
The strategy is "Market Neutral," meaning it bets on the relative performance of stocks rather than the direction of the overall market.

**Mathematical Formulation:**
Let \( N \) = number of stocks selected for longs/shorts (e.g., 100 in full backtest, 15 in app for display).
- Sort all stocks by \( S_i \) in descending order.
- Longs: Top \( N \) stocks with highest \( S_i \)
- Shorts: Bottom \( N \) stocks with lowest \( S_i \)

**Weighting (Assumptions):**
- Each long position weight: \( +\frac{0.5}{100} \) (i.e., +0.5% of portfolio)
- Each short position weight: \( -\frac{0.5}{100} \) (i.e., -0.5% of portfolio)
- Total positions: \( 2N \)

**Exposure:**
- Gross Exposure = \( \sum |\text{weight}| = 100\% \) (50% long + 50% short)
- Net Exposure = \( \sum \text{weight} = 0\% \) (Market Neutral)

**Assumption:** Equal dollar weighting per position for simplicity.

### 5. Risk Management
**Trailing Stop-Loss (Assumptions):**
Each position tracks its "Extreme Price" (EP): highest price since entry for longs, lowest for shorts.
- Initial Stop: Exit if price reaches 20% loss from entry price.
- Profit Trigger: If position gains 20%, activate trailing stop.
- Trailing Stop: If active, exit if price drops 10% from EP.

**Portfolio Value Tracking:**
Let:
- \( V_t \) = portfolio value at time \( t \)
- \( r_{i,t} \) = return of position \( i \) at time \( t \)
- \( w_i \) = weight of position \( i \)
\[ V_{t+1} = V_t \cdot \left(1 + \sum_i w_i \cdot r_{i,t}\right) \]

**Assumption:** Returns are calculated based on price changes, ignoring transaction costs and slippage.
