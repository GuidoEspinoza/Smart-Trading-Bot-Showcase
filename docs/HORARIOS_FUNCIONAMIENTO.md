# 🕒 Horarios de Operación y Sesiones de Mercado

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

---

### Resumen de Rutina Diaria (Chile)

1.  **20:00 CLST (00:00 UTC):** Inicio del "Día Bot". Se resetea la meta de ganancias.
2.  **21:30 CLST:** Posible inicio sesión Asia (si activo).
3.  **03:00 CLST:** Apertura ventanas Forex (Londres).
4.  **10:30 CLST:** Apertura Índices USA.
5.  **17:00 CLST:** Cierre ventanas Metales/Forex.
6.  **17:55 CLST:** Cierre de Mercados USA y fin de sesión operativa.
