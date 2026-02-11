# Trading Bot Architecture

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

The bot is designed with a modular, decoupled architecture, centered around a
decision core (`Bot`) that orchestrates all other components. The workflow follows
a clear lifecycle for each market analysis cycle.

## Decision Flow Diagram

```mermaid
graph TD
    A[▶️ Start Analysis Cycle] --> B{Load Configuration};
    B --> C[For Each Symbol...];
    C --> D{Within Trading Hours?};
    D -- No --> C;
    D -- Yes --> E[Fetch Market Data];
    E --> F[Calculate Technical Indicators];
    F --> G[Calculate Confluence Scores];
    G --> H{Score >= Threshold?};
    H -- No (HOLD) --> C;
    H -- Yes (BUY/SELL) --> I[Calculate Risk Management];
    I --> J[Open Position via Exchange Client];
    J --> C;
```

## Core Components

1.  **Entry Point (`main.py`)**
    - Initializes the `FastAPI` web server to expose monitoring endpoints (e.g.,
      `/health`).
    - Creates and starts a unique main instance of the `Bot` in a background thread,
      which becomes the brain of the application.

2.  **Bot Core (`src/core/bot.py`)**
    - **`Bot` class**: The main class containing the analysis loop (`_run_logic_loop`).
    - **Configuration Loading**: On startup, loads all strategic configurations
      from `src/config/` and credentials from the `.env` file.
    - **Analysis Cycle**: Iterates over the list of symbols (`SYMBOLS`) defined in
      the configuration.
    - **Schedule Control**: Checks if the current time is within allowed trading
      windows for the symbol before proceeding. Includes a **Friday Close Window**
      (21:55 UTC) that halts entries and closes open positions to avoid weekend
      gaps (if `TRADE_ON_WEEKENDS=False`).
    - **Orchestration**: Calls indicator components and the trading client to
      fetch data and execute orders.
    - **Active Management (Parallel Thread)**: Runs an independent thread
      (`_run_monitor`) that monitors open positions every 1 second to apply
      partial close logic and trailing stops without blocking market analysis.

3.  **Configuration (`src/config/`)**
    - **`core_config.py`**: Contains central strategic configuration: symbol list,
      timeframes, confluence threshold (`CONFLUENCE_THRESHOLD`), risk per trade,
      and risk/reward ratio.
    - **`symbols_config.py`**: Defines symbol-specific parameters, such as optimal
      trading hours.
    - **`market_hours_config.py`**: Defines global market open and close times.

4.  **Indicators and Strategy (`src/indicators/`)**
    - **`add_all_indicators`**: Main function that receives a market DataFrame
      and appends all necessary technical indicator columns (EMAs, MACD, RSI,
      Ichimoku, etc.).
    - **`_calculate_confluence_scores`**: (Private method of `Bot`) Once
      indicators are calculated, this method evaluates them and generates a
      **bullish score** and a **bearish score** (from 0 to 8). This is the
      central piece of the decision logic.

5.  **Trading Client (`src/trading_client/`)**
    - Abstracts communication with different exchanges (Capital.com, Bybit).
    - Provides a unified interface with methods like `get_market_data`,
      `open_position`, `get_account_balance`, etc.
    - The `Bot` uses this client without needing to know which exchange it is
      connecting to, making the system extensible.

6.  **Utilities (`src/utils/`)**
    - **`risk_management.py`**: Contains crucial logic for
      `calculate_position_details`. This function takes the signal (BUY/SELL),
      account balance, and risk parameters to determine the **position size
      (volume)**, **Stop Loss** price (ATR-based), and **Take Profit** price.

## Detailed Execution Flow

1.  The `Bot` starts and loads its configuration.
2.  Enters an infinite loop running every few minutes.
3.  Inside the loop, iterates over each `symbol` in the list.
4.  Checks if it is a good time to trade that `symbol` according to
    `symbols_config.py`.
5.  If yes, requests market data for the main `timeframe` via the
    `trading_client`.
6.  Passes data to `add_all_indicators` to enrich the DataFrame.
7.  The `Bot` calculates confluence scores (bullish and bearish) from the
    enriched DataFrame.
8.  Compares scores with `CONFLUENCE_THRESHOLD`.
    - If neither score reaches the threshold, the decision is `HOLD` and moves to
      the next symbol.
    - If a score exceeds the threshold, a `BUY` or `SELL` signal is generated.
9.  With a valid signal, calls `calculate_position_details` to get exact trade
    parameters (volume, SL, TP).
10. Finally, uses the `trading_client` to send the `open_position` order to the
    exchange with all calculated details.
11. The cycle repeats.

---

## 🇪🇸 Versión en Español

# Arquitectura del Bot de Trading (Español)

El bot está diseñado con una arquitectura modular y desacoplada, centrada en un
núcleo de decisión (`Bot`) que orquesta el resto de los componentes. El flujo de
trabajo sigue un ciclo de vida claro para cada análisis de mercado.

## Diagrama de Flujo de Decisiones

```mermaid
graph TD
    A[▶️ Iniciar Ciclo de Análisis] --> B{Cargar Configuración};
    B --> C[Para cada Símbolo...];
    C --> D{¿Dentro de Horario de Trading?};
    D -- No --> C;
    D -- Sí --> E[Obtener Datos de Mercado];
    E --> F[Calcular Indicadores Técnicos];
    F --> G[Calcular Puntuaciones de Confluencia];
    G --> H{¿Puntuación >= Umbral?};
    H -- No (HOLD) --> C;
    H -- Sí (BUY/SELL) --> I[Calcular Gestión de Riesgo];
    I --> J[Abrir Posición Vía Cliente de Exchange];
    J --> C;
```

## Componentes Principales

1.  **Punto de Entrada (`main.py`)**
    - Inicia el servidor web `FastAPI` para exponer endpoints de monitoreo (ej.
      `/health`).
    - Crea e inicia una instancia única y principal del `Bot` en un hilo de
      fondo, que se convierte en el cerebro de la aplicación.

2.  **Núcleo del Bot (`src/core/bot.py`)**
    - **`Bot` class**: Es la clase principal que contiene el bucle de análisis
      (`_run_logic_loop`).
    - **Carga de Configuración**: Al iniciar, carga todas las configuraciones
      estratégicas desde `src/config/` y las credenciales desde el archivo
      `.env`.
    - **Ciclo de Análisis**: Itera sobre la lista de símbolos (`SYMBOLS`)
      definidos en la configuración.
    - **Control de Horarios**: Verifica si el momento actual está dentro de las
      franjas horarias de trading permitidas para el símbolo antes de proceder.
      Incluye una **Ventana de Cierre de Viernes** (21:55 UTC) que detiene
      nuevas entradas y cierra posiciones abiertas para evitar gaps de fin de
      semana (si `TRADE_ON_WEEKENDS=False`).
    - **Orquestación**: Llama a los componentes de indicadores y de cliente de
      trading para obtener datos y ejecutar órdenes.
    - **Gestión Activa (Hilo Paralelo)**: Ejecuta un hilo independiente
      (`_run_monitor`) que supervisa las posiciones abiertas cada 1 segundo
      para aplicar lógica de cierres parciales y trailing stops sin bloquear el
      análisis de mercado.

3.  **Configuración (`src/config/`)**
    - **`core_config.py`**: Contiene la configuración estratégica central: la
      lista de símbolos, las temporalidades, el umbral de confluencia
      (`CONFLUENCE_THRESHOLD`), el riesgo por operación y el ratio
      riesgo/beneficio.
    - **`symbols_config.py`**: Define parámetros específicos por símbolo, como
      los horarios de trading óptimos.
    - **`market_hours_config.py`**: Define los horarios de apertura y cierre de
      los mercados globales.

4.  **Indicadores y Estrategia (`src/indicators/`)**
    - **`add_all_indicators`**: Función principal que recibe un DataFrame de
      mercado y le añade todas las columnas de indicadores técnicos necesarios
      (EMAs, MACD, RSI, Ichimoku, etc.).
    - **`_calculate_confluence_scores`**: (Método privado del `Bot`) Una vez que
      los indicadores están calculados, este método los evalúa y genera una
      **puntuación alcista** y una **puntuación bajista** (de 0 a 8). Esta es la
      pieza central de la lógica de decisión.

5.  **Cliente de Trading (`src/trading_client/`)**
    - Abstrae la comunicación con los diferentes exchanges (Capital.com, Bybit).
    - Proporciona una interfaz unificada con métodos como `get_market_data`,
      `open_position`, `get_account_balance`, etc.
    - El `Bot` utiliza este cliente sin necesidad de saber a qué exchange se
      está conectando, lo que hace que el sistema sea extensible.

6.  **Utilidades (`src/utils/`)**
    - **`risk_management.py`**: Contiene la lógica crucial para
      `calculate_position_details`. Esta función toma la señal (BUY/SELL), el
      balance de la cuenta y los parámetros de riesgo para determinar el
      **tamaño de la posición (volumen)**, el precio de **Stop Loss** (basado en
      ATR) y el precio de **Take Profit**.

## Flujo de Ejecución Detallado

1.  El `Bot` se inicia y carga su configuración.
2.  Entra en un bucle infinito que se ejecuta cada pocos minutos.
3.  Dentro del bucle, itera sobre cada `símbolo` de la lista.
4.  Verifica si es un buen momento para operar ese `símbolo` según
    `symbols_config.py`.
5.  Si es así, solicita los datos de mercado para la `temporalidad` principal a
    través del `trading_client`.
6.  Pasa los datos a `add_all_indicators` para enriquecer el DataFrame.
7.  El `Bot` calcula las puntuaciones de confluencia (alcista y bajista) a
    partir del DataFrame enriquecido.
8.  Compara las puntuaciones con `CONFLUENCE_THRESHOLD`.
    - Si ninguna puntuación alcanza el umbral, la decisión es `HOLD` y pasa al
      siguiente símbolo.
    - Si una puntuación supera el umbral, se genera una señal de `BUY` o `SELL`.
9.  Con una señal válida, llama a `calculate_position_details` para obtener los
    parámetros exactos de la operación (volumen, SL, TP).
10. Finalmente, utiliza el `trading_client` para enviar la orden de
    `open_position` al exchange con todos los detalles calculados.
11. El ciclo se repite.
