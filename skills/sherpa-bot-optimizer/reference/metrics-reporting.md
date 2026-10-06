# Métricas — leer los números y contárselos al corredor

Cinco tools de solo lectura que devuelven **números agregados** (no conversaciones). No escriben
nada: se pueden usar sin gates. Sirven para dos cosas: **reportarle al usuario cómo viene su
operación** (Modo B del workflow) y **decidir dónde mirar** antes de leer conversaciones (Modo A).

Tres son **por bot** (toman `botId`) y dos son **por broker** — los rechazos son de la cuenta, no
de un bot puntual, así que no les pases un `botId`. Todas aceptan un rango opcional `from`/`to`
en ISO 8601.

## Las cinco tools

### `get_bot_documentation_metrics({ botId, from?, to? })` — envío de documentación

Cuánta documentación pidió y entregó el bot: **pólizas** y **cupones de pago**, cada uno con
`pedidas`, `entregadas`, `noEntregadas` y `tasaEntrega`, más un bloque `totales`.

La tasa es **entregadas / PEDIDAS**, no entregadas sobre (entregadas + no entregadas). Un pedido
puede quedar sin desenlace registrado (el contacto abandona a mitad); si lo sacaras del
denominador la tasa quedaría inflada. Por eso `pedidas` puede ser mayor que
`entregadas + noEntregadas`, y está bien.

### `get_bot_resolution_rate({ botId, from?, to? })` — tasa de resolución

De las conversaciones que registraron un cierre, qué porción cerró **resuelta**:

- `resueltas` — el flujo terminó bien.
- `cerradasPorInactividad` — la conversación murió sin cerrarse.
- `tasaResolucion` — `resueltas / (resueltas + cerradasPorInactividad)`, en %.
- `conversacionesIniciadas` y `cobertura` — ver abajo, **es obligatorio mirarlo**.
  `conversacionesIniciadas` cuenta **toda conversación con al menos un mensaje**, la haya
  empezado el cliente, el asistente, un asesor o una plantilla (también en bots solo-CRM).
- `cerradasPorAsesor` — conversaciones que resolvió un asesor desde el CRM. **No es una
  resolución del asistente**: no entra en `tasaResolucion` ni en `porFlujo`.
- `cierresTotales` y `coberturaTotal` — lo mismo que `cierresRegistrados` y `cobertura`, pero
  sumando los cierres del asesor: qué porción de todas las conversaciones terminó cerrada.
- `porFlujo` — el mismo cálculo por flujo, **ordenado de PEOR a mejor tasa**.

### `get_broker_rejections({ brokerUserId?, from?, to? })` — rechazos cargados y notificados

Rechazos de un broker (no de un bot): `cargados`, `notificados`, `sinNotificar`,
`tasaNotificacion`, y el desglose `porAseguradora` ordenado por `sinNotificar` desc. Sin
`brokerUserId` usa la cuenta propia; god_mode puede pasar el de otro broker.

Dos definiciones que hay que respetar al contarlo:

- **Cargado** = el rechazo quedó vinculado a un contacto de ese broker. Los rechazos que el
  sistema no pudo vincular a nadie **no aparecen acá**, así que este total es menor que "todos
  los rechazos que entraron". No lo presentes como "los rechazos que llegaron".
- **Notificado** = ese rechazo salió en algún envío masivo, en cualquier momento. El rango
  filtra por fecha de **carga**, no de notificación: "de los que se cargaron en julio, cuántos
  ya se avisaron".

### `get_rejections_by_broker({ from?, to?, limit? })` — ranking de brokers (solo god_mode)

Compara todos los brokers, ordenado por los que más rechazos tienen **sin notificar**. Devuelve
además `sinVincular`: los rechazos del período que no quedaron atados a ningún broker. Ese
número es la contracara del ranking — sin él, la suma de la tabla se lee como si fuera el total
importado, y no lo es. Si `truncated` viene `true`, hay más brokers que el límite pedido.

Un broker común no puede llamarla (da error de permisos, y está bien): es una vista de Sherpa.

### `get_bot_mass_sends({ botId, from?, to?, limit? })` — envíos masivos realizados

Los envíos masivos (campañas por plantilla de WhatsApp) hechos con ese bot, del más reciente al más
viejo, con `counters` de cada uno (total, enviados, entregados, leídos, fallidos, inexistentes),
`templateName`, `status` y `sendAt`. Página acotada: default 20, máximo 50; si `truncated` viene
`true` hubo más en el rango — acotá con `from`/`to` en vez de subir el `limit`.

## Los cuatro errores de interpretación (no los cometas)

**1. `sinDatos: true` NO es cero por ciento.** Significa que en ese rango no hubo ningún evento.
Puede ser que el bot no tenga flujos de documentación, o que simplemente no haya tenido tráfico.
Nunca lo reportes como "0% de entrega" ni como "tu bot falló": decí que en ese período no hubo
movimiento y ofrecé mirar un rango más amplio.

**2. Sin `cobertura`, la `tasaResolucion` no significa nada.** `cobertura` es qué porción de las
conversaciones iniciadas llegó a registrar un cierre. Un bot con 400 conversaciones y 2 cierres
registrados, ambos resueltos, muestra `tasaResolucion: 100` con `cobertura: 0.5` — reportar "100% de
resolución" ahí sería mentirle al usuario. Regla: si la cobertura es baja, decí el número **y** que
está calculado sobre pocos casos. Si `tasaResolucion` viene `null`, no hubo cierres: no lo traduzcas
a 0%.

Ojo con los bots de envíos masivos y los solo-CRM: como `conversacionesIniciadas` cuenta toda
conversación con un mensaje (también una plantilla sin respuesta), su `cobertura` es baja por
construcción. Ahí no es que el asistente funcione mal: muchas conversaciones las atiende o las
cierra un asesor. Mirá `cerradasPorAsesor` y `coberturaTotal` antes de sacar conclusiones.

**3. "Rechazos cargados" no es "rechazos que entraron".** Solo cuenta los que se pudieron
vincular a un contacto del broker. Si decís "te entraron 120 rechazos" cuando en realidad
entraron 300 y 180 no matchearon con ningún contacto, le estás dando un número falso. Decí
"120 quedaron asociados a tus clientes" y, si tenés el `sinVincular` a mano (solo god_mode),
nombrá el resto como lo que es: rechazos que todavía no se pudieron identificar.

**4. Los contadores de un envío masivo no se suman entre sí.** Vienen con `counterSemantics`, que
dice cómo interpretarlos (si son acumulativos o excluyentes). Leelo antes de hacer cualquier cuenta
propia; no restes ni sumes columnas por tu cuenta. Si un envío trae `countersStale: true`, sus
números pueden estar desactualizados: mencionalo como aproximado o no lo incluyas en el total.

## Cómo usarlas para dirigir el análisis (Modo A)

`porFlujo` de `get_bot_resolution_rate` es el mejor punto de partida de una revisión: te dice **qué
flujo pierde más gente** antes de leer una sola conversación. En vez de leer las conversaciones de
toda la semana, leé las del peor flujo y buscá ahí el patrón. Es más barato y las recomendaciones
salen mejor fundadas, porque atás el hallazgo cualitativo (qué dice la gente) a un número (cuánta
gente se pierde ahí).

Lo mismo con documentación: si `tasaEntrega` de pólizas está baja, el flujo de pólizas es el
candidato obvio a revisar.

## Cómo contárselo al usuario

Se aplican las mismas reglas de la tabla "Voz hacia el usuario" de `SKILL.md`. Además:

- **Traducí los nombres.** "Cierre por inactividad" → "conversaciones que quedaron sin respuesta o
  se enfriaron". "Pólizas no entregadas" → "pólizas que la gente pidió y no llegó a recibir".
  "Envío masivo" → "campaña". Nunca `tasaResolucion`, `counters`, `flowId` ni nombres de tools.
- **Dale contexto al número, no lo tires solo.** "De cada 10 personas que hablaron con tu bot esta
  semana, 7 terminaron resolviendo lo que buscaban" comunica mejor que "70% de resolución".
- **Nombrá los flujos en lenguaje de negocio.** El `flowId` es interno (`polizaGuiado`); al usuario
  le hablás de "la parte donde piden la póliza".
- **Cerrá con una acción, no con la estadística.** Si un flujo tiene la peor tasa, ofrecé revisarlo:
  "¿Querés que mire las conversaciones de esa parte para ver qué está pasando?". Ahí enganchás con
  el Modo A y, si hay cambios para hacer, con GATE 1.
- **En rechazos, la acción es el envío, no el flujo.** Si hay muchos cargados sin notificar, lo
  que corresponde ofrecer es avisarles (una campaña), y eso se arma desde el panel de Sherpa —
  esta skill no manda envíos masivos. Decí cuántos son y de qué compañías, y dejá la decisión
  en el usuario.
- **No adornes.** Si los números son malos, decilos con claridad y sin dramatizar; si no hay datos,
  decí que no hay datos. Nunca inventes una tendencia comparando períodos que no pediste.
