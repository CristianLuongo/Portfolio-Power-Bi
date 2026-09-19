# Observatorio de ahorro y financiamiento en Argentina

**Power BI · Power Query / M · DAX · APIs REST · TMDL · PBIP**

¿Cómo evolucionan el poder adquisitivo del ahorro, el costo del dinero y el crédito en Argentina?

Este proyecto de portafolio transforma datos públicos del **Banco Central de la República Argentina (BCRA)** en un tablero interactivo para explorar esas preguntas. Integra series económicas, compara valores nominales y reales y permite simular la evolución histórica de un capital en pesos.

**Autor:** Cristian Luongo

[Explorar el proyecto](Analisis%20BCRA/BCRA.pbip) · [Consultar la metodología](Analisis%20BCRA/PORTAFOLIO_BCRA.md) · [Ver el modelo semántico](Analisis%20BCRA/BCRA.SemanticModel/definition)

## Qué permite analizar

- **Ahorro y poder adquisitivo:** cuánto capital nominal y real habría resultado de una renovación mensual aproximada de depósitos.
- **Dólar e inflación:** cómo evolucionaron el tipo de cambio mayorista y los precios desde una misma base de comparación.
- **Costo del financiamiento:** cómo cambiaron las tasas de depósitos, BADLAR, préstamos interbancarios y adelantos en cuenta corriente.
- **Crédito privado:** cuánto del crecimiento nominal se mantiene después de descontar la inflación.
- **Ajuste por UVA y CER:** cómo habría variado un importe siguiendo cada referencia.
- **Calidad de datos:** qué cobertura tiene cada serie y cuál es la fecha de su última observación.

## Recorrido por el tablero

| Página | Contenido |
| --- | --- |
| **01 · Resumen** | Indicadores con fecha de observación, inflación mensual, tasas, dólar y precios en base 100, y crecimiento del crédito. |
| **02 · Ahorro y poder adquisitivo** | Simulación con capital e inicio seleccionables; comparación con inflación, dólar mayorista, UVA y CER. |
| **03 · Tasas y crédito** | Tasas nominales anuales, diferencias en puntos porcentuales y evolución del crédito nominal y real. |
| **04 · Metodología y datos** | Fuentes, cobertura, registros por serie, duplicados, meses sin IPC y fecha de actualización. |

### Una forma de explorarlo

En **Ahorro y poder adquisitivo**, seleccioná un capital inicial y un cierre de mes. El tablero compara el capital final nominal con su poder de compra y con el importe necesario para acompañar la inflación.

El capital puede variar entre **$100.000 y $10.000.000**, en pasos de $100.000. El valor predeterminado es $1.000.000 y, sin seleccionar un inicio, se utilizan hasta los últimos 12 meses disponibles dentro del corte de análisis.

## Desarrollo técnico

| Componente | Implementación |
| --- | --- |
| **Obtención de datos** | Consultas a la API REST del BCRA, lectura de JSON y paginación de históricos. |
| **Transformación** | Power Query / M para tipificación, controles de duplicados, cierres mensuales y promedios de tasas. |
| **Modelado** | Observaciones por variable y fecha, catálogo de indicadores, calendario y tabla de análisis mensual. |
| **Cálculos** | DAX para variaciones temporales, capitalización, ajuste por inflación y métricas de calidad. |
| **Interacción** | Parámetros de capital e inicio de simulación, filtros temporales y visuales nativos de Power BI. |
| **Organización del proyecto** | Definiciones del reporte en PBIR y del modelo semántico en TMDL, dentro de un proyecto PBIP. |

El caso muestra el proceso desde la ingesta hasta la presentación: integración de una fuente pública, preparación de series con distintas frecuencias, definición de medidas económicas y documentación de sus supuestos.

## Fuentes y alcance

Se consultan diez series de la API de estadísticas monetarias del BCRA:

| Variables | Identificadores de la API |
| --- | --- |
| Tipo de cambio mayorista | `5` |
| BADLAR privada, tasa interbancaria, depósitos a 30 días y adelantos | `7`, `11`, `12`, `13` |
| Préstamos al sector privado | `26` |
| Inflación mensual e interanual | `27`, `28` |
| CER y UVA | `30`, `31` |

La consulta solicita información desde enero de **2021** con la configuración actual. El inicio se define mediante el parámetro `Periodo`: se toma el 1 de enero de `Periodo − 5`. La cobertura efectiva depende de cada serie y puede consultarse en el tablero.

Las variables y sus unidades de referencia están disponibles en [Principales variables del BCRA](https://www.bcra.gob.ar/principales-variables/). El acceso se realiza mediante su [catálogo de APIs públicas](https://www.bcra.gob.ar/en/central-bank-api-catalog/).

## Criterios de cálculo

- **Tarjetas:** último dato disponible hasta la fecha de corte, acompañado por su fecha de observación.
- **Series mensuales:** promedio de tasas; último dato del mes para dólar, crédito, UVA y CER. Se excluye el mes en curso.
- **Inflación acumulada:** producto de los factores mensuales. El índice reconstruido sirve para comparar poder adquisitivo; no es el nivel oficial del IPC.
- **Crédito real interanual:** `(1 + crecimiento nominal) / (1 + inflación interanual) − 1`.
- **Simulación del depósito:** TNA al cierre del mes anterior, aplicada a los días del mes sobre base 365, con reinversión mensual de intereses.
- **Capital real:** capital nominal dividido por el factor de inflación acumulada, expresado en poder de compra del mes inicial.

La simulación es histórica y aproximada: no reproduce vencimientos contractuales exactos de 30 días ni incorpora aportes, retiros, comisiones o impuestos. El dólar mayorista, UVA y CER funcionan como referencias de evolución. La serie de crédito incluye moneda local y extranjera expresadas en pesos, por lo que también puede reflejar cambios de valuación cambiaria.

Los supuestos completos están en [la documentación metodológica](Analisis%20BCRA/PORTAFOLIO_BCRA.md).

## Cómo abrirlo

1. Descargá o cloná el repositorio completo, conservando su estructura de carpetas.
2. Utilizá una versión de **Power BI Desktop para Windows compatible con PBIP, PBIR y TMDL**.
3. Abrí [`Analisis BCRA/BCRA.pbip`](Analisis%20BCRA/BCRA.pbip).
4. Si Power BI solicita credenciales para `https://api.bcra.gob.ar`, seleccioná **Anónimo**.
5. Ejecutá **Actualizar** para importar las series y recalcular el modelo.
6. Recorré las cuatro páginas y probá los selectores de la simulación.

El archivo `.pbip` necesita las carpetas del reporte y del modelo; no debe descargarse de forma aislada. Microsoft documenta este formato y su apertura en [Power BI Desktop projects](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview).

La actualización requiere conexión a Internet y disponibilidad de la API. Los datos se cargan en modo **Importación**; la actualización programada en Power BI Service no forma parte de esta configuración.

## Estructura principal

```text
.
├── README.md
└── Analisis BCRA/
    ├── BCRA.pbip                   # Punto de entrada al proyecto
    ├── BCRA.Report/                # Páginas, visuales y configuración del reporte
    ├── BCRA.SemanticModel/         # Tablas, relaciones, consultas y medidas
    └── PORTAFOLIO_BCRA.md          # Definiciones y supuestos metodológicos
```

## Validación

Durante la implementación se validaron los esquemas y las referencias de los archivos PBIR, la compilación de **114 medidas DAX** en Power BI Desktop y ocho comprobaciones funcionales. Estas incluyeron controles de integridad, tratamiento de fechas y proporcionalidad del capital simulado.

Los resultados de capital nominal, capital real y capital equivalente al IPC se contrastaron con un cálculo independiente sobre respuestas de la API, con diferencias inferiores a **$0,01**. El detalle del alcance de estas comprobaciones está documentado en [PORTAFOLIO_BCRA.md](Analisis%20BCRA/PORTAFOLIO_BCRA.md).
