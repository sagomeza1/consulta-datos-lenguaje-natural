# Registro de decisiones

## Confirmadas por el usuario el 2026-11-07

### Repositorio independiente, privado e inicializado con README

Motivo: construir la aplicación sin convertir el repositorio de la guía en un proyecto de código.
Alternativa descartada: modificar la guía o mantener la propuesta solo en la conversación.
Revisar si: el usuario decide trasladar el proyecto a otro repositorio.

### Python, FastAPI y DuckDB

Motivo: base elegida para API HTTP y consulta tabular determinista sobre varios formatos.
Alternativa descartada: dejar el stack pendiente.
Revisar si: las pruebas con volúmenes o restricciones reales invalidan esta elección.

### Carpeta local del servidor y catálogo autorizado

Motivo: modalidad elegida para disponer de archivos y hojas en la primera versión.
Alternativa descartada: carga de archivos desde la interfaz web.
Revisar si: se requiere gestión remota de fuentes; revisar entonces permisos, validación y almacenamiento.

## Propuestas pendientes de aprobación

### Plan estructurado, no SQL o código generado ejecutable

Motivo: mantener operaciones verificables y evitar delegar acceso al sistema de archivos al LLM.
Alternativa no elegida: ejecución de SQL libre con filtros de texto, insuficiente como frontera de permisos.
Revisar si: hacen falta nuevas operaciones; ampliar el esquema y sus pruebas.

### Una fuente por pregunta y XLSX rectangular

Motivo: primera versión acotada y trazabilidad clara; Excel requiere contrato de hoja, tipos y fórmulas.
Alternativa no elegida: uniones arbitrarias y soporte de todos los formatos Excel desde el inicio.
Revisar si: preguntas reales necesitan múltiples fuentes o libros no tabulares.

### Ollama primero, proveedor remoto por adaptador y consentimiento

Motivo: requisito de ejecución local y sustitución sencilla del modelo, sin envío remoto automático.
Alternativa no elegida: acoplar el dominio a un SDK o asumir que todos los proveedores tienen la misma API.
Revisar si: cambia la política de datos o las capacidades exigidas al modelo.

### Respuestas concretas deterministas y tablas con procedencia

Motivo: que los hechos se puedan contrastar con el resultado de la consulta, sin reformulación que invente cifras.
Alternativa no elegida: enviar filas al modelo para que calcule o resuma libremente.
Revisar si: se requiere narrativa compleja; definir evaluación de fidelidad y minimización de datos.

### Descarga temporal del resultado y neutralización de fórmulas CSV

Motivo: descargar exactamente el resultado de esa ejecución, evitando reconsultas y fórmulas activas al abrirlo.
Alternativa no elegida: recalcular al descargar o exportar textos peligrosos sin advertencia.
Revisar si: se exige fidelidad textual sin prefijos; aprobar una política alternativa explícita.

### Portal local o backend autenticado; SSH como transporte

Motivo: el navegador no gestiona SSH y el túnel no sustituye autorización HTTP.
Alternativa no elegida: publicar API/Ollama directamente o incluir secretos en JavaScript público.
Revisar si: se requiere servicio público o multiusuario; redefinir identidad y aislamiento.

## Pendientes que bloquean construcción o release

Confirmar alcance/contratos y versiones antes de implementar. Acordar hardware, volumen, tipos de columnas, catálogo, modelo, autenticación y límites. Para release: preguntas reales, umbrales estadísticos, política remota, presupuesto, retención y hosts SSH.

La elección del stack no constituye aprobación del resto de esta propuesta ni del inicio de implementación.
