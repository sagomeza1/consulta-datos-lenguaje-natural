# Plan de construcción

## Estado

Fase 0 en propuesta; no aprobada. Última porción cerrada: ninguna. Siguiente: resolver pendientes y aprobar contratos; después preparar el brief de la porción 1.

## Puerta de salida de fase 0

- [ ] Aprobar alcance inicial: una fuente por pregunta, `.xlsx`, operaciones y exclusiones.
- [ ] Aprobar contratos HTTP, política CSV y arquitectura con plan estructurado.
- [ ] Fijar Python, dependencias, gestor, sistema operativo y hardware.
- [ ] Recibir catálogo y preguntas representativas sin información sensible.
- [ ] Aprobar acceso al portal, límites, retención y política de datos.
- [ ] Elegir modelo Ollama y segundo proveedor; aprobar su uso remoto y presupuesto.
- [ ] Fijar conjunto de evaluación, umbrales, p95 y margen de regresión.

## Porciones verticales propuestas

Cada porción requiere brief y plan aprobado antes de código. Si no cabe en una sesión, dividirla conservando un flujo ejecutable. Los controles de acceso y límites se incorporan desde la primera porción, no al final.

| Porción | Flujo observable y criterio | Dependencia |
| --- | --- | --- |
| 1 | Pregunta HTTP sobre CSV sintético con planificador doble → agregado exacto; planes no permitidos son rechazados. | Fase 0 aprobada |
| 2 | Pregunta HTTP con Ollama → selección del CSV autorizado y agregado correcto; indisponibilidad tiene error explícito. | 1 |
| 3 | Pregunta ambigua → solicitud de aclaración sin ejecutar una fuente arbitraria. | 2 |
| 4 | Pregunta sobre Parquet → resultado equivalente al CSV de referencia. | 2 |
| 5 | Pregunta sobre una hoja XLSX → resultado esperado; otra hoja no se confunde y valores de fórmula ausentes fallan explícitamente. | 2 |
| 6 | Pregunta tabular HTTP → descarga CSV del mismo resultado, con expiración y autorización comprobadas. | 2 |
| 7 | Pregunta desde web mínima → respuesta/aclaración y tabla descargable, mostrando origen y truncamiento. | 3, 6 |
| 8 | Misma pregunta con segundo proveedor → flujo correcto sin modificar el dominio; modo local no realiza llamadas externas. | 2 y autorización remota |
| 9 | Portal mediante túnel SSH → pregunta y descarga autenticadas; caída del túnel produce un error visible. | 7 |
| 10 | Evaluación de versión candidata → métricas por segmento, incertidumbre, p95 y coste cumplen umbrales aprobados. | 3, 4, 5, 7, 8, 9 y conjunto real |

Revisar necesidad de refactor después de cada dos o tres porciones, con pruebas antes y después. No marcar una porción cerrada por una demostración manual aislada ni por respuestas de un doble.

## Evidencia por cierre

Registrar criterio, comando real y resultado, diff revisado y limitaciones. Actualizar este plan y [decisiones.md](decisiones.md). No inventar cobertura, comandos disponibles ni calidad de modelos.

## Deuda y bloqueos

No hay deuda de código: no se ha implementado. La evaluación real y la integración SSH están bloqueadas por falta de datos, modelos y entorno acordados.

## Fuera de alcance

Las exclusiones de [objetivos-proyecto.md](objetivos-proyecto.md) se aplican a todas las porciones. Ampliarlas requiere actualizar y aprobar la especificación antes de implementar.
