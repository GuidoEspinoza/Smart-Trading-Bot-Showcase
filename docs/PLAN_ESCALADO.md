# Plan de Escalado y Crecimiento de Capital 🚀

Este documento define la **Hoja de Ruta Oficial** para la gestión del riesgo y el crecimiento de la cuenta.
La estrategia se basa en un enfoque de "Booster" inicial para capitalizar cuentas pequeñas, seguido de una transición suave hacia la preservación de capital institucional.

---

## 🗺️ Mapa de Ruta: El "Risk Glidepath"

| Fase | Rango de Capital | Riesgo por Trade | Objetivo Diario | Foco Principal | Mentalidad |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **1. Despegue (Booster)** | **$500 - $10,000** | **5.0%** | **10%** | **Crecimiento Exponencial.** Salir de la zona de capital bajo rápidamente. | *"Acepto la volatilidad para construir mi base."* |
| **2. Transición** | **$10,000 - $50,000** | **2.5%** | **5%** | **Consolidación.** Proteger los primeros $10k ganados. | *"Ya tengo algo que perder. Calma."* |
| **3. Crucero (Profesional)** | **> $50,000** | **1.0%** | **2-3%** | **Preservación de Riqueza.** Generar ingresos pasivos constantes. | *"Protejo el imperio. El interés compuesto hace el resto."* |

---

## 📍 Detalle de Fases

### Fase 1: Despegue (Booster) 🚀
*   **Capital:** $500 - $10,000
*   **Configuración Actual:**
    *   `RISK_PER_TRADE_PERCENT`: **5%**
    *   `DAILY_PROFIT_TARGET_PERCENT`: **10%**
    *   `ACTIVE_PROFILE`: "Intraday-Trend"
    *   `AVOID_MARKET_OPEN_HOURS`: **False** (Captura de Volatilidad Activada)
*   **Gestión:**
    *   **Retiros:** Mínimos o nulos. Reinvertir todo para maximizar el interés compuesto.
    *   **Stop Loss Diario:** Si pierdes 12% en un día, APAGA el bot por 24h.

### Fase 2: Transición 🛡️
*   **Capital:** $10,000 - $50,000
*   **Ajustes:**
    *   Reducir Riesgo al **2.5%**.
    *   Reducir Meta Diaria al **5%**.
*   **Gestión:**
    *   **Retiros:** Retira el **20%** de las ganancias mensuales. "Págate a ti mismo".
    *   **Activos:** Empieza a filtrar activos con spreads altos o baja liquidez.

### Fase 3: Crucero (Modo Institucional) 🛳️
*   **Capital:** > $50,000
*   **Ajustes:**
    *   Reducir Riesgo al **1.0%**.
    *   Reducir Meta Diaria al **2-3%**.
*   **Gestión:**
    *   **Retiros:** Retira el **50%** de las ganancias mensuales.
    *   **Psicología:** Aquí una pérdida del 5% serían -$2,500. Bajar al 1% reduce la pérdida a -$500, manteniendo la estabilidad emocional.

---

## 💡 ¿Por qué bajar al 1%? (La Matemática de la Ruina)

Mantener un riesgo del 5% con $100,000 es suicida:
*   **Riesgo 5% en $100k:** Una racha de 4 pérdidas = **-$20,000**. (Pánico total).
*   **Riesgo 1% en $100k:** Una racha de 4 pérdidas = **-$4,000**. (Manejable, es solo un mal día).

> **Aprobación:** Esta metodología alinea la agresividad matemática del bot con la psicología humana necesaria para sostener el éxito a largo plazo.
