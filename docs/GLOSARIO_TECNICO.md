# 📚 Glosario Técnico y Guía de Indicadores

Esta guía detallada explica los conceptos técnicos, indicadores y métricas utilizados por el `Smart Trading Bot`. Está diseñada para ser clara, didáctica y orientada a la práctica.

---

## 📈 1. Indicadores de Trading (src/indicators/indicators.py)

Estos son los "ojos" del bot. Le permiten interpretar el mercado y tomar decisiones objetivas.

### 🔹 ADX (Average Directional Index)
*   **¿Qué es?**: Un medidor de la **fuerza de la tendencia**, sin importar si es alcista o bajista.
*   **Rango**: 0 a 100.
*   **Interpretación**:
    *   **< 20**: Mercado en rango o tendencia débil (el bot suele evitar operar aquí).
    *   **> 25**: Tendencia fuerte establecida (Zona ideal para estrategias de tendencia).
    *   **> 50**: Tendencia extremadamente fuerte (Posible agotamiento).
*   **Uso en el Bot**: Se usa como filtro principal. Si el ADX es bajo, el bot asume que no hay fuerza para mover el precio y puede abstenerse de entrar.

### 🔹 RSI (Relative Strength Index)
*   **¿Qué es?**: Un oscilador que mide la velocidad y el cambio de los movimientos de precios.
*   **Rango**: 0 a 100.
*   **Interpretación**:
    *   **Sobrecompra (> 70)**: El precio ha subido mucho y muy rápido; podría corregir a la baja.
    *   **Sobreventa (< 30)**: El precio ha caído mucho y muy rápido; podría rebotar al alza.
    *   **Nivel 50**: Frontera entre tendencia alcista (>50) y bajista (<50).
*   **Uso en el Bot**: Se usa para confirmar entradas (no comprar en sobrecompra) y, crucialmente, para detectar **Divergencias**.

### 🔹 Divergencias (RSI vs Precio)
*   **¿Qué son?**: Cuando el precio y el RSI no están de acuerdo. Es una señal poderosa de reversión.
*   **Casos**:
    *   **Divergencia Alcista**: El precio hace un mínimo más bajo, pero el RSI hace un mínimo más alto. (Señal de COMPRA).
    *   **Divergencia Bajista**: El precio hace un máximo más alto, pero el RSI hace un máximo más bajo. (Señal de VENTA).
*   **Uso en el Bot**: Una de las señales de entrada más fuertes (High Confluence).

### 🔹 EMA (Exponential Moving Average)
*   **¿Qué es?**: Una línea que suaviza el precio, dando más peso a los datos recientes (a diferencia de la media simple).
*   **Uso en el Bot**:
    *   **Cruce de EMAs**: Usamos una rápida (ej. 9 periodos) y una lenta (ej. 21 periodos).
    *   Rápida cruza hacia arriba a Lenta = Tendencia Alcista.
    *   Rápida cruza hacia abajo a Lenta = Tendencia Bajista.

### 🔹 ATR (Average True Range)
*   **¿Qué es?**: Mide la **volatilidad** del mercado en pips/puntos. No dice la dirección, solo cuánto se mueve el precio.
*   **Uso en el Bot**: Vital para la gestión de riesgo.
    *   **Stop Loss Dinámico**: El SL no es fijo (ej. 50 pips), sino basado en el ATR (ej. 1.5 veces el ATR actual). Si el mercado está volátil, el SL se aleja; si está tranquilo, se acerca.

### 🔹 Order Blocks (Bloques de Órdenes)
*   **¿Qué son?**: Zonas de precio donde las grandes instituciones (bancos, fondos) han dejado órdenes pendientes. Actúan como imanes y zonas de rebote fuertes.
*   **Interpretación**:
    *   Si el precio vuelve a un Order Block alcista antiguo, es probable que rebote hacia arriba.
*   **Uso en el Bot**: Se detectan mediante fractales y se usan como zonas de alta probabilidad para Entradas y Take Profits.

### 🔹 VWAP (Volume Weighted Average Price)
*   **¿Qué es?**: El precio promedio ponderado por volumen. Es el "precio justo" del día según el dinero real negociado.
*   **Interpretación**:
    *   Precio por encima del VWAP = Sentimiento Alcista.
    *   Precio por debajo del VWAP = Sentimiento Bajista.
*   **Uso en el Bot**: Filtro de tendencia intradía. El bot prefiere comprar si está sobre el VWAP.

### 🔹 Ichimoku Cloud (Nube de Ichimoku)
*   **¿Qué es?**: Un sistema completo que muestra soporte, resistencia, tendencia y momentum en un solo gráfico.
*   **Uso en el Bot**: Principalmente la "Nube" (Kumo).
    *   Precio sobre la Nube = Tendencia Alcista fuerte.
    *   Precio bajo la Nube = Tendencia Bajista fuerte.
    *   Precio dentro de la Nube = Zona de ruido/incertidumbre.

---

## 📊 2. Conceptos Financieros y Métricas

Términos clave para entender los reportes de rendimiento.

### 💰 ROI (Return on Investment)
*   **Significado**: Retorno sobre la Inversión.
*   **Fórmula**: `(Ganancia Neta / Capital Inicial) * 100`
*   **Ejemplo**: Si empiezas con $500 y ganas $500, tu ROI es del 100%.

### 📉 Drawdown (DD)
*   **Significado**: La caída máxima desde un pico de capital hasta un valle. Mide el riesgo de ruina o el "dolor" que debes soportar.
*   **Ejemplo**: Si tu cuenta sube a $1000 y luego baja a $900 antes de volver a subir, tuviste un Drawdown del 10% ($100).
*   **Importante**: Un DD bajo es mejor. El Bot busca mantener el DD controlado gestionando el riesgo por operación (máx 3%).

### ⚖️ Breakeven (Punto de Equilibrio)
*   **Significado**: Mover el Stop Loss al precio de entrada para eliminar el riesgo.
*   **Uso**: "Poner la operación en Breakeven" significa que si el mercado se da la vuelta, saldrás con $0 de ganancia/pérdida (menos comisiones), protegiendo tu capital.

### 🎯 Win Rate (Tasa de Acierto)
*   **Significado**: El porcentaje de operaciones ganadoras sobre el total.
*   **Contexto**: Un Win Rate alto (70%+) es genial, pero debe ir acompañado de un buen ratio Riesgo/Beneficio.
*   **Fórmula**: `(Trades Ganadores / Total Trades) * 100`

### 🛡️ Trailing Stop Loss
*   **Significado**: Un Stop Loss que "persigue" al precio. Si el precio sube a tu favor, el SL sube automáticamente para asegurar ganancias.
*   **Beneficio**: Permite dejar correr las ganancias en tendencias fuertes sin riesgo de devolver todo lo ganado.

---

## 🚀 Cómo usar esta información
Al leer los logs del bot o los reportes:
1.  Busca la confluencia de indicadores (ej. "RSI Divergencia + EMA Cross").
2.  Observa el **ADX** para saber si es buen momento para tendencias.
3.  Vigila el **Drawdown** para asegurar que la gestión de riesgo funciona.
