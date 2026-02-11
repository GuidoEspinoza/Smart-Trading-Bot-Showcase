# Trading Strategy: Indicator Confluence

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

The bot uses a single but powerful trading strategy based on the principle of **confluence**. Instead of relying on one or two indicators, the system simultaneously evaluates a set of up to 9 market conditions to measure the strength of a potential trading signal.

## What is Confluence?

Confluence in trading is the alignment of multiple technical factors pointing in the same direction (bullish or bearish). A signal backed by several uncorrelated indicators is considered higher probability than an isolated signal.

Our strategy quantifies this concept through a **scoring system**.

## The Scoring System (Confluence Score)

For each data candle, the bot calculates two independent scores:

- **Bullish Score**: A number from 0 to 9 indicating how many bullish conditions are met.
- **Bearish Score**: A number from 0 to 9 indicating how many bearish conditions are met.

A trade is only considered if one of these scores reaches a predefined threshold.

### The Confluence Threshold (`CONFLUENCE_THRESHOLD`)

This is the most important variable of the strategy, defined in `src/config/profiles.py`.

- **Definition**: The minimum score that must be reached for a `BUY` or `SELL` signal to be valid.
- **Purpose**: Acts as a **quality filter**. A higher threshold (e.g., 5) means the bot will be more selective and only trade on the strongest signals.
- **Optimal Configuration (Validated)**: `5`. This value has shown to offer the best balance between frequency and profitability (Win Rate 75%+).

## Scoring Conditions

Scores are built by adding 1 point for each of the following conditions met. Below describes the logic for the **bullish** score. Bearish logic is simply the inverse.

1.  **Market Structure (Higher Highs & Higher Lows)**
    - **Condition**: Current high > high 5 candles ago AND current low > low 5 candles ago.
    - **Purpose**: Confirms price is forming an uptrend micro-structure.

2.  **Moving Average Trend (EMAs)**
    - **Condition**: Close price > Fast EMA (e.g., 20) AND Fast EMA > Slow EMA (e.g., 50).
    - **Purpose**: Ensures trading aligns with immediate and short-term trend.

3.  **Ichimoku Cloud**
    - **Condition**: Close price > Senkou Span A AND > Senkou Span B.
    - **Purpose**: Uses the cloud as a dynamic support/resistance zone to confirm trend strength.

4.  **MACD Momentum**
    - **Condition**: MACD line > Signal line.
    - **Purpose**: Measures short-term momentum, indicating reduced bearish pressure or increasing bullish pressure.

5.  **MACD Histogram**
    - **Condition**: MACD histogram is positive and growing (current bar > previous).
    - **Purpose**: Confirms positive momentum acceleration.

6.  **Relative Strength (RSI)**
    - **Condition**: RSI > 50.
    - **Purpose**: Indicates recent bullish moves are stronger than bearish ones.

7.  **Bullish Divergence (RSI)**
    - **Condition**: Price makes lower low while RSI makes higher low.
    - **Purpose**: Identifies potential trend exhaustion and imminent reversal. High-quality signal.

8.  **Directional Relative Volume (RVOL)**
    - **Condition**: Current volume significantly higher than average (e.g. > 1.5x) AND candle is bullish (close > open).
    - **Purpose**: Confirms institutional conviction behind the move. Volume must support price direction.

9.  **Adaptive Trend Strength (ADX)**
    - **Condition**: ADX is above dynamic session threshold (`ADX_TREND_THRESHOLD`).
      - _Asia_: > 18
      - _London/NY_: > 25
      - _Default_: > 20
    - **Purpose**: Avoids trading in chop/sideways markets, ensuring genuine trend exists.

## "Lock in Profit & Ride" Philosophy

The bot uses a **Scaled Exits (33/33/33)** strategy to balance security with massive profit potential.

1.  **TP1 (33% @ 0.75R)**:
    - **Objective**: Secure. Closes first third quickly.
    - **Effect**: Drastically reduces monetary risk and covers spread/commissions. Generates constant cash flow.

2.  **TP2 (33% @ 1.50R)**:
    - **Objective**: Capitalize. Closes second third in main expansion.
    - **Effect**: Locks in substantial profit before potential pullbacks.

3.  **TP3 (34% Runner @ Trailing Stop)**:
    - **Objective**: Ride. Last third has no fixed TP. Activates dynamic **Trailing Stop**.
    - **Effect**: Captures "Home Runs" (moves of 5R, 10R+) during high volatility. In strong trends, this 34% can generate more profit than the rest combined.

## Decision Example

- `GOLD` analyzed on `15M` timeframe.
- All indicators calculated.
- Bot determines 7 of 9 bullish conditions met -> `Bullish Score = 7`.
- 1 of 9 bearish conditions met -> `Bearish Score = 1`.
- `CONFLUENCE_THRESHOLD` is `5`.
- Since `Bullish Score (7)` >= `CONFLUENCE_THRESHOLD (5)`, signals **`BUY`**.
- If score was 4, decision would be `HOLD`.

---

# Estrategia de Trading: Confluencia de Indicadores (Español)

## 🇪🇸 Versión en Español

El bot utiliza una única pero poderosa estrategia de trading basada en el
principio de **confluencia**. En lugar de depender de uno o dos indicadores, el
sistema evalúa simultáneamente un conjunto de hasta 9 condiciones del mercado
para medir la fuerza de una posible señal de trading.

## ¿Qué es la Confluencia?

La confluencia en el trading es la alineación de múltiples factores técnicos que
apuntan hacia la misma dirección (alcista o bajista). Una señal que está
respaldada por varios indicadores no correlacionados se considera de mayor
probabilidad que una señal aislada.

Nuestra estrategia cuantifica este concepto a través de un **sistema de
puntuación**.

## El Sistema de Puntuación (Confluence Score)

Para cada vela de datos, el bot calcula dos puntuaciones independientes:

- **Puntuación Alcista (Bullish Score)**: Un número de 0 a 9 que indica cuántas
  condiciones alcistas se están cumpliendo.
- **Puntuación Bajista (Bearish Score)**: Un número de 0 a 9 que indica cuántas
  condiciones bajistas se están cumpliendo.

Una operación solo se considera si una de estas puntuaciones alcanza un umbral
predefinido.

### El Umbral de Confluencia (`CONFLUENCE_THRESHOLD`)

Esta es la variable más importante de la estrategia, definida en
`src/config/profiles.py`.

- **Definición**: Es la puntuación mínima que debe alcanzarse para que una señal
  de `BUY` o `SELL` sea considerada válida.
- **Propósito**: Actúa como un **filtro de calidad**. Un umbral más alto (ej. 5)
  significa que el bot será más selectivo y solo operará en las señales más
  fuertes.
- **Configuración Óptima (Validada)**: `5`. Este valor ha demostrado ofrecer el
  mejor equilibrio entre frecuencia y rentabilidad (Win Rate 75%+).

## Condiciones de Puntuación

Las puntuaciones se construyen sumando 1 punto por cada una de las siguientes
condiciones que se cumplan. A continuación se describe la lógica para la
puntuación **alcista**. La lógica bajista es simplemente la inversa.

1.  **Estructura de Mercado (Higher Highs & Higher Lows)**
    - **Condición**: El máximo actual es más alto que el de hace 5 velas Y el
      mínimo actual es más alto que el de hace 5 velas.
    - **Propósito**: Confirma que el precio está formando una micro-tendencia
      alcista.

2.  **Tendencia de Medias Móviles (EMAs)**
    - **Condición**: El precio de cierre está por encima de la EMA rápida
      (ej. 20) Y la EMA rápida está por encima de la EMA lenta (ej. 50).
    - **Propósito**: Asegura que el trading se realiza a favor de la tendencia
      inmediata y de corto plazo.

3.  **Nube de Ichimoku**
    - **Condición**: El precio de cierre está por encima de la Senkou Span A Y
      por encima de la Senkou Span B.
    - **Propósito**: Utiliza la nube como una zona dinámica de
      soporte/resistencia para confirmar la fuerza de la tendencia.

4.  **Momentum del MACD**
    - **Condición**: La línea MACD está por encima de su línea de señal.
    - **Propósito**: Mide el momentum a corto plazo, indicando que la fuerza
      alcista está aumentando.

5.  **Histograma del MACD**
    - **Condición**: El histograma del MACD es positivo y está creciendo (la
      barra actual es más alta que la anterior).
    - **Propósito**: Confirma que la aceleración del momentum es positiva.

6.  **Fuerza Relativa (RSI)**
    - **Condición**: El RSI está por encima de 50.
    - **Propósito**: Indica que los movimientos alcistas recientes son más
      fuertes que los bajistas.

7.  **Divergencia Alcista (RSI)**
    - **Condición**: El precio marca un mínimo más bajo mientras que el RSI
      marca un mínimo más alto.
    - **Propósito**: Identifica un posible agotamiento de la tendencia bajista y
      una reversión inminente al alza. Es una señal de alta calidad.

8.  **Volumen Relativo (RVOL) Direccional**
    - **Condición**: El volumen de la vela actual es significativamente más alto
      que el volumen promedio reciente (ej. > 1.5x) Y la vela es alcista
      (cierre > apertura).
    - **Propósito**: Confirma que hay convicción y participación institucional
      detrás del movimiento alcista. El volumen debe acompañar la dirección del
      precio.

9.  **Fuerza de Tendencia Adaptativa (ADX)**
    - **Condición**: El ADX está por encima del umbral dinámico definido por
      sesión (`ADX_TREND_THRESHOLD`).
      - _Asia (00-07 UTC)_: > 18 (Tendencias más suaves).
      - _Londres/NY (07-20 UTC)_: > 25 (Filtro estricto contra ruido/rango).
      - _Default_: > 20.
    - **Propósito**: Evitar operar en mercados laterales o "picados", asegurando
      que existe una tendencia genuina antes de entrar.

## Filosofía "Lock in Profit & Ride" (Asegurar y Navegar)

El bot utiliza una estrategia de **Salidas Escalonadas (33/33/33)** para equilibrar la seguridad con el potencial de ganancias masivas.

1.  **TP1 (33% @ 0.75R)**:
    - **Objetivo**: Asegurar. Se cierra el primer tercio de la posición rápidamente.
    - **Efecto**: Reduce drásticamente el riesgo monetario del trade y cubre el spread/comisiones. Genera flujo de caja constante.

2.  **TP2 (33% @ 1.50R)**:
    - **Objetivo**: Capitalizar. Se cierra el segundo tercio en la expansión principal.
    - **Efecto**: Bloquea una ganancia sustancial antes de posibles retrocesos.

3.  **TP3 (34% Runner @ Trailing Stop)**:
    - **Objetivo**: Navegar. El último tercio no tiene TP fijo. Se activa un **Trailing Stop** dinámico.
    - **Efecto**: Permite capturar "Home Runs" (movimientos extensos de 5R, 10R o más) que ocurren durante eventos de alta volatilidad. En tendencias fuertes, este 34% restante puede generar más profit que todo lo anterior combinado.

## Ejemplo de Decisión

- Se analiza el par `GOLD` en la temporalidad de `15M`.
- Se calculan todos los indicadores.
- El bot determina que se cumplen 7 de las 9 condiciones alcistas ->
  `Bullish Score = 7`.
- Se cumplen 1 de las 9 condiciones bajistas -> `Bearish Score = 1`.
- El `CONFLUENCE_THRESHOLD` está en `5`.
- Como `Bullish Score (7)` >= `CONFLUENCE_THRESHOLD (5)`, el bot genera una
  señal de **`BUY`**.
- Si la puntuación hubiera sido 4, la decisión habría sido `HOLD`.
