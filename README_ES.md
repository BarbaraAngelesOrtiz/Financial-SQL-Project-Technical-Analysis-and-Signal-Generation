# 💡Proyecto SQL Financiero: Análisis Técnico y Generación de Señales

Este proyecto consiste en la creación de una base de datos, cuyos datos se obtienen de una API. Los scripts SQL, que utilizan principalmente Expresiones Comunes de Tabla (CTE) recursivas y comandos UPDATE en la tabla de datos, están diseñados para realizar análisis técnicos avanzados y generar señales complejas de compra y venta de activos financieros, a menudo filtradas para varias acciones.

----

## 📝FASE 1: Inicialización de Indicadores y Comparaciones Simples

La fase inicial se centra en establecer indicadores binarios fundamentales (R, E, H) comparando los valores actuales con umbrales fijos o valores inmediatamente anteriores.

| Conjunto de Indicadores | Resumen del Cálculo |
|----------------------------------|--------------------------------------------------------------------------------------------------|
| **Indicadores RSI (R1_x)** | Establece indicadores binarios (1/0) en función de si el RSI_SMA cae por debajo de umbrales como **25, 35, 45** (R1_3, R1_2, R1_1). |
| **Momentum RSI (R2, R3, R4)**| R2: RSI_SMA de hoy > ayer. <br> R3: Valor RSI > RSI_SMA. |
| **Aceleración EMA (E1, E4)**| Comprueba si la tasa de cambio de la **EMA_5** (E1) o la **EMA_10** (E4) ha aumentado durante 3 días consecutivos. |
| **Precio vs. EMA (E2, E5)** | Comprueba si el Cierre Ajustado > EMA_5 (E2) o el Cierre Ajustado > EMA_10 (E5), opcionalmente escalado por los factores A o B. |
| **Histórico (H1–H6)** | H1/H3/H5: HIST_1, HIST_2, HIST_3 > 0. <br> H2/H4/H6: cruce positivo (negativo ayer → positivo hoy). |

----

## 📊FASE 2: Cálculo de Rachas Sostenidas (CTE Recursivos)
Esta fase utiliza CTE Recursivos con altos límites de recursión (OPCIÓN (MAXRECURSION 10000)),,,,,,, para calcular la duración (número de rachas) de relaciones específicas. Los números positivos indican rachas alcistas (EMA A > EMA B) y los números negativos, rachas bajistas (EMA A < EMA B).

| Contador (Columna) | Condición Monitoreada |
|------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| **R21 / C1** | Mide la racha sostenida de la relación entre **RSI_SMA3** y **RSI_SMA7**. El valor final de **C1** se establece cuando R21 = 1. |
| **E1_1, E1_2, E1_3, E1_4** | Rastrear las rachas de las relaciones cruzadas de la EMA: <br> • **E1_1:** EMA_5 vs. EMA_10 <br> • **E1_2:** EMA_10 vs. EMA_20 <br> • **E1_3:** EMA_20 vs. EMA_40 <br> • **E1_4:** EMA_10 vs. EMA_40 |
| **E2_1, E2_2, E2_3, E2_4** | Rastrear las rachas de la relación entre la EMA (5, 10, 20, 40) y el precio de **Cierre Ajustado**. |
| **H1_1, H1_2, H1_3** | Rastrear la racha (positiva/negativa) de los indicadores **HIST_1**, **HIST_2**, **HIST_3**. |
| **H2_1, H2_2, H2_3** | Seguimiento de la racha de la **velocidad** de los indicadores históricos (si HIST_x aumenta o disminuye día a día). |

----

## 📈FASE 3: Estados Complejos del Mercado y Señales Compuestas (F y E3)

Esta fase utiliza las rachas sostenidas calculadas anteriormente para definir las condiciones del mercado y generar señales de activación.

1. **Clasificación E3:** Categoriza el estado del mercado mediante la definición de seis jerarquías distintas: EMA_10, EMA_20 y EMA_40 (Escenarios 1 a 6); de lo contrario, el valor se establece en 0.

2. **Señales Compuestas (F1, F5, F6):** Combinan múltiples contadores de rachas:
◦ F1_1, F1_2: Utilizan umbrales en las rachas cruzadas de la EMA (p. ej., E1_3 >= 20 o E1_1 = 1 o 2) para determinar la señal. ◦ F5_1, F5_2: Definen condiciones extremas, que suelen activar una señal (1) cuando las rachas bajistas son muy largas (p. ej., E1_3 <= -80 o combinaciones de rachas negativas en E1_4 y E2_4).

◦ F6_x: Utilizan combinaciones de rachas de EMA (E1_4, E2_2) y rachas de velocidad (H2_2).

----

## ⏱️PHASE 4: Transaction Logic and Performance Tracking

**Transaction Signals**

Explicit buy and sell signals are defined on the Datos table:

• **Buy Signals** (compra1, compra2, compra3): Require specific bullish EMA hierarchies (e.g., EMA_10 > EMA_20 > EMA_40), RSI > 60, and/or RSI reversing from oversold conditions (< 35).

• **Sell Signals** (venta1, venta2): Triggered when RSI is high (> 70) and Adj_Close drops below a key EMA (EMA_10 or EMA_20).

• **V1, V2 (Alternative Sales):** Defined based on EMA cross conditions and reversals, often filtered for 'RIO'.

**Portfolio Management and Metrics**

1. **ID Generation and Insertion:** A CTE calculates if Hist_3 showed two consecutive days of growth (HM1_2 = 1). These records, specifically for the company 'RIO', are inserted into the Cash table with a sequential ID. Duplicate entries in the Cash table are then explicitly removed.

2. **Average Purchase Price (PROM):** Window functions are used to calculate the running sum of purchase prices (SUMA) and the running count of purchases (CONT). This accumulation is reset whenever a sale occurs (Venta_Flag based on V1, V2, V3),. The average price (PROM) is calculated as SUMA / CONT.

3. **Return on Investment (ROI) and P&L:**
    ◦ ROI is calculated at the time of a sale (V1 or V2 = 1) by comparing the Precio_Venta (Adj_Close) with the Precio_Compra (the Adj_Close of the preceding purchase, retrieved using LAG),,,.
    ◦ Profit or Loss (resultado) is determined by checking if the difference (diferencia) between the sale price and the corresponding purchase price is positive ('Ganancia') or zero/negative ('Perdida').
