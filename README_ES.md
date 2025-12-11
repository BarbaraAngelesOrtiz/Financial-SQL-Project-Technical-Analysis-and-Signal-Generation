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

## ⏱️FASE 4: Lógica de Transacciones y Seguimiento del Rendimiento

**Señales de Transacción**

Las señales explícitas de compra y venta se definen en la tabla Datos:

• **Señales de Compra** (compra1, compra2, compra3): Requieren jerarquías específicas de EMA alcistas (p. ej., EMA_10 > EMA_20 > EMA_40), RSI > 60 o reversión del RSI desde condiciones de sobreventa (< 35).

• **Señales de Venta** (venta1, venta2): Se activan cuando el RSI es alto (> 70) y el Cierre Ajustado cae por debajo de una EMA clave (EMA_10 o EMA_20).

• **V1, V2 (Ventas Alternativas):** Se definen en función de las condiciones de cruce y reversión de la EMA, a menudo filtradas por 'RIO'.

**Gestión de Cartera y Métricas**

1. **Generación e Inserción de ID:** Un CTE calcula si Hist_3 mostró dos días consecutivos de crecimiento (HM1_2 = 1). Estos registros, específicamente los de la empresa 'RIO', se insertan en la tabla de Efectivo con un ID secuencial. Las entradas duplicadas en la tabla de Efectivo se eliminan explícitamente.

2. **Precio Promedio de Compra (PROM):** Las funciones de ventana se utilizan para calcular la suma acumulada de los precios de compra (SUMA) y el recuento acumulado de compras (CONT). Esta acumulación se reinicia cada vez que se produce una venta (Venta_Flag basado en V1, V2, V3). El precio promedio (PROM) se calcula como SUMA / CONT.

3. **Retorno de la Inversión (ROI) y P&L:**
◦ El ROI se calcula al momento de una venta (V1 o V2 ​​= 1) comparando el Precio de Venta (Ajuste de Cierre) con el Precio de Compra (el Ajusto de Cierre de la compra anterior, obtenido mediante LAG).
◦ El Resultado se determina comprobando si la diferencia entre el precio de venta y el precio de compra correspondiente es positiva ('Ganancia') o cero/negativa ('Perdida').
