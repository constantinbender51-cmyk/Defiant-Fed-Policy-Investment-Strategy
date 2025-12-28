Quantitative Regime Analysis Engine
Overview
This application serves as an automated quantitative research assistant. It dynamically assesses the current US macroeconomic environment ("Regime") using Federal Reserve data and applies a regime-congruent factor scoring model to the S&P 500 equity universe. The output is a web-based, academic-style report identifying long and short candidates based on standardized factor exposure.
System Architecture & Flow
 * Macro-Engine (FredEngine): Queries the St. Louis Fed (FRED) to determine the economic state.
 * Micro-Engine (StockEngine): Scrapes the S&P 500 universe and fetches fundamental metrics via Finnhub.
 * Analysis Logic: Normalizes data using Z-Scores and ranks assets based on the active regime.
 * Presentation: Renders a minimal, auto-refreshing HTML report via Flask.
I. Operational & Theoretical Assumptions
The system relies on a specific set of heuristic assumptions to operationalize "investment strategy" into code. These are hardcoded into the logic and define the model's worldview.
1. Macroeconomic Regime Definition
The model assumes the economy can be segmented into four distinct regimes based on the rate of change of Liquidity (Federal Reserve Balance Sheet) and Cost of Capital (Fed Funds Rate).
 * Assumption: The "trend" is defined by the divergence of the current value from a rolling moving average.
   * Interest Rates (Ir): A 5-year (260-week) rolling average is used as the baseline.
   * Balance Sheet (Bs): A 1-year (52-week) rolling average is used as the baseline.
| Regime | Logic | Description | Strategy Assumption |
|---|---|---|---|
| A | Ir < \mu_{Ir} & Bs > \mu_{Bs} | Expansion | Liquidity is high, money is cheap. Favor Growth. |
| B | Ir < \mu_{Ir} & Bs \le \mu_{Bs} | Deflation | Money is cheap but liquidity is contracting. Balanced. |
| C | Ir \ge \mu_{Ir} & Bs > \mu_{Bs} | Inflationary Boom | Costs rising but liquidity remains high. Balanced. |
| D | Ir \ge \mu_{Ir} & Bs \le \mu_{Bs} | Tightening | Costly money and draining liquidity. Favor Profit/Value. |
2. Factor Construction & Scoring
The model assumes that raw fundamental data is insufficient for ranking and must be standardized relative to the peer group (S&P 500).
 * Profit Score Formulation:
   The model creates a custom metric combining operational efficiency and valuation:
   
   
   Assumption: High margins are only valuable if bought at a reasonable earnings yield.
 * Z-Score Normalization:
   To make "Growth" and "Profit" comparable, they are converted to Z-scores (z = \frac{x - \mu}{\sigma}).
   Assumption: A stock is "good" if it is statistically significant (positive outliers) relative to the current market distribution.
3. Universe Selection
 * Source: The model assumes Wikipedia's "List of S&P 500 companies" is an accurate, up-to-date source of truth for the investable universe.
 * Filter: It strictly removes any instrument with a PE \le 0 or missing growth data, assuming unprofitable companies are too volatile for this specific logic.
II. Technical Constraints & Setup
Dependencies
The application requires the following Python libraries:
flask
pandas
numpy
requests
lxml (or html5lib)

Environment Variables
The system expects two API keys to be present in the environment variables. Without these, the engines will initialize but return empty datasets.
| Variable | Description | Source |
|---|---|---|
| FRED_API_KEY | Access to Federal Reserve Economic Data. | stlouisfed.org |
| FINNHUB_API_KEY | Access to stock fundamentals. | finnhub.io |
Execution
Run the application directly via Python. The analysis runs in a background thread to prevent blocking the web server startup.
python app.py

 * Port: Defaults to 5000 (or uses $PORT).
 * Startup Time: The initial analysis involves fetching data for up to 500 stocks with a 1-second delay per request (to avoid rate limits). Expect a ~10-minute warmup time before data appears on the web page.
III. Known Limitations
 * Data Latency: The system fetches data once upon startup (triggered by threading.Thread). It does not re-run the analysis automatically on a schedule. To refresh the data, the application must be restarted.
 * Rate Limiting: The StockEngine enforces a strict time.sleep(1.0) between requests. While this ensures compliance with free-tier API limits, it significantly slows down the generation of the report.
 * Memory persistence: Global state (APP_DATA) is held in memory. If the application crashes or restarts, historical analysis is lost.
Disclaimer: This software is for educational and research purposes only. It does not constitute financial advice, and the "Regime" logic is a simplified heuristic model.
