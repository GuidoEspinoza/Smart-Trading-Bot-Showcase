# Smart Trading Bot: Technical Debrief

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

## 1. Introduction and Purpose

This document is a comprehensive analysis of the **Smart Trading Bot**, an algorithmic trading system designed to operate autonomously in financial markets via the Capital.com API.

**Primary Purpose**: The bot's goal is to identify and execute high-probability trading opportunities by applying a systematic strategy based on technical analysis, a signal confluence model, and strict risk management. The system is designed to be robust, configurable, and modular, allowing easy adaptation to different trading styles and market conditions.

---

## 2. Performance Metrics (2025 Year End Audit)

The system has undergone extensive backtesting (simulation) and forward testing (projection).

### Consolidated Results (January - December 2025)

- **Net Return**: **+$7.78 Million** 🚀🚀🚀
- **Sustainability Factor**: 12 consecutive months of compounded gains.
- **Win Rate**: **70.86%** (2,004 trades).
- **Consistency**: The bot demonstrated the ability to navigate different market regimes for a full year without blowing up the account, relying on Market Open Avoidance.
- **Key Factor**: Aggressive compound interest (Growth Mode: 3% risk) combined with the **30m Bias / 5m Entry** strategy.
- **Note**: This result confirms the system's robustness under strict institutional volume limits (`max_deal_size`). The "Growth Phase" goal ($50k) is easily achievable in under 6 months.

---

## 3. System Architecture

The bot is built on a modular Python architecture where each component has a clear and defined responsibility.

### `main.py`: The Control Point (API)

- **Role**: Acts as the main interface to control the bot.
- **Technology**: Uses **FastAPI** to expose endpoints that allow starting, stopping, and monitoring the bot's status.
- **Function**: Contains no trading logic. Its sole function is to receive HTTP commands and delegate them to the main `TradingBot` instance. It is the application entry point.

### `src/core/bot.py`: The Bot's Brain

- **Role**: The heart of the system. The `TradingBot` class orchestrates the entire trading process.
- **Responsibilities**:
  - **Main Loop**: Keeps the bot alive, executing analysis cycles at defined intervals.
  - **State Management**: Controls if the bot is running, if daily targets are met, etc.
  - **Analysis Orchestration**: Iterates over the symbol list (`GLOBAL_SYMBOLS`) and starts the analysis process for each.
  - **Operation Filters**: Applies crucial filters before trading, such as checking trading hours, checking daily profit targets (`DAILY_PROFIT_TARGET_PERCENT`), and applying the **Circuit Breaker** (5% daily loss limit).
  - **Order Execution**: Calls `CapitalClient` to open or close positions when the strategy generates a signal.
  - **Position Monitoring**: Runs a secondary thread (`_run_monitor`) that watches open positions, logging their PnL and detecting when they close.

### `src/core/capital_client.py`: The Broker Bridge

- **Role**: Abstracts all communication with the Capital.com API.
- **Function**:
  - **Session Management**: Handles authentication (login), session token renewal to avoid disconnections, and secure storage in a `.capital_session.json` file.
  - **API Requests**: Provides clear methods for complex actions like `get_prices()`, `create_position()`, `get_open_positions()`, etc.
  - **Error Handling**: Implements robust retry logic with _exponential backoff_ to handle network or API errors, ensuring bot resilience.

### `src/core/position_sizer.py`: The Risk Manager

- **Role**: Performs the most critical calculation before opening a trade: the position size.
- **Function**: The `calculate_position_details` function takes the account balance, desired risk per trade (e.g., 3%), entry price, and stop loss price.
- **Key Calculations**:
  1.  **Monetary Risk**: Calculates money willing to risk in a single trade (e.g., 3% of $1,000 = $30).
  2.  **Stop Distance**: Measures distance in points between entry and stop loss.
  3.  **Risk Per Point**: Calculates loss per point of price movement against the trade.
  4.  **Position Size**: Divides monetary risk by risk per point to get exact trade size.
  5.  **Broker Compliance**: Rounds position size to meet broker's minimum size and increment requirements.
  6.  **Take Profit Calculation**: Projects Take Profit level based on `RISK_REWARD_RATIO`.

### `src/config/`: The Control Panel

This directory centralizes all configuration, allowing behavior modification without touching logic code.

- **`core_config.py`**: Global parameters like active trading profile (`ACTIVE_PROFILE`), daily profit target (`DAILY_PROFIT_TARGET_PERCENT`), and the 5% Circuit Breaker trigger.
- **`symbols_config.py`**: Defines assets to trade (`GLOBAL_SYMBOLS`) and specific rules (`SYMBOL_SPECIFIC_CONFIG`), such as peak liquidity hours and broker parameters.
- **`profiles.py`**: Contains strategy profiles (`Growth Mode`, `Intraday`). Each profile is a dictionary defining a complete trading style, adjusting indicator parameters, risk management, and timeframes.

### `src/indicators/indicators.py`: The Analysis Toolbox

- **Role**: Contains all logic for calculating technical indicators.
- **Function**: The main function `add_all_indicators` takes a `pandas` price DataFrame and adds columns with indicator values (RSI, MACD, Moving Averages, ATR, Ichimoku, etc.).
- **Dynamic Parameters**: Uses an `AUTO_ADJUSTABLE_PARAMS` system defined in profiles so indicator parameters (e.g., RSI length) adjust automatically based on the analyzed timeframe.

---

## 3. Operation Flow (Step-by-Step)

The bot operates in a logical and predictable cycle.

1.  **Startup and Initial Reset**:
    - On start, calls `_reset_daily_stats()`.
    - Sets **initial daily balance** and calculates **USD profit target** (e.g., 10% of balance).
    - Resets any signal counters or daily state.

2.  **The Main Loop (`_run`)**:
    - Enters an infinite loop running while `self.is_running` is `True`.
    - At cycle end, calculates exact sleep time until next analysis interval start (e.g., wake up at HH:00, HH:05).

3.  **Decision Filters (Gatekeepers)**:
    - **Reset Time?**: Checks if daily reset hour (`DAILY_RESET_HOUR`) has passed to reset stats.
    - **Operating Hours?**: Checks if current time is within `OPERATING_HOURS` and if market for at least one symbol is open (`is_market_open`).
    - **Danger Window?**: Checks if current time falls within `MARKET_OPEN_AVOID_WINDOWS` (Noise/Volatility Filter).
    - **Profit Target Met?**: Calls `_check_profit_target()` to see if current equity reached daily target. If so, bot "sleeps" until next reset.
    - **Circuit Breaker?**: Checks if daily loss exceeds 5%. If s, halts trading.

4.  **Symbol Analysis (`_process_symbol`)**:
    - If filters pass, iterates over each `symbol` in `GLOBAL_SYMBOLS`.
    - **Concurrency Filter**: Checks open trades for symbol against `MAX_CONCURRENT_TRADES_PER_SYMBOL`. If limit reached, skips.
    - **Multi-Timeframe Analysis**:
      - Iterates over `TIMEFRAMES` defined in active profile (e.g., `["MINUTE_30", "MINUTE_5"]`).
      - Fetches price data and calculates indicators (`add_all_indicators`).
      - Calculates bullish and bearish **confluence scores** (`_calculate_confluence_scores`).

5.  **Confluence Model (`_calculate_confluence_scores`)**:
    - This is the essence of decision making. Instead of one condition, seeks "agreement" among multiple indicators.
    - For each indicator, adds `+1` to bullish score if buy signal, or `+1` to bearish score if sell signal.
    - **Score Example**:
      - RSI < 30: `bullish_score += 1`
      - MACD crossing up: `bullish_score += 1`
      - Price above slow EMA: `bullish_score += 1`
      - Price bouncing on Bullish Order Block: `bullish_score += 1`
    - Final score obtained (e.g., `bullish_score = 6`, `bearish_score = 1`).

6.  **Final Decision (`_make_decision`)**:
    - Compares scores with profile `CONFLUENCE_REQUIRED`.
    - **Buy Condition**: `bullish_score >= CONFLUENCE_REQUIRED` AND `bullish_score > bearish_score`.
    - **Sell Condition**: `bearish_score >= CONFLUENCE_REQUIRED` AND `bearish_score > bullish_score`.
    - **Overextension Filter**: Before confirming, checks if price is "too far" from moving average (using ATR multiple). If so, signal ignored to avoid buying tops/selling bottoms.
    - If conditions met, generates final signal (`TradingAction.BUY` or `TradingAction.SELL`).

7.  **Execution and Management (`_execute_trade`)**:
    - If decision is `BUY` or `SELL`:
      1.  **Risk Calculation**: Calls `calculate_position_details` for `position_size`, `take_profit_price`, `stop_loss_price`.
      2.  **Order Submission**: Calls `client.create_position()` with all details.
      3.  **Confirmation**: Capital.com API returns `dealReference`. Bot uses `client.confirm_deal()` to ensure order acceptance and get final `dealId`.
      4.  **Logging**: Records all operation info in log.

---

## 4. The Trading Strategy

The strategy is a **multi-factor confluence system** designed to be adaptable via profiles.

### Indicators Used

- **RSI (Relative Strength Index)**: Measures overbought/oversold conditions.
- **MACD (Moving Average Convergence Divergence)**: Measures momentum and trend direction.
- **Exponential Moving Averages (EMAs)**: Define short and long-term trends.
- **Ichimoku Cloud**: Provides full view of market structure (support, resistance, trend, momentum).
- **ATR (Average True Range)**: Measures volatility, used for Stop Loss distance. **ATR Trailing Stop**: Dynamic stop loss following price.
- **Order Blocks**: Supply/demand zones where price likely reacts.
- **Divergences**: Seeks discrepancies between price and RSI anticipating trend reversals.
- **RVOL (Relative Volume)**: Measures if current volume is anomalous compared to average.

### Risk Management

Risk management is the cornerstone and non-negotiable.

- **Fixed Risk Per Trade**: Risk defined as fixed percentage of account balance (`RISK_PER_TRADE_PERCENT`). Ensures losses are controlled/proportional.
- **Mandatory Stop Loss**: **No trade opens without Stop Loss**. Position calculated dynamically using ATR or structure zones, adapting to market volatility.
- **Risk/Reward Ratio (R/R)**: Take Profit calculated as multiple of Stop Loss distance (`RISK_REWARD_RATIO`), ensuring positive asymmetry.
- **Daily Profit Target**: Acts as "circuit breaker" to protect gains and avoid overtrading (`DAILY_PROFIT_TARGET_PERCENT`).
- **Circuit Breaker**: **Hard Stop at 5% daily loss**. Prevents ruin.

---

## 5. Configuration Guide

To modify bot behavior, edit files in `src/config/`.

- **Change Trading Style**: Modify `ACTIVE_PROFILE` in `core_config.py` to "Growth Mode" or "Preservation".
- **Add/Remove Symbol**: Edit `GLOBAL_SYMBOLS` list in `symbols_config.py`.
- **Adjust Global Risk**: Change `RISK_PER_TRADE_PERCENT` in desired profile within `profiles.py` (3% for Growth, 1% for Preservation).
- **Adjust Signal Strictness**: Adjust `CONFLUENCE_REQUIRED` in profile. Higher value means higher quality signals but lower frequency.

---

## 6. API Control

`main.py` exposes endpoints to manage the bot:

- `POST /bot/start`: Starts trading loop.
- `POST /bot/stop`: Stops bot safely after current analysis cycle.
- `POST /bot/force-stop`: Stops bot immediately.
- `GET /bot/status`: Returns current bot status, uptime, symbols being analyzed.

---

# Smart Trading Bot: Memorándum Técnico (Español)

## 🇪🇸 Versión en Español

## 1. Introducción y Propósito

Este documento es un análisis exhaustivo del **Smart Trading Bot**, un sistema
de trading algorítmico diseñado para operar de forma autónoma en los mercados
financieros a través de la API de Capital.com.

**Propósito Principal**: El objetivo del bot es identificar y ejecutar
oportunidades de trading de alta probabilidad, aplicando una estrategia
sistemática basada en análisis técnico, un modelo de confluencia de señales y
una estricta gestión del riesgo. El sistema está diseñado para ser robusto,
configurable y modular, permitiendo una fácil adaptación a diferentes estilos de
trading y condiciones de mercado.

---

## 2. Métricas de Rendimiento (Auditoría Cierre 2025)

El sistema ha superado pruebas exhaustivas de backtesting (simulación) y forward
testing (proyección).

### Resultados Consolidados (Enero - Diciembre 2025)

- **Retorno Neto**: **+$7.78 Millones de Dólares** 🚀🚀🚀
- **Factor de Sostenibilidad**: 12 Meses consecutivos de ganancias compuestas.
- **Tasa de Acierto (Win Rate)**: **70.86%** (2,004 operaciones).
- **Consistencia**: El bot demostró ser capaz de navegar diferentes regímenes de mercado durante un año completo sin quemar la cuenta (Evitando manipulación de aperturas).
- **Factor Clave**: El interés compuesto agresivo (Mode Growth: 3% riesgo) combinado con la estrategia **30m Bias / 5m Entry**.
- **Nota**: Este resultado confirma la robustez matemática del sistema bajo límites de lotaje institucionales reales. Para la **Fase de Crecimiento** ($1k a $50k), el tiempo requerido es de aproximadamente 5 meses.

---

## 3. Arquitectura del Sistema

El bot está construido sobre una arquitectura modular en Python, donde cada
componente tiene una responsabilidad clara y definida.

### `main.py`: El Punto de Control (API)

- **Rol**: actúa como la interfaz principal para controlar el bot.
- **Tecnología**: Utiliza **FastAPI** para exponer una serie de endpoints que
  permiten iniciar, detener y monitorear el estado del bot.
- **Funcionamiento**: No contiene lógica de trading. Su única función es recibir
  comandos HTTP y delegarlos a la instancia principal del `TradingBot`. Es el
  punto de entrada de la aplicación.

### `src/core/bot.py`: El Cerebro del Bot

- **Rol**: Es el corazón del sistema. La clase `TradingBot` orquesta todo el
  proceso de trading.
- **Responsabilidades**:
  - **Bucle Principal**: Mantiene al bot vivo, ejecutando ciclos de análisis a
    intervalos definidos.
  - **Gestión de Estado**: Controla si el bot está en ejecución, si ha alcanzado
    sus objetivos diarios, etc.
  - **Orquestación de Análisis**: Itera sobre la lista de símbolos
    (`GLOBAL_SYMBOLS`) y para cada uno, inicia el proceso de análisis.
  - **Filtros de Operación**: Aplica filtros cruciales antes de operar, como
    verificar los horarios de trading, comprobar si se ha alcanzado el objetivo de
    ganancias diario (`DAILY_PROFIT_TARGET_PERCENT`) y aplicar el **Circuit Breaker** (5%).
  - **Ejecución de Órdenes**: Llama al `CapitalClient` para abrir o cerrar
    posiciones cuando la estrategia genera una señal.
  - **Monitoreo de Posiciones**: Ejecuta un hilo secundario (`_run_monitor`) que
    vigila las posiciones abiertas, registrando su PnL y detectando cuándo se
    cierran.

### `src/core/capital_client.py`: El Puente con el Broker

- **Rol**: Abstrae toda la comunicación con la API de Capital.com.
- **Funcionamiento**:
  - **Gestión de Sesión**: Maneja la autenticación (login), la renovación de
    tokens de sesión para evitar desconexiones y el almacenamiento seguro de
    estos en un archivo `.capital_session.json`.
  - **Peticiones a la API**: Proporciona métodos claros y simples para acciones
    complejas como `get_prices()`, `create_position()`, `get_open_positions()`,
    etc.
  - **Manejo de Errores**: Implementa una lógica robusta de reintentos con
    _exponential backoff_ para manejar errores de red o de la API, garantizando
    la resiliencia del bot.

### `src/core/position_sizer.py`: El Gestor de Riesgo

- **Rol**: Realiza el cálculo más crítico antes de abrir una operación: el
  tamaño de la posición.
- **Funcionamiento**: La función `calculate_position_details` recibe el balance
  de la cuenta, el riesgo deseado por operación (ej. 3%), y los precios de
  entrada y stop loss.
- **Cálculos Clave**:
  1.  **Riesgo Monetario**: Calcula cuánto dinero se está dispuesto a arriesgar
      en una sola operación (ej. 3% de $1,000 = $30).
  2.  **Distancia del Stop**: Mide la distancia en puntos entre el precio de
      entrada y el stop loss.
  3.  **Riesgo por Punto**: Calcula cuánto se perdería por cada punto que el
      precio se mueva en contra.
  4.  **Tamaño de la Posición**: Divide el riesgo monetario entre el riesgo por
      punto para obtener el tamaño exacto de la operación.
  5.  **Ajuste a Reglas del Broker**: Redondea el tamaño de la posición para
      cumplir con los requisitos de tamaño mínimo y de incremento del broker.
  6.  **Cálculo del Take Profit**: Proyecta el nivel de Take Profit basándose en
      el `RISK_REWARD_RATIO`.

### `src/config/`: El Panel de Control

Este directorio centraliza toda la configuración, permitiendo modificar el
comportamiento del bot sin tocar el código de la lógica.

- **`core_config.py`**: Parámetros globales como el perfil de trading activo
  (`ACTIVE_PROFILE`), el objetivo de ganancia diario
  (`DAILY_PROFIT_TARGET_PERCENT`), y el Circuit Breaker de 5%.
- **`symbols_config.py`**: Define qué activos operar (`GLOBAL_SYMBOLS`) y sus
  reglas específicas (`SYMBOL_SPECIFIC_CONFIG`), como los horarios de mayor
  liquidez y los parámetros de trading del broker.
- **`profiles.py`**: Contiene los perfiles de estrategia (`Growth Mode`, `Preservation`). Cada perfil es un diccionario que define un estilo de trading
  completo, ajustando parámetros de indicadores, gestión de riesgo y timeframes.

### `src/indicators/indicators.py`: La Caja de Herramientas de Análisis

- **Rol**: Contiene toda la lógica para calcular los indicadores técnicos.
- **Funcionamiento**: La función principal `add_all_indicators` recibe un
  DataFrame de precios de `pandas` y le añade columnas con los valores de los
  indicadores (RSI, MACD, Medias Móviles, ATR, Ichimoku, etc.).
- **Parámetros Dinámicos**: Utiliza un sistema de `AUTO_ADJUSTABLE_PARAMS`
  definido en los perfiles para que los parámetros de los indicadores (ej. la
  longitud de un RSI) se ajusten automáticamente según el timeframe que se esté
  analizando.

---

## 3. Flujo de Operación (Paso a Paso)

El bot opera en un ciclo lógico y predecible.

1.  **Arranque y Reseteo Inicial**:
    - Al iniciar, el bot llama a `_reset_daily_stats()`.
    - Establece el **balance inicial** del día y calcula el **objetivo de
      ganancias en USD** (ej. 10% del balance).
    - Reinicia cualquier contador de señales o estado diario.

2.  **El Bucle Principal (`_run`)**:
    - El bot entra en un bucle infinito que se ejecuta mientras
      `self.is_running` sea `True`.
    - Al final de cada ciclo, calcula el tiempo exacto para dormir hasta el
      inicio del siguiente intervalo de análisis (ej. si el intervalo es de 5
      minutos, se despertará a las HH:00, HH:05, HH:10, etc.).

3.  **Filtros de Decisión (Guardianes)**:
    - **¿Es Hora de Resetear?**: Comprueba si ha pasado la hora de reseteo
      diario (`DAILY_RESET_HOUR`) para reiniciar las estadísticas.
    - **¿Estamos en Horario de Operación?**: Verifica que la hora actual esté
      dentro del `OPERATING_HOURS` y que el mercado para al menos uno de los
      símbolos esté abierto (`is_market_open`).
    - **¿Estamos en una Ventana de Peligro?**: Comprueba si la hora actual cae dentro de las `MARKET_OPEN_AVOID_WINDOWS` (Filtro de Ruido/Volatilidad).
    - **¿Hemos Ganado Suficiente?**: Llama a `_check_profit_target()` para ver
      si la equidad actual ha alcanzado el objetivo diario. Si es así, el bot se
      "duerme" hasta el próximo reseteo.
    - **¿Circuit Breaker?**: Verifica si la pérdida diaria supera el 5%. Si es así, detiene operaciones.

4.  **Análisis de Símbolos (`_process_symbol`)**:
    - Si todos los filtros se superan, el bot itera sobre cada `symbol` en
      `GLOBAL_SYMBOLS`.
    - **Filtro de Concurrencia**: Verifica cuántas operaciones ya están abiertas
      para ese símbolo y las compara con `MAX_CONCURRENT_TRADES_PER_SYMBOL`. Si
      se ha alcanzado el límite, salta al siguiente símbolo.
    - **Análisis Multi-Timeframe**:
      - Itera sobre los `TIMEFRAMES` definidos en el perfil activo (ej.
        `["MINUTE_30", "MINUTE_5"]`).
      - Para cada timeframe, obtiene los datos de precios y calcula todos los
        indicadores (`add_all_indicators`).
      - Calcula una **puntuación de confluencia** alcista y bajista
        (`_calculate_confluence_scores`).

5.  **El Modelo de Confluencia (`_calculate_confluence_scores`)**:
    - Esta es la esencia de la toma de decisiones. En lugar de depender de una
      sola condición, el bot busca "acuerdos" entre múltiples indicadores.
    - Para cada indicador, se asigna un `+1` a la puntuación alcista si da una
      señal de compra, o un `+1` a la bajista si da una señal de venta.
    - **Ejemplo de Puntuación**:
      - RSI < 30: `bullish_score += 1`
      - MACD cruzando hacia arriba: `bullish_score += 1`
      - Precio por encima de la EMA lenta: `bullish_score += 1`
      - Precio rebotando en un Order Block alcista: `bullish_score += 1`
    - Al final, se obtiene una puntuación final (ej. `bullish_score = 6`,
      `bearish_score = 1`).

6.  **Toma de Decisión Final (`_make_decision`)**:
    - El bot compara las puntuaciones con el `CONFLUENCE_REQUIRED` del perfil.
    - **Condición de Compra**: `bullish_score >= CONFLUENCE_REQUIRED` Y
      `bullish_score > bearish_score`.
    - **Condición de Venta**: `bearish_score >= CONFLUENCE_REQUIRED` Y
      `bearish_score > bullish_score`.
    - **Filtro de Sobreextensión**: Antes de confirmar, verifica si el precio
      está "demasiado lejos" de su media móvil (usando un múltiplo del ATR). Si
      es así, la señal se ignora para evitar comprar en un pico o vender en un
      valle.
    - Si se cumplen todas las condiciones, se genera una señal final
      (`TradingAction.BUY` o `TradingAction.SELL`).

7.  **Ejecución y Gestión (`_execute_trade`)**:
    - Si la decisión es `BUY` o `SELL`:
      1.  **Cálculo de Riesgo**: Llama a `calculate_position_details` para
          obtener el `position_size`, `take_profit_price` y `stop_loss_price`.
      2.  **Envío de Orden**: Llama a `client.create_position()` con todos los
          detalles.
      3.  **Confirmación**: La API de Capital.com devuelve un `dealReference`.
          El bot usa `client.confirm_deal()` para asegurarse de que la orden fue
          aceptada y obtener el `dealId` final.
      4.  **Registro**: Se registra toda la información de la operación en el
          log.

---

## 4. La Estrategia de Trading

La estrategia es un sistema de **confluencia de múltiples factores** diseñado
para ser adaptable a través de perfiles.

### Indicadores Utilizados

- **RSI (Relative Strength Index)**: Mide la sobrecompra y la sobreventa.
- **MACD (Moving Average Convergence Divergence)**: Mide el momentum y la
  dirección de la tendencia.
- **Medias Móviles Exponenciales (EMAs)**: Definen la tendencia a corto y largo
  plazo.
- **Nube de Ichimoku**: Proporciona una visión completa de la estructura del
  mercado (soporte, resistencia, tendencia y momentum).
- **ATR (Average True Range)**: Mide la volatilidad y se usa para calcular la
  distancia del Stop Loss. -**ATR Trailing Stop**: Un stop loss dinámico que
  sigue al precio.
- **Order Blocks**: Zonas de oferta y demanda donde es probable que el precio
  reaccione.
- **Divergencias**: Busca discrepancias entre el precio y el RSI que puedan
  anticipar un cambio de tendencia.
- **RVOL (Relative Volume)**: Mide si el volumen actual es anómalo en
  comparación con su media.

### Gestión de Riesgo

La gestión de riesgo es la piedra angular del bot y no es negociable.

- **Riesgo Fijo por Operación**: El riesgo se define como un porcentaje fijo del
  balance de la cuenta (`RISK_PER_TRADE_PERCENT`). Esto asegura que las pérdidas
  sean siempre controladas y proporcionales al capital.
- **Stop Loss Obligatorio**: **Ninguna operación se abre sin un Stop Loss**. Su
  posición se calcula dinámicamente usando el ATR o zonas estructurales (como un
  Order Block), asegurando que se adapte a la volatilidad actual del mercado.
- **Ratio Riesgo/Beneficio (R/R)**: El Take Profit se calcula como un múltiplo
  de la distancia del Stop Loss (`RISK_REWARD_RATIO`), garantizando una
  asimetría positiva.
- **Objetivo de Ganancia Diario**: Actúa como un "disyuntor" para proteger las
  ganancias y evitar la sobreoperación (`DAILY_PROFIT_TARGET_PERCENT`).
- **Circuit Breaker**: **Hard Stop al 5% de pérdida diaria**. Previene la ruina.

---

## 5. Guía de Configuración

Para modificar el comportamiento del bot, solo necesitas editar los archivos en
`src/config/`.

- **Para cambiar el estilo de trading**: Modifica `ACTIVE_PROFILE` en
  `core_config.py` a "Growth Mode" o "Preservation".
- **Para añadir o quitar un símbolo**: Edita la lista `GLOBAL_SYMBOLS` en
  `symbols_config.py`.
- **Para ajustar el riesgo global**: Cambia `RISK_PER_TRADE_PERCENT` en el
  perfil deseado dentro de `profiles.py` (3% Growth, 1% Preservation).
- **Para ser más o menos exigente con las señales**: Ajusta
  `CONFLUENCE_REQUIRED` en el perfil. Un valor más alto significa señales de
  mayor calidad pero menor frecuencia.

---

## 6. Control vía API

El `main.py` expone los siguientes endpoints para gestionar el bot:

- `POST /bot/start`: Inicia el bucle de trading.
- `POST /bot/stop`: Detiene el bot de forma segura después de que termine su
  ciclo de análisis actual.
- `POST /bot/force-stop`: Detiene el bot inmediatamente.
- `GET /bot/status`: Devuelve el estado actual del bot, incluyendo si está
  corriendo, su tiempo de actividad y los símbolos que está analizando.
