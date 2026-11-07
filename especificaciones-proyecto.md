# Especificaciones técnicas

## Estado y stack

Propuesta de fase 0; solo las decisiones marcadas como confirmadas en [decisiones.md](decisiones.md) están aprobadas.

Base confirmada: Python, FastAPI y DuckDB. Propuesto: Pydantic para contratos, openpyxl para normalizar `.xlsx`, pytest para pruebas, Ruff para estilo y mypy para tipado. Versiones y gestor de dependencias se fijarán al aprobar el entorno, no se asumen instalados.

CSV y Parquet se leerán con el motor tabular; Excel se normalizará antes de registrarlo. El catálogo declarará codificación, separador, encabezados, tipos y hoja. Las fórmulas no se ejecutan: se usan valores almacenados, y si faltan se informa el error. No se adivinan separadores, fechas ni tablas ambiguas silenciosamente.

## Arquitectura y SOLID

| Módulo propuesto | Responsabilidad | No hace |
| --- | --- | --- |
| Dominio | Contratos de fuentes, planes, resultados y errores | No importa FastAPI ni SDK de proveedores. |
| Catálogo | Resolver identificadores, metadatos y permisos de fuentes | No interpreta preguntas. |
| Planificador | Obtener un plan estructurado mediante un puerto LLM | No abre archivos ni ejecuta consultas. |
| Validador/compilador | Validar el plan y generar operaciones permitidas | No acepta SQL libre ni rutas del modelo. |
| Lectores | Adaptar CSV, Parquet y Excel a relaciones tipadas | No deciden qué responder. |
| Motor de consulta | Ejecutar lectura, filtros y cálculos en DuckDB | No genera lenguaje natural. |
| Servicio de aplicación | Orquestar selección, validación y consulta | No contiene detalles de HTTP o proveedor. |
| Presentador y exportador | Convertir resultados a respuesta/CSV | No recalcula cifras ni llama al modelo. |
| API y web | Autenticación, contratos HTTP y presentación | No implementan reglas tabulares. |

SRP: responsabilidades separadas. OCP: registro de lectores y adaptadores, sin añadir condicionales de proveedor al servicio. LSP: pruebas contractuales comunes. ISP: puertos pequeños de catálogo, planificación, consulta y almacenamiento temporal. DIP: el servicio depende de estos puertos; la composición inyecta implementaciones.

Estructura propuesta: `src/consulta_datos/{domain,application,infrastructure,api,web}`, `tests/{unit,integration,e2e,evals}`. No se crean capas vacías ni abstracciones adicionales sin necesidad.

## Contratos internos propuestos

Firmas conceptuales; los tipos concretos se fijarán antes de la primera porción:

```text
SourceCatalog.list_authorized(principal) -> list[SourceMetadata]
SourceCatalog.resolve(principal, source_id) -> AuthorizedSource
QueryPlanner.plan(question, metadata) -> QueryPlan | Clarification
PlanValidator.validate(plan, authorized_sources) -> ValidatedPlan
DataReader.load(authorized_source) -> TabularRelation
QueryExecutor.execute(validated_plan, relation, limits) -> QueryResult
ResultStore.save(principal, result) -> StoredResult
ResultStore.get(principal, result_id) -> StoredResult
CsvExporter.export(table_result) -> CsvDocument
```

`SourceMetadata`: identificador estable, nombre, descripción de negocio, formato, hoja opcional, columnas con tipo y descripción, versión de fuente. El LLM recibe metadatos autorizados y la pregunta; no rutas, claves ni muestras de filas por defecto. La selección usa tanto el significado de negocio como el esquema.

`QueryPlan`: `source_id`, proyecciones, filtros tipados, agregaciones, agrupaciones, orden, límite y `output_kind` (`text` o `table`). Operaciones iniciales propuestas: comparaciones, pertenencia, nulos y combinación AND; agregaciones `count`, `sum`, `avg`, `min`, `max`. No contiene SQL, Python, funciones de archivo ni expresiones libres. Otros operadores requieren ampliar el contrato.

El validador comprueba el esquema completo, identificadores, tipos y compatibilidad de agregaciones/grupos. El compilador escapa identificadores resueltos desde el catálogo y parametriza valores. Nunca concatena entrada libre en SQL. DuckDB se ejecuta aislado, sin red, instalación de extensiones ni acceso general al sistema de archivos; solo se registran relaciones autorizadas. La configuración de aislamiento se verificará con la versión elegida.

`QueryResult`: columnas tipadas, filas, cantidad devuelta, truncamiento, procedencia y versión de fuente. Orden estable para resultados exportables. Fechas ISO 8601, decimales como cadenas cuando sea necesario para preservar precisión, nulos JSON `null`; NaN/infinito no se serializan como números JSON.

La respuesta concreta usa una plantilla determinista sobre un resultado escalar. El plan solicita una tabla para resultados multirregistro. Un agregado nulo o sin filas no se convierte en cero. Si hay varias fuentes plausibles, faltan unidades o una pregunta exige una unión, se pide aclaración o se declara operación no soportada; no se inventa una respuesta.

## API HTTP propuesta

API versionada en `/api/v1`. JSON con uniones discriminadas por `kind`; los ejemplos muestran la forma, no una implementación disponible.

| Método y ruta | Contrato |
| --- | --- |
| `GET /health` | Estado del proceso, sin rutas ni secretos; no afirma que el LLM esté disponible. |
| `GET /api/v1/sources` | Metadatos del catálogo accesibles al solicitante. |
| `POST /api/v1/queries` | Entrada `{"question":"¿Cuál es el total de ventas?"}`; ejecución síncrona acotada. |
| `GET /api/v1/results/{result_id}/csv` | Descarga de una tabla persistida temporalmente, sin repetir la consulta. |

Respuesta concreta:

```json
{"kind":"text","request_id":"...","answer":"El total de ventas es 1500.","value":"1500","value_type":"decimal","sources":[{"source_id":"ventas","version":"..."}]}
```

Respuesta tabular:

```json
{"kind":"table","request_id":"...","result_id":"...","columns":[{"name":"region","type":"string"},{"name":"total","type":"decimal"}],"rows":[["Norte","1500"]],"row_count":1,"truncated":false,"sources":[{"source_id":"ventas","version":"..."}],"csv_url":"/api/v1/results/.../csv","expires_at":"..."}
```

Aclaración: `{"kind":"clarification","request_id":"...","message":"Indica qué período deseas consultar.","candidate_source_ids":["ventas"]}` con HTTP 200; no ejecuta consultas.

Error: `{"kind":"error","request_id":"...","code":"MODEL_UNAVAILABLE","message":"El modelo no está disponible."}`. HTTP 422 para entrada/operación no válida; 401/403 para acceso; 404 para resultado desconocido o ajeno; 410 para resultado propio expirado; 429 para límites; 502 para salida inválida del proveedor; 503 para indisponibilidad; 504 para timeout; 500 para error inesperado. Un proveedor fallido nunca se presenta como resultado vacío exitoso.

Descarga: UTF-8, `text/csv; charset=utf-8`, `Content-Disposition: attachment` con nombre generado por el servidor. Mismas filas y orden que el resultado visible, incluidos límites; no exporta filas ocultas. Escapado CSV estándar. Propuesto: neutralizar celdas de texto susceptibles de fórmulas de hoja de cálculo al descargarlas; la interfaz advertirá que se añade un prefijo seguro y las pruebas verificarán esta única transformación. Los valores numéricos negativos no se transforman como texto.

El almacenamiento temporal liga cada resultado a su principal, conserva el resultado obtenido aunque cambie la fuente y elimina contenido al expirar. No se aceptan rutas de descarga aportadas por el usuario.

## LLM, privacidad y operación

Ollama: primer adaptador local, URL configurable, modelo configurable y tiempo máximo explícito. Sin fallback remoto automático. Un servicio similar requiere su propio adaptador o demostrar compatibilidad contractual.

Proveedores pagos: secretos mediante variables de entorno o gestor de secretos; configuración separada de la credencial de acceso HTTP. OpenAI/DeepSeek pueden compartir partes del transporte si se comprueba su compatibilidad; Gemini requiere adaptar su contrato. El dominio no depende de sus SDK. Una API KEY no garantiza salida estructurada válida.

Flujo de datos: pregunta + metadatos mínimos → LLM; plan validado → motor local; resultado → portal autorizado. Resultados completos no van al LLM. Antes de activar un proveedor externo, aprobar qué preguntas/metadatos pueden salir, retención, región, condiciones del proveedor y presupuesto. Los logs registran identificador, duración, proveedor/modelo y códigos, no prompts, filas, credenciales ni rutas.

Todo contenido del usuario, catálogo y archivos es no confiable. Las instrucciones del prompt no sustituyen validación ni aislamiento. Las rutas se resuelven dentro de la raíz autorizada, incluyendo enlaces simbólicos. La versión de fuente debe reflejar cambios; si cambia durante una consulta se invalida o se consulta un snapshot consistente.

Propuesta inicial de límites, todavía sin aprobar: pregunta de 4.000 caracteres, 1.000 filas y 5 MiB por resultado serializado, timeout total de 60 segundos, dos consultas concurrentes, sin reintentos automáticos y retención temporal de 15 minutos. Se medirán con hardware y archivos representativos; también debe limitarse memoria, volumen de entrada y tiempo del motor, no solo filas de salida.

## Integración del portal y SSH

HTTP es el protocolo de aplicación; SSH transporta la conexión, no sustituye los endpoints.

Escenario A: portal con backend en otro servidor. El backend mantiene el túnel y llama al puerto local. El navegador llama al backend del portal; nunca maneja claves SSH ni se conecta al `127.0.0.1` del servidor.

Escenario B: acceso personal desde un navegador local. El túnel corre en el equipo del usuario; el navegador abre la interfaz del servicio mediante el puerto reenviado. Servir web y API bajo el mismo origen evita CORS.

Ejemplo ilustrativo, pendiente de prueba en hosts reales:

```sh
ssh -N -T -o ExitOnForwardFailure=yes -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -L 127.0.0.1:18000:127.0.0.1:8000 usuario@servidor-datos
```

El servicio escucha en `127.0.0.1:8000` del servidor de datos. Desde el equipo que mantiene el túnel:

```sh
curl --fail http://127.0.0.1:18000/health
curl --fail -H "Authorization: Bearer <TOKEN_HTTP>" -H "Content-Type: application/json" -d '{"question":"¿Cuál es el total de ventas?"}' http://127.0.0.1:18000/api/v1/queries
```

Comandos de referencia redactados el 2026-11-07; **no ejecutados ni verificados**, y no hay servidor aún. Sustituir hosts, usuario y token; el marcador no es una credencial.

El portal debe interpretar `kind`, mostrar errores/aclaraciones y usar `csv_url` con la misma autenticación. Si usa un proxy, también debe reenviar la descarga y traducir la ruta relativa; una URL `localhost` del backend no se entrega al navegador. Descargar con `fetch` autenticado si el token se transmite por cabecera.

Para operación continua: cuenta SSH dedicada, clave fuera del repositorio, verificación de host estricta y supervisión del túnel con un gestor de servicios. Detectar caída y devolver error explícito. No abrir DuckDB, Ollama ni la API sin autenticación a Internet.

Acceso público futuro: proxy HTTPS, autenticación/autorización y límites; túnel SSH por sí solo no autentica al usuario HTTP. CORS usa una lista explícita si hay orígenes diferentes; CORS no es control de acceso. Se propone token HTTP solo en backend del portal o configuración local, nunca un secreto compartido incrustado en JavaScript público.

## Pruebas y puertas de calidad

Unitarias: esquema del plan, tipos, compilador, rutas, autorización, CSV, presentación y errores. Dobles del LLM: plan válido, ambiguo, JSON malformado, fuente inexistente, timeout, 429 e intento de ejecución libre.

Integración: mismas fixtures en los tres formatos, lectura de hojas, límites, nulos, decimales/fechas, cambios de fuente, expiración, fallos del proveedor y pruebas contractuales de adaptadores. Pruebas negativas de acceso a archivos externos, funciones SQL no permitidas, red, inyección y descarga ajena.

Extremo a extremo: pregunta web → resultado concreto; pregunta → tabla → descarga; ambigüedad; proveedor caído. Integración SSH real en la porción correspondiente, con corte del túnel y descarga autenticada.

Evaluación real: conjunto versionado de preguntas anonimizadas aportadas/revisadas por el usuario, estratificado por formato, selección de fuente, filtros/agregados, ambigüedad y casos adversarios. Separar desarrollo/prueba. Reportar acierto de fuente, exactitud del resultado, aclaraciones correctas y respuestas incorrectas seguras, por segmento, con N e intervalo Wilson al 95%. Registrar modelo, prompt, parámetros, hardware, fecha, p95 y coste. Fixtures sintéticas no sustituyen esta evaluación.

Pendientes: casos reales, tamaño del conjunto, umbrales por segmento, margen de regresión, presupuesto y objetivos de latencia. No afirmar calidad del LLM ni aprobar release sin estos acuerdos y evidencia.

Ruff, mypy y pytest son herramientas propuestas; sus comandos reproducibles se documentarán y ejecutarán al fijar dependencias. No existen todavía scripts, cobertura medida ni CI de aplicación.
