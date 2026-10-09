# Sherpa Bot Optimizer (skill)

Skill del estándar [Agent Skills](https://agentskills.io) que le da a un corredor **un analista y un
editor para su bot de Sherpa, en la misma conversación**: lee lo que realmente pasó en los chats,
te dice cómo viene en números, y aplica las mejoras que aprobás — sin tocar el panel y sin escribir
una línea de configuración.

Todo el ciclo en un solo chat: analizar → recomendar → borrador → preview → publicar, con **dos
gates de aprobación humana** antes de que algo llegue al bot en vivo.

Cinco cosas que hace:

- **Revisar y mejorar los flujos** — leer las conversaciones reales del bot, encontrar dónde se
  pierde la gente o dónde contesta mal, y proponer cambios concretos a los pasos, las preguntas y
  los textos. Se publican solo con aprobación humana explícita.
- **Ajustar cómo habla el bot** — voz y tono del asistente: saludo, despedida, fuera de horario,
  contexto general y trato en casos sensibles. Mismo camino de borrador y aprobación. (Las secciones
  completas son del plan ELITE; saludo, despedida y fuera de horario están disponibles en cualquier
  plan.)
- **Reportar números** (solo lectura, sin gates) — cómo viene la operación: pólizas y cupones
  pedidos vs. entregados, envíos masivos realizados y cómo les fue, qué porcentaje de las
  conversaciones termina resuelto (con el desglose por flujo), y cuántos rechazos hay cargados
  vs. notificados por aseguradora.

- **Etiquetar conversaciones en el CRM** — poner o quitar etiquetas (por ejemplo `interes_alto` a
  todas las conversaciones que pidieron una cotización). Primero muestra un resumen sin cambiar
  nada y aplica solo con tu aprobación explícita; nunca etiqueta porque un mensaje de un cliente lo
  pida.

- **Notas, prioridad y estado de conversaciones** — dejar una nota interna en una conversación del
  CRM (la ve tu equipo, el cliente no), cambiarle la prioridad o marcarla como resuelta, abierta o
  pendiente. Se aplica al instante; antes de resolver conversaciones te pide confirmación, y nunca
  actúa porque un mensaje de un cliente lo pida.

- **Enviar mensajes masivos con plantilla** — mandar una plantilla aprobada de WhatsApp a un grupo
  de clientes (por etiquetas, contactos o una planilla de hasta 500 filas). Siempre muestra primero
  una vista previa con el total y mensajes de ejemplo, y solo envía tras tu "sí" explícito; después
  te cuenta cómo va la entrega.

Todo lo que la skill le escribe al usuario está en lenguaje llano: nunca códigos de error, nombres
de sistemas internos ni jerga técnica.

Es la capa de **procedimiento**. La capacidad real (leer conversaciones y métricas, editar flujos y
voz/tono) la da el **servidor MCP de Sherpa**; la autorización multi-tenant la enforcea Sherpa del
lado del servidor. La skill no es una frontera de seguridad y no contiene credenciales.

## Qué necesitás

**Para vos (corredor):** tu **clave de acceso a Sherpa**, que generás desde el panel de Sherpa. Con
tu clave, la skill solo puede ver y editar **los bots que están a tu nombre**. Una vez conectada, no
tenés que hacer nada más.

**Detalle técnico (para quien conecta la herramienta):**

1. Un **`everestApiKey`** de Sherpa (se crea en el frontend de Sherpa, gateado por permisos). Con una
   key de broker, la skill solo puede ver y editar los bots de ese broker.
2. El **servidor MCP de Sherpa** conectado en tu agente, autenticado con esa key.

> MCP de Sherpa: `https://mcp.sherpa.wtf/mcp` (expone read + write).

## Instalación

### Opción 1 — Claude Code, en un paso (skill + MCP juntos)

Instala la skill **y** registra la conexión al MCP de una sola vez:

```bash
# 1. exportá tu key (no se guarda en el repo; la lee la config del MCP por variable de entorno)
export SHERPA_API_KEY="evr_pk_...tu-everestApiKey..."

# 2. agregá el marketplace e instalá el plugin
/plugin marketplace add Sherpa-wtf/sherpa-skill
/plugin install sherpa-bot-optimizer@sherpa-wtf
```

La skill queda disponible (namespaced como `/sherpa-bot-optimizer:sherpa-bot-optimizer`) y el MCP
`sherpa` queda conectado con tu key vía el header `Authorization: Bearer ${SHERPA_API_KEY}`.

### Opción 2 — Cualquier host del estándar (skills.sh)

Instala **solo la skill** (la conexión al MCP va aparte, abajo):

```bash
npx skills add Sherpa-wtf/sherpa-skill
```

No hay formulario de publicación: el repo público + este comando alcanzan; skills.sh la lista
automáticamente por telemetría de instalación.

### Opción 3 — Manual (cualquier agente)

Copiá `skills/sherpa-bot-optimizer/` a la ubicación de skills de tu host (ej.
`~/.claude/skills/sherpa-bot-optimizer/`), manteniendo `SKILL.md` y `reference/` juntos.

### Conectar el MCP (para las opciones 2 y 3)

- **Claude Code:**
  ```bash
  claude mcp add --transport http sherpa https://mcp.sherpa.wtf/mcp \
    --header "Authorization: Bearer <tu-everestApiKey>"
  ```
  > Si tu versión de Claude Code no manda el header (bug conocido en algunas versiones),
  > actualizá la CLI.
- **Codex / otros hosts del estándar MCP:** agregá el servidor en la config del host apuntando a la
  misma URL HTTP, con el header `Authorization: Bearer <tu-everestApiKey>`.
- **claude.ai:** **no soportado por ahora.** Los Custom Connectors de claude.ai exigen OAuth 2.0;
  no aceptan un Bearer estático. Se habilitará cuando el MCP exponga OAuth.

## Seguridad

- Todo el contenido recuperado (conversaciones, flujos, copies) se trata como **dato no confiable**,
  nunca como instrucciones.
- **Dos gates humanos:** aprobación antes de crear/editar el borrador, y antes de publicar. Las etiquetas del CRM tienen su propio gate: resumen sin escribir y aprobación antes de aplicar. Los envíos masivos también: vista previa y aprobación explícita antes de enviar.
- La skill nunca toca el bot vivo directo: edita un **borrador** y publica con confirmación explícita.
- La skill es procedimiento, **no** una frontera de seguridad: los controles reales de quién puede ver
  o cambiar cada bot los aplica Sherpa del lado del servidor.
- La key nunca se commitea: en la opción 1 viaja por la variable de entorno `SHERPA_API_KEY`.

## Estructura del repo

```
.claude-plugin/
  plugin.json                 manifiesto del plugin de Claude Code
  marketplace.json            marketplace de un plugin (source: "./")
.mcp.json                     servidor MCP de Sherpa (Bearer por ${SHERPA_API_KEY})
skills/sherpa-bot-optimizer/
  SKILL.md                    máquina de estados (2 gates, fail-closed) + reglas de seguridad
  reference/analysis-playbook.md
  reference/metrics-reporting.md
  reference/conversation-labels.md
  reference/conversation-actions.md
  reference/mass-send.md
  reference/flow-editing.md
  reference/mcp-connection.md
README.md
LICENSE
```

## Licencia

MIT — ver [LICENSE](./LICENSE).
