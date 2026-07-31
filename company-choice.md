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

## Objetivo del milestone

Con base en el análisis anterior, el objetivo de este proyecto es:

> Dar visibilidad y control en tiempo real a **Operaciones de restaurante** (ventas, stock y pedidos sugeridos por local) y estandarizar la **Formación** (recetas versionadas y onboarding) a través de los 14 locales de Brasaland en Colombia y EE. UU., cerrando la brecha entre lo que pasa en cada local y lo que sabe la sede en Medellín.

Este objetivo se considera alcanzado cuando, para cada departamento, se pueda responder sin llamadas ni hojas de Excel a las preguntas que hoy no tienen respuesta:

- **Operaciones:** ¿cuánto llevamos vendido hoy, por local y consolidado? ¿qué local se está quedando sin un ingrediente? ¿qué local abierto no ha reportado ventas?
- **Formación:** ¿qué versión de una receta está vigente en cada local? ¿qué locales aún no la han confirmado? ¿en qué punto de su onboarding está cada empleado nuevo?

El *cómo* (arquitectura, stack, modelo de datos, agente) se define en el siguiente hito.
