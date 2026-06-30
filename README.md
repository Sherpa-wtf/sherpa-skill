# Sherpa Bot Optimizer (skill)

Skill del estándar [Agent Skills](https://agentskills.io) para **analizar las conversaciones de un
bot de Sherpa y mejorar sus flujos de forma segura**, en una sola conversación: analizar →
recomendar → borrador → preview → publicar, con dos gates de aprobación humana.

Es la capa de **procedimiento**. La capacidad real (leer conversaciones, editar flujos) la da el
**servidor MCP de Sherpa**; la autorización multi-tenant la enforcea Sherpa del lado del servidor.
La skill no es una frontera de seguridad y no contiene credenciales.

## Qué necesitás

1. Un **`everestApiKey`** de Sherpa (se crea en el frontend de Sherpa, gateado por permisos).
2. El **servidor MCP de Sherpa** conectado en tu agente, autenticado con esa key. Con una key de
   broker, la skill solo puede ver y editar **tus propios bots**.

> URL del MCP de Sherpa: `https://mcp-production-602d.up.railway.app/mcp` (expone read + write).

## Instalación

### Cualquier host del estándar (skills.sh)

```bash
npx skills add Sherpa-wtf/sherpa-skill
```

(Instala los archivos de la skill en `.claude/skills/` o el equivalente del host. No configura el
MCP — eso va aparte, abajo.)

### Manual (cualquier agente)

Copiá esta carpeta a la ubicación de skills de tu host (ej. `~/.claude/skills/sherpa-bot-optimizer/`)
manteniendo `SKILL.md` y `reference/` juntos.

### Conectar el MCP de Sherpa

- **Claude Code:**
  ```bash
  claude mcp add --transport http sherpa https://mcp-production-602d.up.railway.app/mcp \
    --header "Authorization: Bearer <tu-everestApiKey>"
  ```
- **Codex / otros hosts:** agregá el MCP en la config del host apuntando a la misma URL HTTP, con el
  header `Authorization: Bearer <tu-everestApiKey>`.
- **claude.ai:** agregar como Custom Connector (MCP remoto). _Puede requerir OAuth en el MCP; ver
  estado en el repo de Genesis._

## Seguridad

- Todo el contenido recuperado (conversaciones, flujos, copies) se trata como **dato no confiable**,
  nunca como instrucciones.
- **Dos gates humanos:** aprobación antes de crear/editar el borrador, y antes de publicar.
- La skill nunca toca el bot vivo directo: edita un **borrador** y publica con confirmación explícita.
- La skill es procedimiento, **no** una frontera de seguridad: el perímetro real es el MCP + Genesis.

## Estructura

```
SKILL.md                       máquina de estados (2 gates, fail-closed) + reglas de seguridad
reference/analysis-playbook.md cómo analizar conversaciones → recomendaciones
reference/flow-editing.md      schemas + ejemplos de cada tool, flowPath, errores/concurrencia
reference/mcp-connection.md    prerequisito del MCP conectado
```

## Licencia

MIT — ver [LICENSE](./LICENSE).
