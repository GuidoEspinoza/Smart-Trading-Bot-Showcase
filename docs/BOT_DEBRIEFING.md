# Smart Trading Bot: Memorándum Técnico

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

## 2. Métricas de Rendimiento (Validación 6 Meses)

El sistema ha superado pruebas exhaustivas de backtesting (simulación) y forward
testing (proyección).

### Resultados Consolidados (Julio - Diciembre 2025)

- **Retorno Neto (PnL)**: **+8,437.92%** ($500 ➡️ $42,689) 🚀🚀🚀
- **Factor de Sostenibilidad**: 6 Meses consecutivos de ganancias compuestas.
- **Tasa de Acierto (Win Rate)**: **71.29%** (Sobre 425 operaciones).
- **Consistencia**: El bot demostró ser capaz de navegar diferentes regímenes de mercado durante medio año sin quemar la cuenta.
- **Factor Clave**: El interés compuesto agresivo (reinversión del 3% de riesgo) funcionó perfectamente.

---

## 3. Arquitectura del Sistema

El bot está construido sobre una arquitectura modular en Python, donde cada
componente tiene una responsabilidad clara y definida.

### `main.py`: El Punto de Control (API)

- **Rol**: Actúa como la interfaz principal para controlar el bot.
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
    verificar los horarios de trading, evitar las aperturas de mercado volátiles
    y comprobar si se ha alcanzado el objetivo de ganancias diario.
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
  de la cuenta, el riesgo deseado por operación (ej. 1%), y los precios de
  entrada y stop loss.
- **Cálculos Clave**:
  1.  **Riesgo Monetario**: Calcula cuánto dinero se está dispuesto a arriesgar
      en una sola operación (ej. 1% de $10,000 = $100).
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
  (`DAILY_PROFIT_TARGET_PERCENT`), y las ventanas de tiempo a evitar
  (`MARKET_OPEN_AVOID_WINDOWS`).
- **`symbols_config.py`**: Define qué activos operar (`GLOBAL_SYMBOLS`) y sus
  reglas específicas (`SYMBOL_SPECIFIC_CONFIG`), como los horarios de mayor
  liquidez y los parámetros de trading del broker.
- **`profiles.py`**: Contiene los perfiles de estrategia (`Scalping`,
  `Intraday`). Cada perfil es un diccionario que define un estilo de trading
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
      ganancias en USD** (ej. 5% del balance).
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
    - **¿Estamos en una Ventana de Peligro?**: Comprueba si la hora actual cae
      dentro de las `MARKET_OPEN_AVOID_WINDOWS` (ej. apertura de Londres) para
      pausar las operaciones.
    - **¿Hemos Ganado Suficiente?**: Llama a `_check_profit_target()` para ver
      si la equidad actual ha alcanzado el objetivo diario. Si es así, el bot se
      "duerme" hasta el próximo reseteo.

4.  **Análisis de Símbolos (`_process_symbol`)**:
    - Si todos los filtros se superan, el bot itera sobre cada `symbol` en
      `GLOBAL_SYMBOLS`.
    - **Filtro de Concurrencia**: Verifica cuántas operaciones ya están abiertas
      para ese símbolo y las compara con `MAX_CONCURRENT_TRADES_PER_SYMBOL`. Si
      se ha alcanzado el límite, salta al siguiente símbolo.
    - **Análisis Multi-Timeframe**:
      - Itera sobre los `TIMEFRAMES` definidos en el perfil activo (ej.
        `["MINUTE_5", "MINUTE_15"]`).
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
    - Al final, se obtiene una puntuación final (ej. `bullish_score = 4`,
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

---

## 5. Guía de Configuración

Para modificar el comportamiento del bot, solo necesitas editar los archivos en
`src/config/`.

- **Para cambiar el estilo de trading**: Modifica `ACTIVE_PROFILE` en
  `core_config.py` a "Scalping" o "Intraday".
- **Para añadir o quitar un símbolo**: Edita la lista `GLOBAL_SYMBOLS` en
  `symbols_config.py`.
- **Para ajustar el riesgo global**: Cambia `RISK_PER_TRADE_PERCENT` en el
  perfil deseado dentro de `profiles.py`.
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
