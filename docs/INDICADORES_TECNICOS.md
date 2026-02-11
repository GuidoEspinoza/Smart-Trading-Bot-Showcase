# Indicadores Técnicos Utilizados

El bot basa su estrategia de confluencia en un conjunto de indicadores técnicos
probados, cada uno aportando una perspectiva diferente sobre el estado del
mercado. Todos los indicadores se calculan utilizando la librería `pandas-ta`.

A continuación se detallan los indicadores que componen el sistema de puntuación
y la gestión de riesgo.

### 1. Medias Móviles Exponenciales (EMAs)

- **Definición**: Medias móviles que dan más peso a los precios recientes. Se
  utilizan una EMA rápida (ej. 20 periodos) y una lenta (ej. 50 periodos).
- **Rol en la Estrategia**: Determinar la dirección y fuerza de la tendencia a
  corto y medio plazo. La alineación del precio y las EMAs (precio > EMA
  rápida > EMA lenta) es una fuerte señal de tendencia alcista.

### 2. Nube de Ichimoku (Ichimoku Kinko Hyo)

- **Definición**: Un sistema de seguimiento de tendencia que define zonas de
  soporte y resistencia.
- **Rol en la Estrategia**: Confirmar la tendencia. Si el precio está por encima
  de la nube (Senkou Span A y B), se considera un entorno alcista. Si está por
  debajo, bajista.

### 3. Convergencia/Divergencia de Medias Móviles (MACD)

- **Definición**: Un indicador de momentum que muestra la relación entre dos
  EMAs. Consiste en la línea MACD, la línea de Señal y el Histograma.
- **Rol en la Estrategia**:
  - **Momentum**: El cruce de la línea MACD sobre la línea de Señal indica un
    aumento del momentum alcista.
  - **Aceleración**: El crecimiento del Histograma confirma que el momentum se
    está acelerando.

### 4. Índice de Fuerza Relativa (RSI)

- **Definición**: Un oscilador de momentum que mide la velocidad y el cambio de
  los movimientos de precios.
- **Rol en la Estrategia**:
  - **Fuerza General**: Un RSI por encima de 50 indica que la fuerza alcista
    predomina.
  - **Divergencias**: Una divergencia alcista (precio más bajo, RSI más alto) es
    una de las señales de reversión más potentes y tiene un gran peso en la
    puntuación.

### 5. Volumen Relativo (RVOL)

- **Definición**: Mide el volumen de la vela actual en comparación con el
  volumen promedio de un período anterior.
- **Rol en la Estrategia**: Validar la convicción detrás de un movimiento. Un
  volumen alto en una ruptura de tendencia confirma el interés y aumenta la
  probabilidad de que el movimiento continúe.

### 6. Estructura de Mercado (Pivotes)

- **Definición**: No es un indicador tradicional, sino un análisis de la acción
  del precio para identificar máximos más altos (Higher Highs) y mínimos más
  altos (Higher Lows).
- **Rol en la Estrategia**: Confirmar la formación de una micro-tendencia. Es la
  definición más básica de una tendencia alcista.

### 7. Rango Verdadero Promedio (ATR)

- **Definición**: Un indicador que mide la volatilidad del mercado.
- **Rol en la Estrategia**: No se usa para la puntuación de entrada, sino
  exclusivamente para la **gestión de riesgo**. Determina la distancia del Stop
  Loss dinámico, adaptándolo a las condiciones actuales del mercado.

### 8. Índice Direccional Promedio (ADX)

- **Definición**: Un indicador oscilador que varía de 0 a 100 para cuantificar
  la fuerza de una tendencia (independientemente de su dirección).
- **Rol en la Estrategia**: Actúa como un **filtro de régimen de mercado**.
  - Si ADX > Umbral, el mercado está en tendencia. **Operamos**.
  - Si ADX < Umbral, el mercado está lateral/rango. **No Operamos**.
- **Innovación**: El bot utiliza un **Umbral Adaptativo por Sesión** (más bajo
  en Asia, más alto en Londres/NY) para adaptarse al "ruido" característico de
  cada hora del día.
