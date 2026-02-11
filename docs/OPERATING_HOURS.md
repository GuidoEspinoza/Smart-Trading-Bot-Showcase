# 🕒 Operating Hours & Market Sessions

> **Choose Language / Elige Idioma**: [🇺🇸 English](#-english-version) | [🇪🇸 Español](#-versión-en-español)

---

## 🇺🇸 English Version

Centralized documentation of all **Smart-Trading-Bot** operating schedules.
All internal bot schedules run in **UTC (Coordinated Universal Time)** to avoid discrepancies.

This document includes conversion to **Chile Time** (Continental).

---

## 🌎 Quick Conversion Table

| Time Zone | Code | Diff from UTC |
| :--- | :---: | :---: |
| **UTC (Bot)** | UTC | +0 |
| **Chile Winter** | CLT (UTC-4) | -4 hours |
| **Chile Summer** | CLST (UTC-3) | -3 hours |

> **Note:** Chile DST usually runs from **first Saturday of September** to **first Saturday of April**.

---

## ⚙️ General Bot Schedules

These are "master switches" configured in `src/config/core_config.py`.

| Setting | UTC Value | Description |
| :--- | :---: | :--- |
| **Operating Cycle** | **00:00 - 24:00** | Bot is active 24h, but respects specific market closes. |
| **Daily Reset** | **00:00 UTC** | Resets counters and checks daily profit target. |
| **Friday Close** | **21:55 UTC** | **Forced Close** of all trades before weekend. |

---

## 🕒 Production Hours ("Volatility Capture")

The bot operates under a **Volatility Capture** configuration. This means:
1.  **Does NOT trade 24/7 indiscriminately.**
2.  Focuses on **maximum liquidity** and **volatility** windows (Opens).
3.  **Does not avoid** market opens, seeking to capitalize on strong initial moves.

### Active Windows Table (UTC)

| Market | Operating Window (UTC) | Description |
| :--- | :---: | :--- |
| **Forex Major** | **06:00 - 20:00** | Covers London + Full New York. |
| **Metales (Gold/Silver)** | **07:00 - 20:00** | Max metal volatility. |
| **US Indices (US30/100)** | **13:00 - 22:00** | American Session (Includes 14:30 open). |
| **EU Indices (DAX/FTSE)** | **07:00 - 16:00** | European Session (Includes 08:00 open). |
| **Asia (J225/HK50)** | **00:00 - 08:00** | Asian Session (Full Session). |
| **Crypto** | **24/7** | No restrictions (except maintenance). |

> **Note:** These hours are hardcoded in `src/config/symbols_config.py` ensuring bot never trades outside safe zones.

---

## 📅 Operating Days

*   **Monday to Thursday:** Normal Operation.
*   **Friday:** Total close at **21:55 UTC**.
*   **Saturday & Sunday:** Operation **DISABLED** (Crypto also stops if `TRADE_ON_WEEKENDS = False`).

---

# 🕒 Horarios de Operación y Sesiones de Mercado (Español)

## 🇪🇸 Versión en Español

Documentación centralizada de todos los horarios operativos del **Smart-Trading-Bot**.
Todos los horarios internos del bot funcionan en **UTC (Tiempo Universal Coordinado)** para evitar discrepancias.

Este documento incluye la conversión a **Hora de Chile** (Continental).

---

## 🌎 Tabla de Conversión Rápida

| Zona Horaria | Código | Diferencia con UTC |
| :--- | :---: | :---: |
| **UTC (Bot)** | UTC | +0 |
| **Chile Invierno** | CLT (UTC-4) | -4 horas |
| **Chile Verano** | CLST (UTC-3) | -3 horas |

> **Nota:** El horario de verano en Chile generalmente va desde el **primer sábado de septiembre** hasta el **primer sábado de abril**.

---

## ⚙️ Horarios Generales del Bot

Estos son los "interruptores maestros" configurados en `src/config/core_config.py`.

| Configuración | Valor UTC | Chile Invierno (UTC-4) | Chile Verano (UTC-3) | Descripción |
| :--- | :---: | :---: | :---: | :--- |
| **Ciclo Operativo** | **00:00 - 24:00** | 20:00 - 20:00 (Día sig.) | 21:00 - 21:00 (Día sig.) | El bot está activo 24h, pero respeta cierres de mercado específicos. |
| **Reinicio Diario** | **00:00 UTC** | 20:00 hrs | 21:00 hrs | Se resetean contadores y se verifica la meta de ganancia diaria. |
| **Cierre Viernes** | **21:55 UTC** | 17:55 hrs | 18:55 hrs | **Cierre forzoso** de todas las operaciones antes del fin de semana. |

---

## 🕒 Horarios de Producción ("Volatility Capture")

El bot opera bajo una configuración de **Captura de Volatilidad**. Esto significa que:
1.  **NO opera 24/7 indiscriminadamente.**
2.  Se enfoca en las ventanas de **máxima liquidez** y **volatilidad** (Aperturas).
3.  **No evita** las aperturas de mercado, buscando capitalizar los movimientos fuertes iniciales.

### Tabla de Ventanas Activas (UTC)

| Mercado | Ventana Operativa (UTC) | Descripción |
| :--- | :---: | :--- |
| **Forex Major** | **06:00 - 20:00** | Cubre Londres + Nueva York completo. |
| **Metales (Gold/Silver)** | **07:00 - 20:00** | Máxima volatilidad de metales. |
| **Índices US (US30/100)** | **13:00 - 22:00** | Sesión Americana (Incluye Open 14:30). |
| **Índices EU (DAX/FTSE)** | **07:00 - 16:00** | Sesión Europea (Incluye Open 08:00). |
| **Asia (J225/HK50)** | **00:00 - 08:00** | Sesión Asiática (Full Session). |
| **Crypto** | **24/7** | Sin restricciones (salvo mantenimiento). |

> **Nota:** Estos horarios están "hardcodeados" en `src/config/symbols_config.py` para garantizar que el bot nunca opere fuera de estas zonas seguras.

---

## 📅 Días Operativos

*   **Lunes a Jueves:** Operativa Normal (respetando ventanas de bloqueo).
*   **Viernes:** Cierre total a las **21:55 UTC** (17:55 / 18:55 Chile).
*   **Sábado y Domingo:** Operativa **DESACTIVADA** (Crypto también se detiene si `TRADE_ON_WEEKENDS = False`).
