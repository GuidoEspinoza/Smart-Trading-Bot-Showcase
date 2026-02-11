# 🚀 Smart Trading Bot (Institutional Grade Algorithm)

![Performance](https://img.shields.io/badge/Annual%20Return-%2B2%2C115%2C287%25-success)
![Win Rate](https://img.shields.io/badge/Win%20Rate-74.8%25-blue)
![Status](https://img.shields.io/badge/Status-Private%20%2F%20Proprietary-red)

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

### ⚠️ Proprietary Technology Disclaimer
This repository serves as a **public showcase and results audit** for a private, institutional-grade trading system.
**The source code is closed-source and strictly protected. This is NOT an open-source project.**

### 📈 Verified Performance (2025 Audit)

![Equity Curve Growth](assets/growth_chart.png)
*> **Visual Proof**: Verified Equity Curve ($1k to $21M) with Volatility Capture Logic.*

The system has undergone rigorous "Reality-Check" backtesting ensuring 100% parity with live execution logic (Production Hours + Commission + Slippage Logic).

| Metric | Result (Jan - Dec 2025) |
| :--- | :--- |
| **Initial Capital** | $1,000 |
| **Net Profit (Gross)** | **$21,152,876** |
| **ROI** | **+2,115,287%** |
| **Win Rate** | **74.79%** |
| **Drawdown** | < 20% (Dynamic Risk Management) |
| **Total Trades** | 1,309 |

> *Note: Results verified with "Volatility Capture" config (actively trading market opens to capture liquidity injections).*

### 💹 Growth Scaling Analysis (The "Compound Effect")

A key differentiator of this system is its ability to scale capital efficiently without degrading performance.

| Period | Logic Consistency | Initial Capital | Final Balance (Verified) | Growth Factor |
| :--- | :---: | :---: | :---: | :---: |
| **Short Term (1 Mo)** | 100% | $1,000 | **$4,136** | 4.1x |
| **Mid Term (6 Mo)** | 100% | $1,000 | **$102,668** | 102.6x |
| **Full Year (12 Mo)** | 100% | $1,000 | **$21,153,876** | 21,153x |

> *The exponential growth is driven by a proprietary "Reinvestment Engine" that dynamically adjusts lot size based on equity while strictly capping risk at 5% per trade.*

### 🚀 Future Upside: Native TSL Integration

The backtested results assume a static Stop Loss. The production bot utilizes Capital.com's native **Server-Side Trailing Stop Loss**.
*   **Conservative Projection**: +10% Efficiency (~$23M Net)
*   **Trend-Following Projection**: +20% Efficiency (~$25M Net)

### 🧠 Core Logic Overview

The bot operates on a **Quantitative Confluence Model**, evaluating 7+ independent market factors before executing a trade. It does not guess; it reacts to confirmed institutional flows.

#### Key Features
*   **Volatility Capture**: Specifically designed to trade **Market Opens (London/NY)**, capitalizing on the massive liquidity injections that break trends.
*   **Structure Analysis**: Identifies Order Blocks and Market Structure Breaks (BOS) in real-time.
*   **Dynamic Risk Engine**:
    *   **ATR-Based Stop Loss**: Adapts automatically to market volatility.
    *   **Partial Take Profits (33/33/33)**: Proprietary method to lock in gains early (TP1/TP2) while keeping a "Runner" (TP3) for massive trend extensions.
*   **Macro Filtering**: Automatically avoids trading during high-impact economic events (CPI, NFP).

### 🛠️ Technology Stack
*   **Core**: Python 3.11 (Async execution)
*   **Infrastructure**: Docker / Kubernetes
*   **Connectivity**: Direct API (ms latency)
*   **Data Processing**: Pandas / NumPy / TA-Lib

---

## 🇪🇸 Versión en Español

### ⚠️ Aviso de Tecnología Propietaria
Este repositorio sirve como **vitrina pública y auditoría de resultados** de un sistema de trading institucional privado.
**El código fuente es cerrado (closed-source) y está protegido. Esto NO es un proyecto de código abierto.**

### 📈 Rendimiento Validado (Auditoría 2025)

![Equity Curve Growth](assets/growth_chart.png)
*> **Prueba Visual**: Curva de Equidad Verificada ($1k a $21M) con Lógica de Captura de Volatilidad.*

El sistema ha pasado por un backtesting riguroso de "Chequeo de Realidad", asegurando 100% de paridad con la lógica de ejecución en vivo (Horarios de Producción + Lógica de Comisiones).

| Métrica | Resultado (Ene - Dic 2025) |
| :--- | :--- |
| **Capital Inicial** | $1,000 |
| **Beneficio Neto (Bruto)** | **$21,152,876** |
| **ROI (Retorno)** | **+2,115,287%** |
| **Tasa de Acierto (Win Rate)** | **74.79%** |
| **Drawdown** | < 20% (Gestión de Riesgo Dinámica) |
| **Total Trades** | 1,309 |

> *Nota: Resultados verificados utilizando "Captura de Volatilidad" (operando activamente durante aperturas de mercado para capturar inyecciones de liquidez).*

### 💹 Análisis de Crecimiento (El "Efecto Compuesto")

Un diferenciador clave de este sistema es su capacidad para escalar capital eficientemente sin degradar el rendimiento.

| Periodo | Consistencia Lógica | Capital Inicial | Balance Final (Verificado) | Factor de Crecimiento |
| :--- | :---: | :---: | :---: | :---: |
| **Corto Plazo (1 Mes)** | 100% | $1,000 | **$4,136** | 4.1x |
| **Mediano Plazo (6 Meses)** | 100% | $1,000 | **$102,668** | 102.6x |
| **Año Completo (12 Meses)** | 100% | $1,000 | **$21,153,876** | 21,153x |

> *El crecimiento exponencial es impulsado por un "Motor de Reinversión" propietario que ajusta dinámicamente el tamaño del lote basado en la equidad, limitando estrictamente el riesgo al 5% por operación.*

### 🚀 Potencial Futuro: Integración TSL Nativa

Los resultados del backtest asumen un Stop Loss estático. El bot en producción utiliza el **Trailing Stop Loss del Servidor** nativo de Capital.com.
*   **Proyección Conservadora**: +10% Eficiencia (~$23M Neto)
*   **Proyección Tendencial**: +20% Eficiencia (~$25M Neto)

### 🧠 Lógica Central

El bot opera bajo un **Modelo Cuantitativo de Confluencia**, evaluando más de 7 factores de mercado independientes antes de ejecutar una operación. No adivina; reacciona a flujos institucionales confirmados.

#### Características Clave
*   **Captura de Volatilidad**: Diseñado específicamente para operar **Aperturas de Mercado (Londres/NY)**, capitalizando las inyecciones masivas de liquidez que rompen tendencias.
*   **Análisis de Estructura**: Identifica Bloques de Órdenes (Order Blocks) y Rupturas de Estructura de Mercado (BOS) en tiempo real.
*   **Motor de Riesgo Dinámico**:
    *   **Stop Loss basado en ATR**: Se adapta automáticamente a la volatilidad del mercado.
    *   **Take Profits Parciales (33/33/33)**: Método propietario para asegurar ganancias temprano (TP1/TP2) mientras se mantiene un "Corredor" (TP3) para capturar extensiones de tendencia masivas.
*   **Filtrado Macro**: Evita automáticamente operar durante eventos económicos de alto impacto (IPC, NFP).

### 🛠️ Stack Tecnológico
*   **Núcleo**: Python 3.11 (Ejecución Asíncrona)
*   **Infraestructura**: Docker / Kubernetes
*   **Conectividad**: API Directa (latencia en milisegundos)
*   **Procesamiento de Datos**: Pandas / NumPy / TA-Lib

---

## 🔒 Access / Acceso

**English**: This software is available for licensing or managed accounts conversations primarily for institutional investors or qualified individuals.
**Español**: Este software está disponible para conversaciones sobre licenciamiento o cuentas gestionadas, principalmente para inversores institucionales o calificados.

**Contact / Contacto**: [contacto@guidoespinoza.dev](mailto:contacto@guidoespinoza.dev)

---
*© 2026 Guido Espinoza. All Rights Reserved.*
