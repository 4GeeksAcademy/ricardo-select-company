# Empresa Elegida: Brasaland
referencia: Pagina de [Brasaland](https://github.com/4GeeksAcademy/ai-engineering-syllabus/blob/main/content/contexts/00-general-contexts/CONTEXT-brasaland-briefing.es.md)

la tematica de restaurante y su problema de saber la consumición en tiempo real.

### Departamentos elegidos:

Operaciones de restaurante - problema de saber que platos se estan pidiendo en tiempo real
Formación y estándares de calidad - el tender centralizada preparacion, mmenus para los restaurantes

## Mi idea de agente de IA  

Eres un arquitecto de software especialidado tanto en front-end como en back-end

usaremos para el front: react, typescrypt, style-componentes, apirest, websocket. Back: nodejs, typescript, postgresSql, apirest, websocket

nos vamos a limitar a diseñar la arquitectura para la solucion de como estructuraremos el projecto para resolver los problemas de como gestion de ventas en tiempo real,
ingredientes, stock de los ingredientes, gestion de pedidos sugerencias en base al stock alertas de local por no reportar ventas, o bajo stock.

tambien para la formacion, gestion recetas y cambios de la misma receta como recetas nuevas, ingredientes que utiliza compartido con la de operaciones,
onboarding para empleados, los caminos y los pasos, lista de empleados, prgreso de cada uno, porxima fecha de incorporacion


Modelo de datos (unificado). Diseña el esquema SQL completo, empezando por las tablas compartidas y luego las de cada dominio.

Base compartida:

locations (14 locales: país, moneda, zona horaria, horario de apertura por día, idioma por defecto).
users / roles (roles unificados de ambos dominios).

Operaciones:

sales / sale_items (venta con local, timestamp, importe en moneda local, nº de cubiertos).
ingredients (con unidad de medida) — compartida con Formación.
location_stock (stock de cada ingrediente por local, con umbral mínimo).
stock_movements (entradas/salidas/consumo).
suggested_orders (pedidos sugeridos por el motor).
alerts (tipo, local, estado, timestamp).

Formación:

recipes y recipe_versions (solo una versión "publicada" vigente).
recipe_translations (título, descripción, pasos, notas de presentación por idioma).
recipe_ingredients (relación receta↔ingredients con cantidades — conecta ambos dominios y permite descontar stock por venta).
categories.
recipe_acknowledgements (qué local ha confirmado qué versión y cuándo).
onboarding_paths, onboarding_steps.
employees / employee_progress /employee_start_date.

API REST (unificada). Define los endpoints con métodos, rutas, payloads y códigos de estado, agrupados por dominio pero compartiendo auth, roles e i18n.

Operaciones: ingesta de ventas (simula el POS); consulta de ventas (cadena y por local, por rango de fechas, en USD); consulta de stock; generación/consulta de pedidos sugeridos; gestión de alertas; conversión de moneda (tipo de cambio configurable).

Formación: CRUD de recetas y publicación de versiones; búsqueda (texto + filtros por categoría/ingrediente + idioma); distribución de versión a los 14 locales y registro de acuses; gestión de itinerarios y progreso de empleados; selección de idioma con fallback a español.

Lógica de negocio clave:

Motor de pedidos sugeridos: dado el histórico de ventas y el stock actual de un local, calcula qué reponer y cuánto. Heurística explicada y parametrizable (p. ej. consumo medio diario × días de cobertura − stock actual). Sin ML.
Sistema de alertas: job programado que, según horario de apertura y zona horaria de cada local, detecta locales abiertos sin ventas y genera alerta.
Agregación en tiempo real: recálculo y push de totales del dashboard al llegar nuevas ventas (WebSockets).
Publicación y distribución de recetas: al publicar una versión, marca la anterior como obsoleta, genera "pendiente de confirmar" para los 14 locales y expone el estado por local.
Búsqueda: full-text de PostgreSQL respetando el idioma con fallback al base.
Progreso de onboarding: al completar un paso, actualiza el progreso y calcula el % de avance.

Frontend (React). Componentes para ambos dominios bajo una misma shell con navegación y selector de idioma:

Operaciones: dashboard principal (total de cadena hoy en COP y USD, desglose por local, tiempo real, cubiertos y ticket medio); vista de local (ventas + stock + pedidos sugeridos); panel de alertas; selector de moneda de visualización.

Formación: catálogo de recetas con búsqueda, filtros y selector de idioma; detalle de receta (versión vigente, ingredientes, pasos, notas de presentación); panel del equipo de Formación para editar y publicar versión, con estado de confirmación por local; vista de onboarding de un empleado con su progreso.


