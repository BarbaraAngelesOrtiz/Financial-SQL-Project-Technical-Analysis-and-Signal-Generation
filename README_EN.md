# 💡Financial SQL Project: Backtesting, Technical Analysis and Signal Generation

This project consists of the creation of a database, whose data is obtained from an API. The SQL scripts, primarily using Recursive Common Table Expressions (CTEs) and UPDATE commands on the Data table, designed to perform advanced technical analysis and generate complex buying and selling signals for financial assets, often filtered for several stocks.

----

## 📝PHASE 1: Indicator Initialization and Simple Comparisons

The initial phase focuses on establishing foundational binary indicators (R, E, H) by comparing current values to fixed thresholds or immediate prior values.

| Indicator Set                | Calculation Summary                                                                                 | 
|------------------------------|-----------------------------------------------------------------------------------------------------|
| **RSI Indicators (R1_x)**    | Sets binary flags (1/0) based on whether the RSI_SMA falls below thresholds such as **25, 35, 45** (R1_3, R1_2, R1_1). | 
| **RSI Momentum (R2, R3, R4)**| R2: today’s RSI_SMA > yesterday's. <br> R3: RSI value > RSI_SMA.                                     |      
| **EMA Acceleration (E1, E4)**| Checks if the rate of change of **EMA_5** (E1) or **EMA_10** (E4) has been increasing for 3 consecutive days. |        
| **Price vs. EMA (E2, E5)**   | Checks if Adj_Close > EMA_5 (E2) or Adj_Close > EMA_10 (E5), optionally scaled by factors A or B.   |        
| **Historical (H1–H6)**       | H1/H3/H5: HIST_1, HIST_2, HIST_3 > 0. <br> H2/H4/H6: positive crossover (negative yesterday → positive today). |        

----

## 📊PHASE 2: Calculation of Sustained Streaks (Recursive CTEs)
This phase uses Recursive CTEs with high recursion limits (OPTION (MAXRECURSION 10000)) to calculate the duration (streak count) of specific relationships. Positive counts denote bullish streaks (EMA A > EMA B) and negative counts denote bearish streaks (EMA A < EMA B).

| Counter (Column)            | Condition Tracked                                                                                                                          |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **R21 / C1**                | Measures the sustained streak of the relationship between **RSI_SMA3** and **RSI_SMA7**. The final **C1** value is set when R21 = 1.        |
| **E1_1, E1_2, E1_3, E1_4**  | Track streaks of EMA cross-relationships: <br> • **E1_1:** EMA_5 vs EMA_10 <br> • **E1_2:** EMA_10 vs EMA_20 <br> • **E1_3:** EMA_20 vs EMA_40 <br> • **E1_4:** EMA_10 vs EMA_40 |
| **E2_1, E2_2, E2_3, E2_4**  | Track streaks of the relationship between EMA (5, 10, 20, 40) and **Adj_Close** price.                                                     |
| **H1_1, H1_2, H1_3**        | Track the streak (positive/negative) of **HIST_1**, **HIST_2**, **HIST_3** indicators.                                                      |
| **H2_1, H2_2, H2_3**        | Track the streak of the **velocity** of historical indicators (whether HIST_x increases or decreases day-over-day).                         |

----

## 📈PHASE 3: Complex Market States and Compound Signals (F & E3)

This phase uses the sustained streaks calculated above to define market conditions and generate trigger signals.

1. **E3 Classification:** Categorizes the market state by defining six distinct hierarchies among EMA_10, EMA_20, and EMA_40 (Scenarios 1 through 6), otherwise setting the value to 0.
   
2. **Compound Signals (F1, F5, F6):** These combine multiple streak counters:
    ◦ F1_1, F1_2: Use thresholds on EMA cross streaks (e.g., E1_3 >= 20 or E1_1 = 1 or 2) to determine the signal.
   
    ◦ F5_1, F5_2: Define extreme conditions, often triggering a signal (1) when bearish streaks are very long (e.g., E1_3 <= -80 or combinations of negative streaks in E1_4 and E2_4).
   
    ◦ F6_x: Use combinations of EMA streaks (E1_4, E2_2) and velocity streaks (H2_2).

----
   
## ⏱️PHASE 4: Transaction Logic and Performance Tracking

**Transaction Signals**

Explicit buy and sell signals are defined on the Datos table:

• **Buy Signals** (compra1, compra2, compra3): Require specific bullish EMA hierarchies (e.g., EMA_10 > EMA_20 > EMA_40), RSI > 60, and/or RSI reversing from oversold conditions (< 35).

• **Sell Signals** (venta1, venta2): Triggered when RSI is high (> 70) and Adj_Close drops below a key EMA (EMA_10 or EMA_20).

• **V1, V2 (Alternative Sales):** Defined based on EMA cross conditions and reversals, often filtered for 'RIO'.

**Portfolio Management and Metrics**

1. **ID Generation and Insertion:** A CTE calculates if Hist_3 showed two consecutive days of growth (HM1_2 = 1). These records, specifically for the company 'RIO', are inserted into the Cash table with a sequential ID. Duplicate entries in the Cash table are then explicitly removed.

2. **Average Purchase Price (PROM):** Window functions are used to calculate the running sum of purchase prices (SUMA) and the running count of purchases (CONT). This accumulation is reset whenever a sale occurs (Venta_Flag based on V1, V2, V3). The average price (PROM) is calculated as SUMA / CONT.

3. **Return on Investment (ROI) and P&L:**
   
    ◦ ROI is calculated at the time of a sale (V1 or V2 = 1) by comparing the Precio_Venta (Adj_Close) with the Precio_Compra (the Adj_Close of the preceding purchase, retrieved using LAG).
   
    ◦ Profit or Loss (resultado) is determined by checking if the difference (diferencia) between the sale price and the corresponding purchase price is positive ('Ganancia') or zero/negative ('Perdida').

----
## 📌 Summary

This project is a robust and innovative SQL-based backtesting engine. Its foundation is strong, and with a few optimizations it can scale efficiently while remaining clean and extensible.

## 🚀 Key Takeaways

1. Advanced SQL for Time-Series Analysis

The project makes impressive use of recursive CTEs and sequential logic to compute streaks, momentum, and multi-layer indicators. Components like Counter, EMA acceleration (E1_x), and RSI streaks (R21) highlight deep mastery of SQL’s analytical capabilities.

2. Well-Designed and Sophisticated Signals

Signals such as F1, F5, and F6 incorporate duration, momentum, trend fatigue, and historical context—far beyond simple moving-average crossovers. This leads to more realistic and nuanced backtesting.

3. Realistic Backtesting Engine

The handling of average entry price, position resets, buy/sell flags, and ROI calculation reflects a system designed to mimic real trading conditions with precision.

----

## 🔧 Future Improvements
1. Optimize Recursive CTEs

Recursive CTEs are powerful but expensive. Consider evaluating:

- CLR-based user-defined functions (UDFs)

- Efficient procedural logic (when supported)

- Temporary table optimizations

2. Consolidate Multiple UPDATE Statements

Many simple indicators can be calculated in a single statement or CTE. Consolidation will improve performance and maintainability.

3. Prefer Window Functions (LAG / LEAD)

Standardizing sequential logic with window functions can simplify code and significantly improve performance.

4. Strengthen Numeric Precision

Ensure all EMA and historical indicator columns use appropriate DECIMAL/NUMERIC precision, reducing the need for repeated rounding inside CTEs.

----

## Author
**Bárbara Ángeles Ortiz**

<img src="https://github.com/user-attachments/assets/30ea0d40-a7a9-4b19-a835-c474b5cc50fb" width="115">

[LinkedIn](https://www.linkedin.com/in/barbaraangelesortiz/) | [GitHub](https://github.com/BarbaraAngelesOrtiz)

![Status](https://img.shields.io/badge/status-finished-brightgreen) 📅 December 2025

![SQL Server](https://img.shields.io/badge/Sql-Server-orange)
