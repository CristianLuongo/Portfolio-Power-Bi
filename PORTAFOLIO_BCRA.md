# Observatorio de ahorro y financiamiento en Argentina

Proyecto: `BCRA.pbip`. Fuente: APIs públicas del Banco Central de la República Argentina.

## Páginas

1. **Resumen:** último dato y fecha de observación por indicador; inflación mensual; tasas; dólar y precios en base 100; crédito nominal y real.
2. **Ahorro y poder adquisitivo:** capital inicial seleccionable entre $100.000 y $10.000.000, inicio por cierre de mes y comparación histórica de depósitos, inflación, dólar mayorista, UVA y CER.
3. **Tasas y crédito:** cuatro tasas nominales anuales, diferencias en puntos porcentuales, crecimiento real interanual del crédito y rendimiento real mensual aproximado del depósito.
4. **Metodología y datos:** cobertura por serie, fechas, cantidad de registros, duplicados, meses sin inflación y actualización en hora argentina.

## Datos y actualización

Se importan las variables **5, 7, 11, 12, 13, 26, 27, 28, 30 y 31**. Tanto el catálogo como los históricos se consultan directamente en `https://api.bcra.gob.ar/estadisticas/v4.0/Monetarias`.

- `Periodo = 2026` determina el comienzo solicitado: 1 de enero de `Periodo − 5` (2021). Es un parámetro editable de Power Query.
- Las consultas usan una URL base constante, `RelativePath` y `Query`, con páginas de hasta 1.000 registros. Los datos incompletos por errores de API no se sustituyen silenciosamente por una tabla vacía.
- `BCRA_Datos` conserva las observaciones de origen. `Analisis Mensual` construye cierres y promedios por mes, excluyendo el mes en curso.
- El catálogo importado ya no depende de una ruta local al CSV. El CSV original queda conservado como referencia.
- Las funciones existentes de Central de Deudores, cheques y Ollama se conservan; no son fuentes de los visuales de este portafolio.
- Credenciales para las consultas BCRA: **Anónimo**, cuando Power BI las solicite. Para traer nueva información, usar **Actualizar** en Power BI Desktop. La programación en el servicio no fue configurada.

## Definiciones económicas

**Último dato:** observación más reciente de cada serie hasta la fecha de corte seleccionada. La fecha mostrada es la fecha de la observación, no la fecha de publicación. Una tasa mensual y una diaria pueden tener fechas diferentes sin que exista un error de carga.

**Tasas:** las series de origen vienen en porcentaje nominal anual y las medidas las convierten a proporciones decimales con formato porcentual. Los gráficos mensuales muestran el promedio de observaciones del mes. Las diferencias entre tasas se expresan en puntos porcentuales y no equivalen al margen contable de los bancos.

**Inflación:** la variable 27 es variación mensual del IPC; la 28 es variación interanual. El índice de precios reconstruido acumula `PRODUCTO(1 + inflación mensual)`, con base teórica 100 al cierre anterior al inicio de la historia. No es el nivel oficial del IPC. Un mes faltante interrumpe la construcción del índice y las comparaciones que dependen de él.

**Crédito real interanual:** `(1 + crecimiento nominal interanual) / (1 + inflación interanual) − 1`. Los saldos se toman al cierre de cada mes. La serie 26 incluye moneda local y extranjera expresadas en pesos: variaciones del tipo de cambio pueden modificar el saldo sin representar nuevos préstamos.

**Simulación de ahorro:** inicio al cierre del mes seleccionado; final en el último mes completo con información comparable. Sin selección de inicio se usan hasta los últimos 12 meses disponibles dentro del corte seleccionado. El capital inicial por defecto es $1.000.000.

Para cada mes posterior al inicio se usa la TNA de depósitos a 30 días observada al cierre del mes anterior:

`factor del mes = 1 + TNA decimal × días del mes / 365`

Se multiplican los factores mensuales y se reinvierten los intereses. Es una aproximación de renovación mensual, no una reconstrucción contractual de plazos exactos de 30 días. No incorpora aportes, retiros, impuestos, comisiones ni tasas futuras conocidas de antemano.

`capital real = capital nominal / factor de inflación acumulada`

Se expresa en pesos con poder adquisitivo del cierre inicial. UVA, CER y dólar mayorista se comparan por cocientes entre cierres; son referencias, no instrumentos disponibles para contratar ni simulaciones de rendimientos adicionales. Las curvas no prolongan datos a meses todavía no comparables.

## Cambios en medidas anteriores

- Se corrigieron las definiciones de último valor, inflación mensual/trimestral/acumulada, crecimiento real del crédito y simulación de ahorro.
- `Indice IPC` se conserva oculto como alias de inflación interanual para no romper referencias antiguas.
- `Acumulado Anual` compara contra el cierre de diciembre anterior; no repite el cálculo interanual.
- Las medias móviles se anclan a la observación seleccionada y no a una fecha fija de hoy.
- Las medidas de agregación genérica quedan ocultas y exigen una única variable.
- Los índices sintéticos sin metodología y las comparaciones de unidades incompatibles quedan ocultos en `99 · Retiradas`, devolviendo vacío. Sus fórmulas originales permanecen en el respaldo.

## Respaldo y comprobaciones

`_backups/BCRA_antes_portafolio.zip` conserva las definiciones originales, incluidos los scripts TMDL y recursos del reporte. Excluye la caché `.pbi` y la configuración local del usuario.

`tools/bcra/validation.json` contiene la validación de esquemas públicos PBIR, referencias a tablas/columnas/medidas y límites del lienzo. `tools/bcra/qa_data` contiene respuestas públicas empleadas para auditar la paginación; estas copias no alimentan el modelo.

La instancia BCRA de Power BI Desktop cargó las 10 tablas del modelo, 12.630 observaciones de origen y 68 meses de análisis. Las 114 medidas pasaron la compilación DAX. Se comprobaron ocho casos funcionales: integridad, inflación mensual, fecha histórica, capital sin tiempo transcurrido, inicio posterior al corte y proporcionalidad del capital. Los resultados monetarios se conciliaron además con un cálculo independiente sobre respuestas de la API. No se realizó una inspección visual mediante capturas de pantalla.

Los generadores `build_model.py` y `build_report.py` documentan esta implementación. **No ejecutarlos para actualizar datos:** reconstruyen las definiciones y pueden reemplazar ajustes posteriores. Para actualizar datos, utilizar Power BI Desktop.

Documentación de referencia:

- [Catálogo de APIs del BCRA](https://www.bcra.gob.ar/en/central-bank-api-catalog/)
- [Principales variables del BCRA](https://www.bcra.gob.ar/principales-variables/)
- [Formato de proyecto y reporte PBIR de Microsoft](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report)
