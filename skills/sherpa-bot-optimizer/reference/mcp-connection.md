# Conexión MCP — prerequisito mínimo

Esta skill maneja el **servidor MCP de Sherpa**. El onboarding completo por host y la distribución
quedan fuera de scope para v1 (diferidos). Por ahora, el único requisito es:

## Prerequisito

El servidor MCP de Sherpa tiene que estar conectado en tu agente/host, autenticado con un
`everestApiKey` válido, y exponiendo estas tools:

```
whoami, list_bots,
get_chatwoot_bot_conversations, get_chatwoot_contact_conversations,
get_chatwoot_conversation_messages, get_chatwoot_broker_conversations,
get_bot_conversations, get_contact_conversation,
get_conversation_transcript, get_bot_transcripts, get_bot_flows,
get_bot_documentation_metrics, get_bot_resolution_rate, get_bot_mass_sends,
get_broker_rejections, get_rejections_by_broker,
create_flow_draft, update_flow_subflow, update_flow_questions,
update_flow_copies, preview_flow_draft, publish_flow_draft,
get_voice_tone, update_voice_tone,
list_crm_labels, create_crm_label,
add_conversation_labels, remove_conversation_labels
```
> Las `get_chatwoot_*` son las de lectura por defecto (cubren TODOS los bots). Las de Andes
> (`get_bot_conversations`, etc.) solo cubren bots con Andes activo. Si faltan las `get_chatwoot_*`,
> el MCP está desactualizado.
>
> Las cuatro tools de etiquetas (`list_crm_labels`, `create_crm_label`, `add_conversation_labels`,
> `remove_conversation_labels`) habilitan el Modo C (etiquetar conversaciones). El servidor puede
> ocultar las tres que escriben (`MCP_LABEL_WRITE=false`): si faltan, el etiquetado no está
> habilitado; seguí con el resto de la skill y no simules el cambio. Ver
> `reference/conversation-labels.md`.
>
> Las cinco tools de métricas (`get_bot_documentation_metrics`, `get_bot_resolution_rate`,
> `get_bot_mass_sends`, `get_broker_rejections`, `get_rejections_by_broker`) habilitan el Modo B
> (reportar números). Si faltan, el resto de la skill funciona igual: seguí con el Modo A y no le
> prometas números al usuario. `get_rejections_by_broker` además es solo para god_mode: que un
> broker común reciba un error de permisos ahí es lo esperado, no una conexión rota.

Si falta alguna tool requerida, **PARAR**. Al usuario NO le hables de "MCP", "servidor" ni "tools":
usá la fila "Conexión con Sherpa incompleta" de la tabla "Voz hacia el usuario" (SKILL.md) — "Tu
conexión con Sherpa todavía no está lista del todo… escribí a soporte de Sherpa para que la revisen".

## Identidad y alcance

- La autenticación es un Bearer `everestApiKey` que el host le manda al MCP; la skill nunca la
  tiene.
- El alcance se enforcea del lado del servidor: una key de broker (`accessMode: self`) solo puede
  ver y editar los bots de ese broker. Corré `whoami` para confirmar el alcance antes de operar.

## Notas (diferidas a la fase de distribución)

- Los one-liners de conexión por host (Claude Code `claude mcp add`, Custom Connector de claude.ai,
  config MCP de Codex), un repo público de la skill, y un listado en skills.sh / marketplace son
  **trabajo futuro**.
- El everestApiKey se crea/gestiona en el frontend de Sherpa (gateado por permisos). Los flujos de
  alta, rotación y revocación son parte de la fase de distribución, no del build de la skill.
