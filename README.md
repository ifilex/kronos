# Kronos

### Tendencia, compounding y piramidación

Kronos es un Expert Advisor escrito en **MQL4 para MetaTrader 4**. Este repositorio comparte el código fuente de la **versión 1.00** para estudio, revisión y experimentación.

Combina señales de tendencia y ruptura con dimensionamiento basado en equity, un multiplicador adicional de crecimiento y entradas sobre movimientos favorables.

**La configuración incluida es agresiva y activa por defecto la restricción a cuentas demo.**

## Contexto del proyecto

Como neuropsicólogo y desarrollador, me interesa explorar cómo se convierten observaciones y expectativas en reglas de decisión.

Kronos permite examinar una cuestión concreta: cómo cambia el comportamiento de un sistema cuando incrementa su exposición a partir del crecimiento de la cuenta y de movimientos favorables del precio.

Es un sistema basado en reglas. No incorpora aprendizaje automático ni un modelo neuropsicológico.

## Funcionamiento

### Señales de tendencia

Utiliza tres medias móviles exponenciales:

- EMA rápida de **21 períodos**.
- EMA lenta de **55 períodos**.
- EMA de tendencia de **200 períodos**.

La condición compradora requiere que la EMA rápida esté por encima de la lenta y que el precio esté por encima de la EMA de tendencia.

La condición vendedora utiliza las relaciones inversas.

Aunque algunos comentarios del código mencionan “cruces”, esta implementación evalúa la posición relativa actual de las medias. No comprueba que haya ocurrido un cruce entre dos velas.

### Rupturas de rango

Opcionalmente, puede generar una señal cuando el precio supera el máximo o cae por debajo del mínimo de las últimas **20 barras cerradas**, manteniendo la condición de tendencia.

La ruptura es una vía alternativa de entrada; no es un requisito adicional para todas las señales.

Los indicadores utilizan el período del gráfico y, en varios cálculos, la vela en formación.

### Entradas iniciales

Solo intenta abrir una entrada inicial cuando no existen órdenes del mismo símbolo con el número mágico del EA.

Una marca temporal compartida limita las entradas iniciales exitosas a una por barra del gráfico, entre todos los símbolos procesados.

### Stop loss y take profit

Las distancias iniciales se calculan con el ATR:

- Stop loss: **ATR × 2**.
- Take profit: **ATR × 8**.

Las operaciones añadidas por piramidación también reciben distancias calculadas con ATR.

Estas relaciones describen los parámetros del código; no acreditan rentabilidad ni una relación efectiva de resultados.

## Dimensionamiento y compounding

Con `UsarCompounding=true`, el lote inicial se estima a partir de:

1. Un porcentaje del equity actual.
2. La distancia al stop.
3. El valor y tamaño del tick informados por el bróker.
4. Un multiplicador adicional si el equity supera `EquityBaseReferencia`.

La fórmula del multiplicador es:

    MultiplicadorCrecimiento ^ (log(equity / referencia) / log(1.1))

Con el valor predeterminado de 1.15, cada incremento relativo del 10% sobre la referencia multiplica por 1.15 el lote ya calculado por riesgo, antes de aplicar los límites.

Por ello, `RiesgoPorTradePct` no representa un techo efectivo de riesgo: el multiplicador puede elevarlo y las pirámides agregan exposición.

Con `UsarCompounding=false`, las entradas iniciales utilizan el lote mínimo del instrumento.

## Piramidación

El sistema puede agregar operaciones cuando el precio se desplaza a favor de una posición.

Los parámetros predeterminados contemplan:

- Hasta **3 órdenes de pirámide abiertas por símbolo y dirección**.
- Distancia base configurada de **300 pips**.
- Lote añadido calculado como el **50% del lote de la orden evaluada**.

El recorrido evalúa tanto órdenes iniciales como órdenes añadidas. No constituye necesariamente una secuencia en la que cada nueva orden sea la mitad de la anterior.

El conteo depende de los comentarios de las órdenes que contienen `PIR`.

## Gestión de posiciones

Incluye:

- Trailing stop.
- Ajuste de trailing a partir de determinados beneficios.
- Movimiento del stop hacia el precio de entrada más un desplazamiento.
- Intento de cierre por umbrales de pérdida.
- Cierre opcional los viernes desde las 22:00 del servidor.

Mover un stop cerca de la entrada no garantiza un resultado neto sin pérdidas: intervienen costes y condiciones de ejecución.

## Kill switch

Cuando está activado, el sistema puede bloquear nuevas entradas y pirámides e intentar cerrar las posiciones de su número mágico.

### Umbral total

Compara el equity de toda la cuenta con el balance registrado al inicializar el EA.

No calcula drawdown desde el máximo histórico, aunque el código mantiene una variable con el máximo de equity observado.

### Umbral denominado “diario”

En esta versión también utiliza el balance inicial de la ejecución, no el balance al comienzo de cada día.

Por tanto, no debe interpretarse como una medición correcta de pérdida diaria.

### Alcance del cierre

El cierre se limita a las órdenes que coinciden con `MagicNumber`.

Los umbrales se calculan sobre toda la cuenta, por lo que otras operaciones, depósitos o retiros pueden afectar su activación.

El cierre depende de que las solicitudes sean aceptadas. Los umbrales no garantizan una pérdida máxima.

## Instrumentos

Contempla estos nombres base:

| Instrumento | Activado por defecto |
|---|---|
| BTCUSD | Sí |
| ETHUSD | Sí |
| SOLUSD | No |
| BNBUSD | No |
| XRPUSD | No |

Busca los símbolos con el sufijo detectado desde el gráfico y, como alternativa, sin sufijo.

La disponibilidad depende del bróker. No incluye conexión directa a exchanges ni gestión de billeteras.

## Parámetros principales

| Parámetro | Valor predeterminado | Función |
|---|---:|---|
| `MagicNumber` | `8888` | Identificador de órdenes |
| `AutoEjecutar` | `true` | Permite entradas iniciales |
| `UsarCompounding` | `true` | Activa dimensionamiento y multiplicador de crecimiento |
| `RiesgoPorTradePct` | `5.0` | Porcentaje base usado en el cálculo |
| `MultiplicadorCrecimiento` | `1.15` | Factor adicional de crecimiento |
| `EquityBaseReferencia` | `1000.0` | Referencia para el multiplicador |
| `LoteMaximoAbsoluto` | `1.0` | Tope previsto para entradas iniciales |
| `UsarPiramide` | `true` | Activa entradas adicionales favorables |
| `MaxPiramides` | `3` | Máximo de pirámides abiertas por símbolo y dirección |
| `DistanciaPiramidePips` | `300.0` | Distancia base para añadidos |
| `RatioLotePiramide` | `0.5` | Proporción del lote de la orden evaluada |
| `TrailingPips` | `200.0` | Distancia inicial del trailing |
| `TrailingAcelerado` | `100.0` | Distancia reducida del trailing |
| `BreakevenPips` | `150.0` | Umbral del ajuste hacia la entrada |
| `ActivarKillSwitch` | `true` | Activa las comprobaciones de pérdida y cierre de viernes |
| `MaxDrawdownTotalPct` | `40.0` | Umbral total de bloqueo y cierre |
| `PerdidaMaximaDiariaPct` | `15.0` | Umbral cuya referencia diaria requiere corrección |
| `MaxOrdenesTotales` | `20` | Límite comprobado para entradas iniciales |
| `MaxParesSimultaneos` | `2` | Límite previsto de símbolos |
| `SoloCuentaDemo` | `true` | Impide inicializar el EA fuera de una cuenta demo |

Los valores describen el archivo publicado. No son recomendaciones.

## Instalación para pruebas

1. Abrir MetaTrader 4 en una cuenta demo.
2. Seleccionar **Archivo → Abrir carpeta de datos**.
3. Copiar `KRONOS.mq4` dentro de `MQL4/Experts`.
4. Abrirlo en MetaEditor y compilar.
5. Revisar los errores y advertencias del compilador.
6. Actualizar la lista de Expert Advisors.
7. Cargar el EA y comprobar símbolos, históricos y parámetros.
8. Mantener `SoloCuentaDemo=true` durante la evaluación.

El procesamiento multiactivo depende de los ticks del gráfico donde está instalado.

Evitar compartir el número mágico con otros sistemas o instancias, ya que la gestión y los cierres se realizan utilizando ese identificador.

## Estado y limitaciones conocidas

Puntos identificados por lectura del código:

- **Pérdida diaria:** utiliza el balance inicial de la ejecución. El reinicio del contador no cambia esa referencia.
- **Persistencia:** reinicializar el EA restablece referencias, bloqueos y marca temporal de señales.
- **Cierres fallidos:** el kill switch no reintenta sistemáticamente el cierre mientras persiste el mismo motivo de bloqueo.
- **Pausa parcial:** `AutoEjecutar=false` detiene entradas iniciales, pero no la piramidación ni la gestión de posiciones.
- **Límite de órdenes:** no se comprueba dentro de la rutina de piramidación.
- **Límite de símbolos:** el conteo no se actualiza después de cada apertura dentro del recorrido.
- **Trailing:** la rama para beneficios superiores a 1000 pips es inalcanzable porque antes se evalúa una condición superior a 500.
- **Selección de órdenes:** el conteo de pirámides utiliza `OrderSelect` y no restaura la orden que estaba evaluando el recorrido exterior. Es necesario revisar qué operación queda seleccionada antes de leer sus datos.
- **Lotes de pirámide:** no aplican el mismo límite absoluto ni el ajuste completo al paso de lote de las entradas iniciales.
- **Lote mínimo:** puede prevalecer sobre el límite absoluto configurado si este es inferior al mínimo del bróker.
- **Unidades:** las conversiones entre ticks, precios y pips requieren validación para cada contrato.
- **Ejecución:** falta una comprobación integral de margen, restricciones de stops y resultados de algunas solicitudes.
- **Cierre de viernes:** no impide posteriores reaperturas y solo se ejecuta si el kill switch está activado.
- **Panel:** sumar lotes de instrumentos distintos no representa una medida homogénea de exposición monetaria o riesgo.

## Resultados y validación

Este repositorio no incluye un historial auditado ni una validación independiente de rendimiento.

La documentación se basa en la lectura del código. No acredita compilación, ejecución ni resultados de backtesting.

## Autor

Desarrollado por **Emanuel**, neuropsicólogo y desarrollador del proyecto **WineBOX**.

- https://winebox.cloud
- https://wineboxtool.cloud

## Contribuciones

Se reciben reportes de errores y propuestas mediante Issues y Pull Requests.

Incluir versión de MetaTrader 4, instrumento, período del gráfico, parámetros y mensajes relevantes del registro, sin credenciales ni datos personales.

## Licencia

Este proyecto se distribuye bajo la licencia **Apache License 2.0**.

Consultar el archivo [LICENSE](LICENSE) para conocer los términos completos.

## Aviso

Material técnico y educativo. La configuración agresiva puede generar pérdidas importantes. No constituye asesoramiento financiero ni una promesa de resultados.
