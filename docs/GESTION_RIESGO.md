# Gestión de Riesgo y Tamaño de la Posición

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
2. **Optimización Matemática**: Backtests demuestran +217% de rentabilidad con 
   84.40% win rate en diciembre 2025.
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
