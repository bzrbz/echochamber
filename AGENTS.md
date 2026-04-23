# AGENTS.md — echochamber

Instrucciones para agentes de código que trabajen en este repo.

## Qué es echochamber

Red social cerrada por invitación. Lee `PLAN.md` para la tesis, decisiones previas, fases y criterios de salida. Lee `README.md` para el resumen.

**Nota sobre naming:** el repo y el proyecto se llaman `echochamber`; el dominio de producción es `echo.bzr.bz` (corto, pronunciable). Usa ambos nombres según contexto — respeta `echochamber` en código, imports, nombres de servicios; usa `echo.bzr.bz` en docs de despliegue, emails, y textos cara al usuario.

## Principios innegociables

- **Sin votos, scoring ni karma.** Es una decisión de diseño, no de alcance. En un círculo pequeño la curación la hace la invitación. No propongas "solo upvotes" ni "score oculto" como evolución — cambia el tipo de contenido que la gente se atreve a postear.
- **Sin feed público, sin descubrimiento, sin algoritmo.** Feed ordenado por actividad reciente: los posts con último comentario más nuevo suben; el resto cae por fecha de creación. Simple y predecible.
- **Monocírculo primero.** Todo el MVP asume un solo círculo global. El refactor a multicírculo (Fase 3) solo ocurre si hay señal real. No estructures el código "preparado para multicírculo" especulativamente.
- **Auth por magic link.** Nunca passwords. Email + token de un solo uso con expiración corta.

## Stack propuesto

- **Backend**: Elixir/Phoenix con LiveView (primera opción; tiempo real nativo, formularios triviales, auth resuelta). Alternativas válidas: Go+HTMX o Deno Fresh si el humano lo prefiere.
- **DB**: SQLite en disco, con Litestream replicando a S3/R2 para backup. Postgres es overkill para este tamaño.
- **Email**: Resend (tier gratuito sobra al principio) para magic links y notificaciones.
- **Hosting**: Fly.io (single-region cerca de usuarios) o VPS en Hetzner (5-10€/mes).
- **Analítica**: Plausible o Umami self-hosted. Métricas a seguir: DAU, posts/día, comentarios/día. Nada más.

Si acabas usando Phoenix, ten cuidado con patrones viejos del ecosistema. Verifica la versión exacta de Phoenix y LiveView del `mix.exs` antes de proponer código — la API ha cambiado en los últimos major releases.

## Esquema de datos (MVP)

Tablas esperadas en SQLite:

- `users` — id, email (único), handle, created_at, banned_at
- `invites` — code, created_by (user_id), created_at, used_by, used_at, expires_at
- `posts` — id, author_id, url, title, body, created_at, last_activity_at, deleted_at
- `comments` — id, post_id, parent_comment_id (nullable, max 3 niveles), author_id, body, created_at, deleted_at
- `bans` — user_id, banned_by, reason, created_at

**No crees** tabla `votes` ni campos de score/karma en users/posts.

El campo `last_activity_at` en `posts` es el que alimenta el orden del feed: se actualiza al crear un comentario. Feed principal: `ORDER BY last_activity_at DESC`.

## Consideraciones legales (no opcional)

Alojas contenido de usuarios. Antes de cualquier despliegue público:

- **Política de privacidad clara** en `/privacy`: qué datos se guardan, durante cuánto, cómo se borran.
- **Términos mínimos** en `/terms`: responsabilidad del usuario, derecho del owner a kickear, posibilidad de cerrar con preaviso.
- **Borrado de cuenta real** (no soft delete) accesible desde ajustes del usuario.
- **Export propio** en JSON desde ajustes: todos tus posts, comentarios y metadata.
- **Cabeceras básicas de seguridad**: CSP, HSTS, X-Frame-Options, Referrer-Policy.

Jurisdicción asumida: UE (GDPR aplica). Si el humano indica lo contrario, ajusta.

## Ejecución y comandos

(Rellenar cuando exista el proyecto. Ejemplo para Phoenix:)

```sh
mix setup              # instala deps, crea db, migra, seeds
mix phx.server         # dev en http://localhost:4000
iex -S mix phx.server  # con repl interactivo
mix test               # test suite
mix format             # formatter
mix credo              # linter
```

Secretos en `.env` (nunca commitear). `.env.example` debe estar en el repo.

## Flujo con el agente

- **Una sesión por ítem del checklist de Fase 1 en `PLAN.md`**. Marcar checkboxes al terminar.
- **Migraciones nunca destructivas**: añadir columnas nullables, migrar datos con backfill, dropear columnas viejas solo después de un release estable.
- **Backups probados**: una migración que no se ha probado restaurando es una migración que no existe. Antes de cada cambio de esquema serio, ejecutar el flow de backup y restore local.
- **Tests**: auth y permisos sí se testean desde el principio (es la superficie donde más fácil es meter la pata). El resto, según criterio.
- **Commits atómicos**, estilo conventional, en español o inglés consistente.

## Criterios de "hecho" para una feature

1. Funciona con 5 usuarios concurrentes sin romper (smoke test manual o LiveView test).
2. No expone datos de un círculo a quien no pertenece (test de autorización obligatorio).
3. Emails transaccionales renderizan en Gmail, Apple Mail y un cliente texto-plano.
4. UI usable en móvil (la mitad de los clicks van a venir de ahí).
5. Accesible: formularios con labels, focus visible, contraste suficiente.

## Cosas que NO hacer en el MVP

Explícito en `PLAN.md`, repetido aquí por comodidad:

- Upload de imágenes (solo enlaces externos).
- Mensajes privados entre usuarios.
- Perfiles con bio, avatar custom, historial.
- App móvil nativa (responsive web basta).
- **Votos, scoring, karma** (decisión de diseño permanente).
- Multi-idioma.
- Integración con servicios externos (Twitter cross-post, RSS…).
