# consulta-datos-lenguaje-natural

Consulta de CSV, Parquet y Excel mediante lenguaje natural, con proveedores LLM intercambiables y API HTTP.

**Estado: propuesta de fase 0, pendiente de aprobación. No hay aplicación implementada.**

La construcción sigue la [guía de trabajo agéntico de sagomeza1](https://github.com/sagomeza1/guia-trabajo-agentico): contratos antes del código, porciones verticales y verificación con evidencia. Esta documentación es específica del proyecto; no modifica ni reproduce la guía.

## Documentos del proyecto

| Documento | Propósito |
| --- | --- |
| [objetivos-proyecto.md](objetivos-proyecto.md) | Necesidad, alcance y resultados esperados. |
| [especificaciones-proyecto.md](especificaciones-proyecto.md) | Arquitectura SOLID, contratos, pruebas e integración HTTP/SSH. |
| [plan.md](plan.md) | Porciones de construcción y puerta de aprobación. |
| [decisiones.md](decisiones.md) | Decisiones confirmadas, propuestas y pendientes. |
| [AGENTS.md](AGENTS.md) | Reglas para agentes y próximas sesiones. |

## Diseño propuesto

Pregunta web → API HTTP → catálogo autorizado → plan estructurado del LLM → validación → consulta determinista → respuesta verificable o tabla exportable.

Python, FastAPI y DuckDB son la base acordada. Ollama será la integración local inicial. Los servicios remotos se incorporarán mediante adaptadores, sin cambiar el dominio ni activar el envío de datos fuera del servidor por defecto.

No se ejecutará Python, SQL libre ni comandos producidos por el modelo. Las rutas de los archivos permanecerán bajo control del servidor.
