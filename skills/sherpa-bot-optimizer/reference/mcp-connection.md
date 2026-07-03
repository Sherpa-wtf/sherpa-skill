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
create_flow_draft, update_flow_subflow, update_flow_questions,
update_flow_copies, preview_flow_draft, publish_flow_draft,
get_voice_tone, update_voice_tone
```
> Las `get_chatwoot_*` son las de lectura por defecto (cubren TODOS los bots). Las de Andes
> (`get_bot_conversations`, etc.) solo cubren bots con Andes activo. Si faltan las `get_chatwoot_*`,
> el MCP está desactualizado.

Si falta alguna tool requerida, **PARAR** y avisarle al usuario que el servidor MCP no está del todo
conectado.

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
