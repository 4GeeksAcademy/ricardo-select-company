# Progress — Monorepo Brasaland

| Hito | Estado | Fecha | Resumen |
|---|---|---|---|
| Hito 1 — Sitio web público | Completado | — | Landing pública + formulario "Brasa Points", migrado a Next.js en Hito 4 (antes HTML estático). |
| Hito 2 — Fundamentos de programación | Completado | — | Utilidades TypeScript puras de procesamiento de datos de Brasaland en `packages/data-utils` (filtrado, búsqueda, cálculos financieros, scoring, validaciones). |
| Hito 3 — Talent Pipeline Tracker | Completado | — | App Next.js en `uis/talent-pipeline-tracker` para gestionar candidaturas de selección de personal, consumiendo la API REST del curso. |
| Hito 4 — Ingeniería impulsada por IA | Completado | 2026-08-06 | Infraestructura de agentes (`memory-bank/`, `AGENTS.md`, `.agents/`) + migración de `uis/website` a Next.js + creación de `uis/backoffice` importando `packages/data-utils`. |
| Project 1 — Directorio de Proveedores | Completado | 2026-08-07 | Primer backend real del monorepo: API FastAPI + TinyDB + Pydantic en `services/api` para el directorio de proveedores de Compras y Proveedores, más página `/suppliers` en `uis/backoffice`. |
| Project 2 (AUTH-03) — Recuperación y cambio de contraseña | Completado | 2026-08-07 | Primer sistema de autenticación del monorepo: login (JWT) + forgot/reset/change-password en `services/api`, envío de email vía Resend, 4 páginas nuevas en `uis/backoffice`. |
| Hito 5 — Backoffice de Gestión de Inventario | Frontend completado, bloqueado para pruebas end-to-end | 2026-08-10 | 4 vistas nuevas en `uis/backoffice/src/app/inventory` (productos, entrada, salida, historial) consumiendo una API `/inventory` que **todavía no existe** en `services/api` — ver detalle. |

## Hito 5 — detalle de lo entregado

- **Gap real entre el rule y el repo, dejado sin resolver por decisión explícita del desarrollador**: `rule-5.md` asume un backend `/inventory` ya entregado ("handoff del equipo de backend"), pero `services/api` solo tiene `/auth` y `/suppliers`. A diferencia de Project 2 (donde el gap de auth sí se resolvió construyendo el mínimo backend), aquí se decidió seguir el alcance literal de `rule-5.md` (solo frontend) y no construir `/inventory`. Consecuencia: el build y el lint de `uis/backoffice` están en verde, pero **no se ha podido verificar ningún flujo contra datos reales** — ver `context/plans/hito5.md`, sección "Pendiente".
- `CONTEXT.md` raíz reemplazado por el briefing de `context/context/hito-5.md` (Gestión de Inventario — Ingredient/IngredientEntry/IngredientExit).
- `uis/backoffice`: `NEXT_PUBLIC_INVENTORY_API_URL` nueva en `.env.example`/`.env.local` (rule-5 la pide separada de `NEXT_PUBLIC_API_URL`, aunque en local apunte al mismo `services/api:8000`). `lib/inventoryApi.ts` (mismo patrón de `ApiError`/`request` duplicado que `suppliersApi.ts`/`authApi.ts`), `lib/inventoryLabels.ts`, `types/inventory.ts`. `lib/session.ts` gana `getCurrentUserId()` (decodifica el `sub` del JWT) porque el modelo de usuario de este repo no tiene campo `uuid` — se envía el id numérico como string en `user_uuid`, autocompletado, no es un campo del formulario.
- Rutas sin el prefijo `/backoffice` que pide literalmente rule-5 (`/inventory/products`, `/inventory/orders/inbound`, `/inventory/orders/outbound`, `/inventory/orders`) — misma adaptación que ya rompió Project 1 con `/suppliers`.
- Nuevo hook `useRequireAuth` (redirección real a `/login`, no mensaje inline) — patrón más estricto que el usado en `/account/change-password`, no retrofiteado ahí.
- Umbrales de stock arbitrarios y documentados en `StockBadge.tsx` (`<=0` sin stock, `<20` stock bajo, resto saludable) — no hay campo de stock mínimo en el spec.
- Verificación pendiente: no se pudo probar en navegador contra datos reales (backend `/inventory` inexistente). Solo se verificó `npm run build` y `npm run lint` en verde.

## Project 2 (AUTH-03) — detalle de lo entregado

- **Prerequisito no pedido por el rule, añadido por necesidad real**: el rule asumía un sistema de autenticación ya funcionando, pero no existía ninguno en el monorepo (verificado en todas las ramas). Se añadió lo mínimo: modelo `User`, hashing con `bcrypt`, `POST /auth/login` (JWT de sesión) y página `/login`. Usuarios de prueba seedeados (`felipe.guerrero@brasaland.com`, `jake.morrison@brasaland.com`, password `brasaland2026`), sin endpoint/página de registro (el rule no lo evalúa).
- `services/api`: `config.py` (carga `.env` con `python-dotenv`), `auth.py` (hashing + JWT + dependencia `get_current_user`), `mail.py` (Resend vía `httpx`, plantilla HTML inline), `routes/auth.py` con 4 endpoints (`login`, `forgot-password`, `reset-password`, `change-password`). Token de sesión = JWT (24h); token de reset = cadena aleatoria hasheada (`sha256`) en TinyDB con expiración de 30 min e invalidación tras un solo uso (`used=True`).
- `uis/backoffice`: `/login`, `/forgot-password`, `/reset-password` (lee `token` de la URL), `/account/change-password` — las 4 páginas con estado explícito `idle/submitting/success/error` en el `onSubmit` (variante del patrón de `frontend-fetch-pattern.md` para envíos de formulario, no fetch-on-mount). Sesión en `localStorage` (`lib/session.ts`).
- Verificado end-to-end con Playwright headless: login, cambio de contraseña autenticado (rechazo de contraseña actual incorrecta + éxito), flujo completo de olvido (email real recibido vía Resend, confirmación fija para email existente e inexistente, token inválido con error + enlace de vuelta, token válido con reset + redirección a `/login` + login con la nueva contraseña). Build y lint de `uis/backoffice` en verde.
- Servicio de email elegido: **Resend** (remitente de onboarding, sin dominio propio necesario en dev). Variable de entorno: `RESEND_API_KEY` en `services/api/.env` (nunca commiteada).

## Project 1 — detalle de lo entregado

- `services/api`: FastAPI + TinyDB + Pydantic, empaquetado con `uv` (`uv sync`, `uv run seed`, `uv run uvicorn main:app`). 6 endpoints (`POST/GET/GET-by-id/PATCH rate/PATCH status/DELETE /suppliers`), validación cruzada `country`/`currency` y de `categories`/`rate_per_unit` con 422, `seed.py` con los 15 proveedores literales de `CONTEXT.md` (idempotente). CORS habilitado para `http://localhost:3000`.
- `uis/backoffice/src/app/suppliers`: listado con filtros por país/categoría en la URL (sin recarga), alta de proveedor con validación cliente + manejo de error 422 de la API, edición de tarifa y toggle activo/suspendido con badge visual, todo con estado de fetch explícito y `AbortController` (`.agents/rules/frontend-fetch-pattern.md`).
- `CONTEXT.md` raíz reemplazado por el briefing del Milestone 09 (Directorio de Proveedores).
- Verificado end-to-end con Playwright headless (filtro por URL, alta de proveedor, toggle de estado) además de `uv run seed` + Swagger. Build y lint de `uis/backoffice` en verde.

## Hito 4 — detalle de lo entregado

- `memory-bank/projectbrief.md`, `techContext.md`, `progress.md`.
- `AGENTS.md` raíz con flujo de 6 pasos antes de cada commit.
- `.agents/rules/monorepo-imports.md` (siempre activa) y `.agents/rules/frontend-fetch-pattern.md` (por patrón de archivo, `uis/**/*.tsx`).
- `.agents/skills/close-milestone/SKILL.md`.
- `uis/website`: migrado de HTML estático a Next.js + TypeScript, 9 componentes reutilizables, rutas `/` y `/brasa-points`. Build y lint en verde.
- `uis/backoffice`: nueva app Next.js, layout propio, `/` importa `packages/data-utils` (ranking de locaciones + top ítems vendidos) directamente desde su ubicación original. Build y lint en verde.

## Próximos pasos

- Abrir PR `feature/password-reset` → `main` con capturas del email real recibido, Swagger de `reset-password` devolviendo 400 en un token reusado, y el flujo de `/account/change-password`.
- Futuros proyectos: el resto de las funcionalidades de Operaciones/Formación descritas en `projectbrief.md`, ahora con auth y backend real ya como precedente en el monorepo. Si en el futuro se necesita alta de usuarios desde el frontend, `POST /auth/register` no existe todavía — es una decisión de alcance nueva.
- **Bloqueante para cerrar Hito 5**: no existe backend `/inventory` en ningún sitio accesible. Antes de dar por probado el hito hace falta, o bien construirlo en `services/api` (ver `context/plans/hito5.md`, decisión de alcance), o bien apuntar `NEXT_PUBLIC_INVENTORY_API_URL` a uno ya existente en otro lugar.
