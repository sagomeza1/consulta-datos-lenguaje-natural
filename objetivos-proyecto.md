# Objetivos del proyecto

## Problema

Consultar información distribuida entre archivos requiere conocer dónde está cada dato y cómo filtrar o calcular sobre formatos diferentes. El usuario necesita preguntar desde un portal web sin escribir consultas técnicas.

## Objetivo general

Responder preguntas concretas sobre archivos tabulares autorizados mediante una interfaz HTTP, usando lenguaje natural para planificar y ejecución determinista para obtener los datos.

## Objetivos específicos

- Seleccionar la fuente y, para Excel, la hoja que corresponde a la pregunta.
- Consultar CSV, Parquet y tablas de Excel sin modificar los archivos originales.
- Entregar una respuesta concreta o una tabla según la intención de la pregunta.
- Exportar los resultados tabulares a CSV.
- Sustituir el proveedor LLM sin modificar las reglas de consulta.
- Permitir ejecución local con Ollama y habilitación explícita de proveedores con API KEY.
- Documentar la conexión de un portal por HTTP, incluidos túneles SSH.

## Alcance

### Confirmado

Repositorio de aplicación independiente y privado. Python, FastAPI y DuckDB. Archivos en una carpeta local del servidor, registrados en un catálogo autorizado. No se modifica el repositorio de la guía.

### Propuesta para la primera versión

Una fuente por pregunta; cada hoja tabular de Excel es una fuente independiente. Selección de columnas, filtros, orden, agregaciones y agrupación. Interfaz web mínima con pregunta, resultado, procedencia y descarga CSV. Uso local por un usuario o un portal autenticado, sin exposición pública directa.

Ollama como primer adaptador real; un segundo proveedor se elegirá para comprobar la sustitución. OpenAI, DeepSeek y Gemini son opciones de extensión, no integraciones ya disponibles ni necesariamente compatibles entre sí.

### Fuera de alcance propuesto

Uniones entre archivos, escritura sobre las fuentes, ejecución arbitraria de código, PDF o imágenes, entrenamiento de modelos, conversaciones con memoria, carga web de archivos y permisos multiusuario por fila. Archivos Excel con macros o formatos antiguos requieren ampliar el alcance; inicialmente se propone `.xlsx` con tablas rectangulares.

## Criterios de éxito

Los criterios siguientes son propuestos y deben aprobarse antes de construir:

| Resultado | Evidencia |
| --- | --- |
| La selección de fuente respeta el catálogo | Preguntas de prueba eligen el identificador esperado; una pregunta ambigua solicita aclaración. |
| Los tres formatos producen resultados correctos | El mismo conjunto de datos sintéticos en CSV, Parquet y XLSX devuelve valores equivalentes. |
| Ninguna cifra es inventada por el modelo | Respuestas concretas y tablas se obtienen del resultado del motor y conservan procedencia. |
| La descarga corresponde al resultado visible | CSV contiene las mismas filas y columnas; se comprueban nulos, comillas, Unicode y truncamiento. |
| Cambiar de proveedor no altera el dominio | Pruebas contractuales compartidas y evaluación del mismo conjunto con dos adaptadores reales. |
| El portal funciona por HTTP y por túnel SSH | Flujo de pregunta y descarga comprobado en ambos escenarios. |

La calidad de un modelo real no se considerará demostrada por pruebas con dobles. El conjunto de evaluación, hardware, umbrales de acierto, latencia y presupuesto siguen pendientes; véase [especificaciones-proyecto.md](especificaciones-proyecto.md).

## Restricciones

Modo local predeterminado. No enviar datos a proveedores remotos sin autorización explícita. Credenciales fuera del código y los prompts. Solo lectura de fuentes autorizadas. No declarar terminado el proyecto mientras haya criterios de aceptación sin evidencia.
