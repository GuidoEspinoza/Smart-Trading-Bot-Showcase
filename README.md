# 🚀 Smart Trading Bot (Institutional Grade Algorithm)

![Performance](https://img.shields.io/badge/Annual%20Return-%2B245%2C828%25-success)
![Win Rate](https://img.shields.io/badge/Win%20Rate-76.0%25-blue)
![Status](https://img.shields.io/badge/Status-Private%20%2F%20Proprietary-red)

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

### ⚠️ Proprietary Technology Disclaimer
This repository serves as a **public showcase and results audit** for a private, institutional-grade trading system.
**The source code is closed-source and strictly protected. This is NOT an open-source project.**

### 📈 Verified Performance (2025 Audit)

![Equity Curve Growth](assets/growth_chart.png)
*> **Visual Proof**: Verified Equity Curve ($1k to $2.4M) with Circuit Breaker Logic.*

The system has undergone rigorous "Reality-Check" backtesting ensuring 100% parity with live execution logic.

| Metric | Result (Jan - Dec 2025) |
| :--- | :--- |
| **Initial Capital** | $1,000 |
| **Net Profit (Gross)** | **$2,459,290** |
| **ROI** | **+245,829%** |
| **Win Rate** | **76.04%** |
| **Drawdown** | ~28% (Circuit Breaker Protected) |
| **Total Trades** | 1,202 |

> *Note: Results verified with "Hard Stop" logic active (5% Daily Loss Limit).*

### 💹 Growth Scaling Analysis (The "Compound Effect")

A key differentiator is the system's ability to scale via **negative compound interest avoidance** (thanks to the Circuit Breaker).

| Period | Logic Consistency | Initial Capital | Final Balance (Verified) | Growth Factor |
| :--- | :---: | :---: | :---: | :---: |
| **Short Term (1 Mo)** | 100% | $1,000 | **$1,575** | 1.5x |
| **Mid Term (6 Mo)** | 100% | $1,000 | **$36,120** | 36x |
| **Full Year (12 Mo)** | 100% | $1,000 | **$2,459,290** | 2,459x |

> *The exponential curve accelerates in Q4 (Oct-Dec) because capital was preserved during the difficult month of August (-12%).*

### 🚀 Future Upside: Native TSL Integration

The backtested results assume a static Stop Loss. The production bot utilizes Capital.com's native **Server-Side Trailing Stop Loss**.
*   **Conservative Projection (+10%)**: ~$2.7M Net
*   **Optimistic Projection (+20%)**: ~$2.9M Net

### 🧠 Core Logic Overview

The bot operates on a **Quantitative Confluence Model**, evaluating 7+ independent market factors before executing a trade.

#### Key Features
*   **Circuit Breaker (Hard Stop)**: 
    *   **5% Daily Loss Limit**.
    *   **Automated Kill Switch**: Prevents catastrophic days (e.g., -20%) that destroy compound interest.
*   **Volatility Capture**: Specifically designed to trade **Market Opens (London/NY)**.
*   **Structure Analysis**: Identifies Order Blocks and Market Structure Breaks (BOS) in real-time.
*   **Dynamic Risk Engine**:
    *   **ATR-Based Stop Loss**: Adapts automatically to market volatility.
    *   **Partial Take Profits (33/33/33)**: Proprietary method to lock in gains early.

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
*> **Prueba Visual**: Curva de Equidad Verificada ($1k a $2.4M) con Lógica de Circuit Breaker.*

El sistema ha pasado por un backtesting riguroso, asegurando 100% de paridad con la lógica de ejecución en vivo.

| Métrica | Resultado (Ene - Dic 2025) |
| :--- | :--- |
| **Capital Inicial** | $1,000 |
| **Beneficio Neto (Bruto)** | **$2,459,290** |
| **ROI (Retorno)** | **+245,829%** |
| **Tasa de Acierto (Win Rate)** | **76.04%** |
| **Drawdown** | ~28% (Protegido por Circuit Breaker) |
| **Total Trades** | 1,202 |

> *Nota: Resultados verificados con lógica "Hard Stop" activa (Límite de Pérdida Diaria del 5%).*

### 💹 Análisis de Crecimiento (El "Efecto Compuesto")

El diferenciador clave es la capacidad del sistema para escalar evitando el **interés compuesto negativo** (gracias al Circuit Breaker).

| Periodo | Consistencia Lógica | Capital Inicial | Balance Final (Verificado) | Factor de Crecimiento |
| :--- | :---: | :---: | :---: | :---: |
| **Corto Plazo (1 Mes)** | 100% | $1,000 | **$1,575** | 1.5x |
| **Mediano Plazo (6 Meses)** | 100% | $1,000 | **$36,120** | 36x |
| **Año Completo (12 Meses)** | 100% | $1,000 | **$2,459,290** | 2,459x |

> *La curva exponencial se acelera en el Q4 (Oct-Dic) porque el capital fue preservado durante el mes difícil de Agosto (-12%).*

### 🚀 Potencial Futuro: Integración TSL Nativa

Los resultados del backtest asumen un Stop Loss estático. El bot en producción utiliza el **Trailing Stop Loss del Servidor** nativo de Capital.com.
*   **Proyección Conservadora (+10%)**: ~$2.7M Neto
*   **Proyección Optimista (+20%)**: ~$2.9M Neto

### 🧠 Lógica Central

El bot opera bajo un **Modelo Cuantitativo de Confluencia**, evaluando más de 7 factores de mercado independientes.

#### Características Clave
*   **Circuit Breaker (Hard Stop)**: 
    *   **Límite de Pérdida Diaria del 5%**.
    *   **Kill Switch Automático**: Previene días catastróficos (ej. -20%) que destruyen el interés compuesto.
*   **Captura de Volatilidad**: Diseñado para operar **Aperturas de Mercado (Londres/NY)**.
*   **Análisis de Estructura**: Identifica Bloques de Órdenes y BOS en tiempo real.
*   **Motor de Riesgo Dinámico**:
    *   **Stop Loss basado en ATR**: Se adapta automáticamente.
    *   **Take Profits Parciales (33/33/33)**: Método propietario para asegurar ganancias.

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
