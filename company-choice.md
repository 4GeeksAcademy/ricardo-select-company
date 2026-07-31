# Empresa elegida: Brasaland

Referencia: [Briefing de Brasaland](https://github.com/4GeeksAcademy/ai-engineering-syllabus/blob/main/content/contexts/00-general-contexts/CONTEXT-brasaland-briefing.es.md)

Brasaland es una cadena de restaurantes de cocina a la brasa con 14 locales en Colombia y Estados Unidos (Florida). Factura ~$6M/año y opera con herramientas pensadas para un único restaurante local (WhatsApp, Excel, tarjetas de sello físicas), no para una cadena multipaís.

## Por qué elegí esta empresa

<!-- TODO: ajusta este párrafo a tu motivación real; esto es un borrador de partida -->

Elegí Brasaland por tres razones:

- **Complejidad multipaís real**: dos países, dos monedas (COP/USD), dos idiomas y dos zonas horarias. Es un problema de ingeniería más interesante que un CRUD de un solo mercado.
- **Dominio fácil de entender**: todo el mundo sabe qué es un restaurante, así que puedo centrarme en resolver bien el problema técnico (tiempo real, consistencia de datos, i18n) en lugar de explicar el negocio.
- **Impacto medible y tangible**: los problemas de Brasaland (pedidos por WhatsApp, alertas manuales, recetas desactualizadas) tienen consecuencias operativas claras — se puede argumentar el "antes vs. después" de la solución sin ambigüedad.

## Departamentos elegidos y casuísticas

### 🍖 Operaciones de restaurante

**Responsable en la empresa:** Felipe Guerrero

**Problema actual:** cada uno de los 14 locales opera de forma aislada. No hay visibilidad centralizada de ventas en tiempo real (ni cubiertos servidos ni ticket medio). Los pedidos de ingredientes se hacen por WhatsApp o teléfono, sin datos de stock detrás, lo que provoca exceso de inventario en unos locales y roturas de stock en otros. No hay alertas cuando un local abierto no reporta ventas.

**Impacto:** decisiones a ciegas — Felipe no puede responder "¿cuánto llevamos vendido hoy en Miami?" sin llamar. El exceso/rotura de stock afecta directamente al margen y a la experiencia del cliente.

**Casuísticas concretas a resolver:**
- Ver ventas y cubiertos en tiempo real, por local y consolidado, en COP y USD.
- Saber en qué momento un local se está quedando sin un ingrediente clave.
- Generar automáticamente una sugerencia de pedido de reposición en vez de depender de la intuición del encargado.
- Detectar y alertar cuando un local abierto lleva X tiempo sin registrar ventas (posible incidencia operativa).

### 🎓 Formación y estándares de calidad

**Responsable en la empresa:** Jake Morrison

**Problema actual:** las recetas y estándares de preparación viven en un Google Drive compartido difícil de navegar. Cuando cambia una receta, comunicarlo a los 14 locales en dos idiomas lleva días y genera confusión sobre qué versión está vigente en cada local. El onboarding de personal de cocina (con alta rotación) es manual.

**Impacto:** inconsistencia de producto entre locales (la misma hamburguesa no sabe ni se presenta igual en Medellín que en Miami), y tiempo perdido en aclarar qué versión de una receta aplica.

**Casuísticas concretas a resolver:**
- Publicar una nueva versión de receta y saber qué locales ya la han confirmado y cuáles siguen con la anterior.
- Buscar recetas por texto, categoría o ingrediente, en el idioma del usuario con fallback a español.
- Dar de alta a un empleado nuevo con un itinerario de onboarding con pasos y progreso visible.
- Conectar recetas con ingredientes reales (`recipe_ingredients`) para poder, a futuro, descontar stock automáticamente al vender un plato — este es el puente entre Formación y Operaciones.

## Mi idea de agente de IA

### Alcance

Diseñar la arquitectura (modelo de datos, API y lógica de negocio) que resuelve, para los dos departamentos anteriores:
- Gestión de ventas en tiempo real, ingredientes, stock, pedidos sugeridos y alertas por local (Operaciones).
- Gestión de recetas y sus versiones, ingredientes compartidos con Operaciones, y onboarding de empleados con seguimiento de progreso (Formación).

### Stack tecnológico

| Capa | Tecnologías |
| --- | --- |
| Frontend | React, TypeScript, styled-components, API REST, WebSocket |
| Backend | Node.js, TypeScript, PostgreSQL, API REST, WebSocket |

### Modelo de datos (unificado)

**Base compartida**
- `locations` (14 locales: país, moneda, zona horaria, horario de apertura por día, idioma por defecto).
- `users` / `roles` (roles unificados de ambos dominios).

**Operaciones**
- `sales` / `sale_items` (venta con local, timestamp, importe en moneda local, nº de cubiertos).
- `ingredients` (con unidad de medida) — compartida con Formación.
- `location_stock` (stock de cada ingrediente por local, con umbral mínimo).
- `stock_movements` (entradas/salidas/consumo).
- `suggested_orders` (pedidos sugeridos por el motor).
- `alerts` (tipo, local, estado, timestamp).

**Formación**
- `recipes` y `recipe_versions` (solo una versión "publicada" vigente).
- `recipe_translations` (título, descripción, pasos, notas de presentación por idioma).
- `recipe_ingredients` (relación receta↔ingredientes con cantidades — conecta ambos dominios y permite descontar stock por venta).
- `categories`.
- `recipe_acknowledgements` (qué local ha confirmado qué versión y cuándo).
- `onboarding_paths`, `onboarding_steps`.
- `employees` / `employee_progress` / `employee_start_date`.

### API REST (unificada)

Agrupada por dominio, compartiendo auth, roles e i18n.

- **Operaciones:** ingesta de ventas (simula el POS); consulta de ventas (cadena y por local, por rango de fechas, en USD); consulta de stock; generación/consulta de pedidos sugeridos; gestión de alertas; conversión de moneda (tipo de cambio configurable).
- **Formación:** CRUD de recetas y publicación de versiones; búsqueda (texto + filtros por categoría/ingrediente + idioma); distribución de versión a los 14 locales y registro de acuses; gestión de itinerarios y progreso de empleados; selección de idioma con fallback a español.

### Lógica de negocio clave

- **Motor de pedidos sugeridos:** dado el histórico de ventas y el stock actual de un local, calcula qué reponer y cuánto. Heurística explicada y parametrizable (p. ej. consumo medio diario × días de cobertura − stock actual). Sin ML.
- **Sistema de alertas:** job programado que, según horario de apertura y zona horaria de cada local, detecta locales abiertos sin ventas y genera alerta.
- **Agregación en tiempo real:** recálculo y push de totales del dashboard al llegar nuevas ventas (WebSockets).
- **Publicación y distribución de recetas:** al publicar una versión, marca la anterior como obsoleta, genera "pendiente de confirmar" para los 14 locales y expone el estado por local.
- **Búsqueda:** full-text de PostgreSQL respetando el idioma con fallback al base.
- **Progreso de onboarding:** al completar un paso, actualiza el progreso y calcula el % de avance.

### Frontend (React)

Componentes para ambos dominios bajo una misma shell con navegación y selector de idioma.

- **Operaciones:** dashboard principal (total de cadena hoy en COP y USD, desglose por local, tiempo real, cubiertos y ticket medio); vista de local (ventas + stock + pedidos sugeridos); panel de alertas; selector de moneda de visualización.
- **Formación:** catálogo de recetas con búsqueda, filtros y selector de idioma; detalle de receta (versión vigente, ingredientes, pasos, notas de presentación); panel del equipo de Formación para editar y publicar versión, con estado de confirmación por local; vista de onboarding de un empleado con su progreso.
