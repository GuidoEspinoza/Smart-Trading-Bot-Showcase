# Risk Management & Position Sizing

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

A profitable trading strategy depends not only on good entry signals but on exceptional risk management. The bot incorporates a **dynamic, three-pillar risk management system** to protect capital and maximize gains.

All logic is encapsulated in the `calculate_position_details` function within `src/utils/risk_management.py`.

## The 6 Pillars of Risk Management

### 1. Fixed Risk Per Trade (`RISK_PER_TRADE_PERCENT`)

This parameter, defined in `src/config/core_config.py`, is the foundation of the entire system.

- **Definition**: The maximum percentage of total account capital willing to be lost in a single trade.
- **Example**: If account balance is $10,000 and `RISK_PER_TRADE_PERCENT` is `1.0`, max risk is $100.
- **Purpose**: Ensures no single trade causes catastrophic damage, allowing survival through losing streaks.

### 2. Dynamic Stop Loss (ATR Based)

Stop Loss is not fixed but adapts to current market volatility using the **Average True Range (ATR)**.

- **Calculation**:
  1.  Get current candle ATR.
  2.  Multiply by `ATR_MULTIPLIER` (configurable).
  3.  Subtract from entry (buy) or add to entry (sell).
- **Formula (Buy)**:
  `Stop Loss = Entry Price - (ATR * ATR_MULTIPLIER)`
- **Purpose**: In volatile markets, ATR increases, widening Stop Loss to avoid noise "wick-outs". In calm markets, ATR decreases, tightening Stop Loss to protect gains.

### 3. Take Profit (Risk/Reward Based)

Profit target is calculated directly from Stop Loss distance to maintain a constant **Risk/Reward Ratio** (`RISK_REWARD_RATIO`).

- **Calculation**:
  1.  Calculate distance in pips/points from Entry to Stop Loss (`Risk in Pips`).
  2.  Multiply by `RISK_REWARD_RATIO`.
  3.  Add/Subtract from Entry Price.
- **Formula (Buy)**:
  `Take Profit = Entry Price + (Risk in Pips * RISK_REWARD_RATIO)`
- **Purpose**: Ensures winning trades are, on average, significantly larger than losers. A `2.0` ratio means seeking $2 gain for every $1 risked.

### 4. Scaled Partial Close Strategy 33/33/33 (Optimized)

The bot implements a **scaled exit strategy** maximizing gains while progressively securing capital.

#### TP1: First Partial Close (33% @ 0.75:1 R/R)

- **Condition**: Price reaches 75% of initial risk distance.
- **Action**: Closes **33% of volume**.
- **Purpose**:
  - Secure early profit.
  - Reduce psychological risk.
  - Fund the "free ride" for remaining position.
- **Formula (Buy)**:
  `TP1 = Entry Price + (Risk in Pips * 0.75)`

#### TP2: Second Partial Close (33% @ 1.5:1 R/R)

- **Condition**: Price reaches 1.5x initial risk.
- **Action**: Closes another **33% of volume**.
- **Effect**:
  - Locks in substantial gain (66% of position closed in profit).
  - Mathematically impossible to lose money on the total trade from here.
  - Activates trailing stop for remaining 34%.
- **Formula (Buy)**:
  `TP2 = Entry Price + (Risk in Pips * 1.5)`

#### TP3: Trailing Stop (Remaining 34% - "Let Winners Run")

For the **remaining 34%**, the bot activates a dynamic trailing stop chasing the trend.

- **Activation**: Automatic after TP2 close.
- **Mechanism**: Stop Loss adjusts dynamically to lock in accumulated profit while allowing capture of extended moves.
- **Execution**: Updates Stop Level on broker (Capital.com).
- **Purpose**: Capture "Home Runs" (massive trends) that exponentially multiply PnL.

#### Advantages of 33/33/33 Strategy

1. **Progressive Securing**: Three profit-taking levels gradually reduce risk.
2. **Mathematical Optimization**: Backtests show **exponential growth potential ($7.78M)** under strict volume limits and with a **70.86%** win rate (2025).
3. **Psychological Balance**: Combines safety (early closes) with big win potential (trailing stop).
4. **Size Management**: Tracks original size to ensure correct percentages at each close.

### 5. Minimum Size Validation

Before each partial close, validates that both closing size and remaining size are >= symbol's `min_deal_size`.

- **Condition**: If position too small to split.
- **Action**: Closes entire position to secure profit.
- **Purpose**: Prevents execution errors due to invalid sizes.

### 6. The Circuit Breaker (Daily Hard Stop)

The **Circuit Breaker** is the system's ultimate fail-safe, designed to preserve capital during anomalous or manipulated market days.

- **Config**: **5% Daily Loss Limit**.
- **Mechanism**:
    1. Monitors real-time equity drawdown.
    2. If daily loss hits 5%, executes emergency protocol.
    3. **Action**: Closes ALL positions immediately and pauses trading until the next session.
- **Performance Impact**:
    - Prevents "bad days" from becoming catastrophic (e.g., -20%).
    - Preserves base capital for compound interest to work effectively the next day.
    - Verified to enable the exponential curve by cutting loss tails in difficult months (e.g. August).

## Position Sizing Calculation (Volume)

Knowing dollar risk and Stop Loss distance in pips, the bot calculates exact position size.

- **Simplified Formula**:
  `Position Size = Dollar Risk / (Stop Loss Distance in Pips * Pip Value)`
- **Process**:
  1.  **Dollar Risk**: `Account Balance * (RISK_PER_TRADE_PERCENT / 100)`
  2.  **Stop Loss Distance**: `abs(Entry Price - Stop Loss Price)`
  3.  Calculates volume needed so if price hits Stop Loss, loss is exactly defined `Dollar Risk`.

This integrated approach ensures every trade opens with a complete, predefined risk plan, eliminating emotional decision making and maintaining long-term discipline.

---

# Gestión de Riesgo y Tamaño de la Posición (Español)

## 🇪🇸 Versión en Español

Una estrategia de trading rentable no solo depende de buenas señales de entrada,
sino de una gestión de riesgo excepcional. El bot incorpora un sistema de
gestión de riesgo dinámico y de tres pilares para proteger el capital y
maximizar las ganancias.

Toda la lógica se encuentra encapsulada en la función
`calculate_position_details` dentro de `src/utils/risk_management.py`.

## Los 6 Pilares de la Gestión de Riesgo

### 1. Riesgo Fijo por Operación (`RISK_PER_TRADE_PERCENT`)

Este parámetro, definido en `src/config/core_config.py`, es la base de todo el
sistema.

- **Definición**: Es el porcentaje máximo del capital total de la cuenta que se
  está dispuesto a perder en una sola operación.
- **Ejemplo**: Si el balance de la cuenta es de $10,000 y
  `RISK_PER_TRADE_PERCENT` es `1.0`, el riesgo máximo por operación será de
  $100.
- **Propósito**: Asegura que ninguna operación individual pueda causar un daño
  catastrófico a la cuenta, permitiendo sobrevivir a una racha de pérdidas.

### 2. Stop Loss Dinámico (Basado en ATR)

El Stop Loss no es un valor fijo, sino que se adapta a la volatilidad actual del
mercado utilizando el indicador **Average True Range (ATR)**.

- **Cálculo**:
  1.  Se obtiene el valor del ATR de la vela actual.
  2.  Se multiplica este valor por un multiplicador (`ATR_MULTIPLIER`,
      configurable).
  3.  El resultado se resta del precio de entrada para una compra, o se suma
      para una venta.
- **Fórmula (para una compra)**:
  `Stop Loss = Precio de Entrada - (ATR * ATR_MULTIPLIER)`
- **Propósito**: En mercados volátiles, el ATR será mayor, colocando el Stop
  Loss más lejos para evitar ser "barrido" por el ruido del mercado. En mercados
  tranquilos, el ATR será menor, ajustando el Stop Loss más cerca para proteger
  las ganancias.

### 3. Take Profit (Basado en Ratio Riesgo/Beneficio)

El objetivo de ganancia se calcula directamente a partir del Stop Loss para
mantener una relación riesgo/beneficio (`RISK_REWARD_RATIO`) constante.

- **Cálculo**:
  1.  Se calcula la distancia en pips/puntos desde el precio de entrada hasta el
      Stop Loss (`Riesgo en Pips`).
  2.  Se multiplica esta distancia por el `RISK_REWARD_RATIO`.
  3.  El resultado se suma al precio de entrada para una compra, o se resta para
      una venta.
- **Fórmula (para una compra)**:
  `Take Profit = Precio de Entrada + (Riesgo en Pips * RISK_REWARD_RATIO)`
- **Propósito**: Garantiza que las operaciones ganadoras sean, en promedio,
  significativamente más grandes que las perdedoras. Un ratio de `2.0` significa
  que por cada $1 arriesgado, se busca ganar $2.

### 4. Estrategia de Cierres Parciales 33/33/33 (Optimizada)

El bot implementa una estrategia de **cierres parciales escalonados** que maximiza 
las ganancias mientras asegura capital progresivamente.

#### TP1: Primer Cierre Parcial (33% en 0.75:1 R/R)

- **Condición**: Cuando el precio alcanza el 75% del riesgo inicial (0.75:1 Ratio).
- **Acción**: Cierra el **33% del volumen** de la posición original.
- **Propósito**: 
  - Asegura ganancia temprana en el bolsillo.
  - Reduce el riesgo psicológico de la operación.
  - Permite que el resto de la posición capture movimientos mayores.
- **Cálculo (para una compra)**:
  ```
  TP1 = Precio de Entrada + (Riesgo en Pips * 0.75)
  ```

#### TP2: Segundo Cierre Parcial (33% en 1.5:1 R/R)

- **Condición**: Cuando el precio alcanza 1.5 veces el riesgo inicial (1.5:1 Ratio).
- **Acción**: Cierra otro **33% del volumen** de la posición original.
- **Efecto**:
  - Asegura ganancia sustancial (66% de la posición cerrada con profit).
  - Matemáticamente, es imposible perder dinero en la operación total desde este punto.
  - Activa el trailing stop para el 34% restante.
- **Cálculo (para una compra)**:
  ```
  TP2 = Precio de Entrada + (Riesgo en Pips * 1.5)
  ```

#### TP3: Trailing Stop (34% restante - "Let Winners Run")

Para el **34% restante** de la posición (el "Runner"), el bot activa un trailing 
stop dinámico que persigue la tendencia.

- **Activación**: Automática después de cerrar TP2.
- **Mecanismo**: El Stop Loss se ajusta dinámicamente para proteger ganancias 
  acumuladas mientras permite que la posición capture tendencias extendidas.
- **Ejecución**: El bot actualiza el Stop Level en el broker (Capital.com) para 
  proteger las ganancias.
- **Propósito**: Capturar "Home Runs" (tendencias masivas) que multiplican el 
  PnL exponencialmente.

#### Ventajas de la Estrategia 33/33/33

1. **Aseguramiento Progresivo**: Tres niveles de toma de ganancias reducen el 
   riesgo gradualmente.
2. **Optimización Matemática**: Backtests demuestran **crecimiento exponencial ($7.78 Millones)** respetando límites de `max_deal_size`, con un **70.86%** de win rate en 2025.
3. **Balance Psicológico**: Combina seguridad (cierres tempranos) con potencial 
   de ganancias grandes (trailing stop).
4. **Gestión de Tamaño**: Tracking del tamaño original asegura porcentajes 
   correctos en cada cierre.

### 5. Validación de Tamaño Mínimo

Antes de cada cierre parcial, el bot valida que tanto el tamaño a cerrar como el 
restante sean mayores o iguales al `min_deal_size` del símbolo.

- **Condición**: Si la posición es demasiado pequeña para dividir.
- **Acción**: Cierra la posición completa para asegurar la ganancia.
- **Propósito**: Evita errores de ejecución por tamaños de posición inválidos.

### 6. Circuit Breaker (Cortafuegos Diario)

El **Circuit Breaker** (Hard Stop) es la "válvula de seguridad" final del sistema, diseñada para preservar el capital ante días de mercado anómalos o manipulados.

- **Configuración**: Límite de Pérdida Diaria del **5%** (`DAILY_LOSS_LIMIT_PERCENT`).
- **Mecanismo**: 
    1. El bot monitorea el `drawdown` de la equidad en tiempo real.
    2. Si la pérdida acumulada del día alcanza el 5%, el sistema entra en estado de emergencia.
    3. **Acción**: 
        - Cierra todas las posiciones abiertas inmediatamente (Hard Close).
        - Se auto-desactiva y pausa cualquier nueva operación hasta el siguiente día de trading.
- **Impacto en Rendimiento**:
    - Evita que un "mal día" se convierta en una catástrofe (ej. -20%).
    - Preserva el capital base para que el interés compuesto funcione al día siguiente.
    - Transformó el rendimiento anual 2025 asegurando la supervivencia de la cuenta y la explosión compuesta posterior.

## El Cálculo del Tamaño de la Posición (Volumen)

Una vez que se conocen el riesgo en dólares y la distancia del Stop Loss en
pips, el bot puede calcular el tamaño exacto de la posición.

- **Fórmula Simplificada**:
  `Tamaño de la Posición = Riesgo en Dólares / (Distancia al Stop Loss en Pips * Valor del Pip)`
- **Proceso**:
  1.  **Riesgo en Dólares**:
      `Balance de la Cuenta * (RISK_PER_TRADE_PERCENT / 100)`
  2.  **Distancia al Stop Loss**: `abs(Precio de Entrada - Precio de Stop Loss)`
  3.  El bot calcula el volumen necesario para que, si el precio toca el Stop
      Loss, la pérdida sea exactamente el `Riesgo en Dólares` definido.

Este enfoque integrado asegura que cada operación se abre con un plan de riesgo
completo y predefinido, eliminando la toma de decisiones emocional y manteniendo
la disciplina a largo plazo.
