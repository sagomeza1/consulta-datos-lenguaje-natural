# AGENTS.md

## Proyecto y estado

Consulta de archivos tabulares mediante lenguaje natural y HTTP. Fase 0 pendiente de aprobación; no hay aplicación ni comandos de pruebas todavía.

## Contexto obligatorio

Leer [objetivos-proyecto.md](objetivos-proyecto.md), [especificaciones-proyecto.md](especificaciones-proyecto.md), [plan.md](plan.md) y [decisiones.md](decisiones.md). Seguir la [guía de trabajo agéntico](https://github.com/sagomeza1/guia-trabajo-agentico).

## Reglas

- No implementar hasta aprobar la fase 0 y el brief/plan de la porción.
- Base confirmada: Python, FastAPI, DuckDB; versiones y otras dependencias pendientes.
- Documentación y mensajes de usuario en español; nombres de código propuestos en inglés.
- Mantener el dominio independiente de HTTP, lectores y SDK de proveedores.
- No ejecutar código ni SQL libre del LLM; validar planes y resolver fuentes desde el catálogo.
- Fuentes de solo lectura. No enviar datos a servicios externos sin autorización explícita.
- No guardar claves, filas ni prompts en código, fixtures o logs.
- Mostrar fallos como errores explícitos; no convertirlos en resultados vacíos ni activar fallback remoto.
- No ampliar alcance ni modificar la guía de referencia como parte de este proyecto.

## Verificación y cierre

Documentación: revisar enlaces locales, coherencia de contratos y diff completo. Al crear código, fijar y ejecutar comandos reproducibles de estilo, tipado y pruebas; actualizar esta sección.

Cada porción requiere evidencia de su criterio, actualización del plan y decisiones, y explicación de límites. Dobles prueban el sistema, no la calidad del LLM. No declarar una evaluación real completada sin datos y métricas.
