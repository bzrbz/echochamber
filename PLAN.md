# Plan de trabajo — `echochamber`

> Red social cerrada, solo por invitación. Como un Reddit de bolsillo para comentar enlaces con amigos.

## La idea en dos párrafos

Un sitio donde un grupo pequeño de amigos (digamos: 5-50 personas) comparte enlaces y los comenta entre ellos. La estructura se parece a Reddit en el formato (posts con URL + hilo de comentarios) pero se aleja en lo esencial: sin feed público, sin algoritmo, sin descubrimiento, **y sin votos**. Solo se entra con invitación de alguien que ya está dentro, y lo que se publica solo lo ven los miembros del grupo.

El nombre es deliberadamente irónico. Una cámara de eco puede ser tóxica cuando es involuntaria y masiva, pero es exactamente lo que uno busca cuando lo que quiere es hablar con gente que comparte sus referencias. Aquí la cámara de eco es opt-in, pequeña, y tú sabes quién está dentro.

## Por qué puede ser interesante

- La gente ha migrado de Twitter a docenas de sitios distintos. Muchos grupos de amigos han acabado usando un canal de Telegram o WhatsApp para pasarse enlaces, y esa UX es terrible para hilos largos.
- El formato "enlace + hilo" de Reddit es muy bueno para conversar, pero no existe una versión cerrada y sencilla.
- Los costes son bajísimos: pocos usuarios, poco tráfico, poca moderación (las invitaciones la hacen por ti).

## Riesgos y asunciones

- **Asunción más fuerte**: que la gente quiere un espacio pequeño y que sus amigos se apuntarán. Señal temprana: si no consigues que 5 amigos entren en la primera semana, el problema es real. Sin masa crítica no hay producto.
- **Riesgo de contenido**: alojas enlaces y texto de otros. Jurisdicción (probablemente UE), GDPR, derecho al olvido. Tenerlo en cuenta desde el día 1, no cuando llegue un email de un abogado.
- **Riesgo de abuso**: aunque sea por invitación, puede haber drama interno. Mitigación: dar al "dueño del círculo" un kick/ban trivial desde el principio.
- **Riesgo de abandono**: redes pequeñas mueren si no hay actividad sostenida. Mitigación: aceptar que algunas fallecen y que es parte del modelo. Ofrecer export del círculo completo antes de archivar.

## Decisión de producto previa: ¿un círculo o muchos?

Dos caminos posibles, decidir antes de empezar:

**Opción A — monocírculo**: todo echo.bzr.bz es un solo grupo. Tú invitas a tus amigos, es vuestro sitio. Más simple de construir, más limitado. Cero escalabilidad.

**Opción B — multicírculo**: cualquiera puede crear un "círculo" y gestionar sus invitaciones. echochamber es la plataforma; los círculos son los grupos. Más trabajo, pero el producto puede crecer orgánicamente (cada círculo invita al siguiente) sin que tú tengas que hacer nada.

**Recomendación**: empezar por A para tener algo publicable en un mes, y si funciona evolucionar a B. El MVP que se describe abajo asume A.

## Fases

### Fase 0 — Sketch (1-2 días)

Decidir: pila técnica, modelo de datos (users, invites, posts, comments, votes), política de moderación (qué dice el "código de casa"), y diseño visual (tres pantallas: login, feed, hilo de post). Elegir tipografía y paleta que no recuerden a Reddit — evitar el naranja a toda costa.

### Fase 1 — MVP publicable (2-3 semanas)

Las funcionalidades mínimas para que un grupo de 5 amigos lo use una semana sin frustrarse:

Registro solo vía código de invitación (generas códigos a mano al principio). Login por email + magic link (nada de contraseñas). Crear post con URL y título opcional. Hilo de comentarios anidados (máximo 3 niveles de profundidad, suficiente para la mayoría de conversaciones). Feed ordenado por actividad reciente: los posts con comentario más nuevo suben al principio; los que no tienen actividad van cayendo por fecha. Vista de post individual. Editar y borrar tus propios posts y comentarios. Kick básico (el owner puede expulsar a alguien y revocar sus invitaciones).

Checklist concreto:

- [ ] Repo `bzr-echochamber` inicializado
- [ ] Esquema de base de datos (SQLite): users, invites, posts, comments, bans
- [ ] Auth con magic link (email)
- [ ] Generación y consumo de códigos de invitación
- [ ] CRUD de posts y comentarios
- [ ] Feed ordenado por actividad reciente (bump por último comentario, fallback a fecha de creación)
- [ ] Panel de admin mínimo (lista usuarios, botón kick)
- [ ] Email transaccional (Resend, Postmark o similar)
- [ ] Desplegado en `echo.bzr.bz`
- [ ] Política de privacidad y términos mínimos (hay que tenerlo sí o sí)

**Criterio de salida**: 5 amigos usándolo durante 7 días consecutivos.

### Fase 2 — Ajustes de vida real (2-4 semanas después de publicar)

Lo típico que se ve solo con uso real:

Preview de enlaces (og:image y og:description) al postear. Notificaciones por email cuando alguien responde a tu comentario (con opt-out). Menciones tipo `@usuario` que avisan al mencionado. Búsqueda simple por texto en posts y comentarios. Marcar posts como "ya leídos" para que el feed muestre solo lo nuevo.

La prioridad sale del uso, no de una lista preescrita. Si nadie pide previews pero todos piden menciones, menciones primero.

### Fase 3 — Multicírculo (solo si la Fase 2 funciona)

Si se cumple que hay >20 usuarios activos semanales y que al menos dos personas diferentes han pedido "cómo crearía yo mi propio círculo", entonces empieza el trabajo de abstraer la app para soportar múltiples círculos aislados. Esto es un refactor grande, no un add-on; no se hace hasta tener la señal.

### Fase 4 — Graduación o deprecación

Decisión a los 3 meses del lanzamiento:

- Si hay actividad sostenida (>3 posts nuevos por semana + >10 comentarios) → graduar: endurecer infra, añadir backups automáticos, documentar.
- Si el círculo está en silencio radial → export de todo el contenido para los miembros, banner de deprecación, archivo en un mes.

## Stack propuesto

**Backend**: Elixir/Phoenix o Go+HTMX o Deno Fresh. Los tres permiten publicar rápido con poca infraestructura. Si no hay preferencia fuerte, **Phoenix LiveView** encaja muy bien con este tipo de producto (tiempo real nativo, formularios triviales, auth bien resuelta).

**Base de datos**: SQLite en disco, con Litestream haciendo replicación continua a S3/R2 para backup. No merece la pena Postgres para este tamaño.

**Emails**: Resend. Tier gratuito alcanza de sobra al principio.

**Hosting**: Fly.io (una sola región cerca de los usuarios) o un VPS de 5€/mes en Hetzner.

**Analítica**: Plausible o Umami self-hosted. Contar DAU, posts/día, comentarios/día. Nada más.

## Cosas que expresamente no se hacen en el MVP

- Upload de imágenes propias (solo enlaces externos)
- Mensajes privados entre usuarios
- Perfiles con bio, avatar custom, historial
- App móvil (la web responsive basta)
- Feed algorítmico
- **Votos, scoring y karma** — decisión de diseño, no de alcance: en un grupo pequeño la curación la hace la invitación, no un contador. Introducir votos convertiría la conversación en concurso de popularidad y cambiaría el tipo de contenido que la gente se atreve a postear.
- Multi-idioma
- Integración con nada externo (Twitter cross-post, RSS, etc.)

## Consideraciones legales mínimas (importantes)

No es opcional cuando alojas contenido de usuarios. Antes de invitar a nadie:

- Política de privacidad clara: qué datos guardas (email, contenido), durante cuánto tiempo, cómo se borran.
- Términos básicos: el usuario es responsable de lo que postea, el dueño puede kickear sin motivo, el servicio puede cerrar con preaviso.
- Proceso de baja: un botón "borrar mi cuenta" que borre de verdad, no que desactive.
- Backup del propio usuario: "descargar todos mis datos en JSON" en la página de ajustes.

Si se hace multicírculo en Fase 3, revisar todo esto de nuevo con cabeza.

## Primer hito concreto

**Objetivo**: en 3 semanas desde hoy, 5 amigos con cuenta creada y al menos 10 posts entre todos. Si en la semana 2 no está la auth funcionando en producción, algo va mal y hay que reducir alcance del MVP.
