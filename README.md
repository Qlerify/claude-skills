# Qlerify Plugins

Official Qlerify plugins for AI coding assistants — domain model sync, data export, and more. Works with Claude Code,
Gemini CLI, and Cursor.

## Prerequisites

1. **Qlerify account** with a workflow created
2. **Qlerify MCP server** configured with your API token (see setup per tool below)

## Installation

### Claude Code

Configure MCP server in `~/.claude.json`:

```json
{
  "mcpServers": {
    "qlerify": {
      "type": "url",
      "url": "https://mcp.qlerify.com",
      "headers": {
        "x-api-key": "YOUR_API_TOKEN"
      }
    }
  }
}
```

Install the plugin:

```bash
/plugin marketplace add qlerify/qlerify-plugins
/plugin install mcp-companion@qlerify-plugins
```

After installation, skills are available as `/mcp-companion:workflow-creation`, `/mcp-companion:sync`, and `/mcp-companion:download`.

### Gemini CLI

Install each skill using the `--path` flag:

```bash
gemini skills install https://github.com/qlerify/qlerify-plugins.git --path plugins/mcp-companion/skills/workflow-creation
gemini skills install https://github.com/qlerify/qlerify-plugins.git --path plugins/mcp-companion/skills/sync
gemini skills install https://github.com/qlerify/qlerify-plugins.git --path plugins/mcp-companion/skills/download
```

Configure the Qlerify MCP server in `~/.gemini/settings.json` per
[Gemini CLI docs](https://geminicli.com/docs/cli/mcp/).

### Cursor

1. Open Cursor Settings (`Cmd+Shift+J`)
2. Go to **Rules** > **Add Rule** > **Remote Rule (GitHub)**
3. Enter: `https://github.com/qlerify/qlerify-plugins`

Configure the Qlerify MCP server in Cursor's MCP settings.

Get your API token from the Qlerify UI.

## Plugins

### `mcp-companion`

Teaches AI agents how to effectively use Qlerify's MCP server. Contains the following skills:

#### `workflow-creation`

Guides AI agents through building complete Qlerify workflows — lanes, groups, domain events, entities, commands, read
models, cards, and bounded contexts. Also handles reverse-engineering: when the source is an existing or legacy
codebase, Phase 0 extracts a single DDD aggregate (root entity, related entities, value objects, commands, domain
events, read models, attributes, invariants, external references) and feeds it directly into the rest of the creation
sequence — no separate review step. Includes a full tool reference and a worked e-commerce example.

**Triggers:**

- "create a workflow"
- "build a domain model"
- "set up domain events"
- "add commands and read models to workflow"
- "extract the Order aggregate from shop-api"
- "reverse engineer a domain model from this code"
- "model the Subscription module as a DDD aggregate"
- Any request involving building a Qlerify workflow, adding structural elements, or modeling from existing code

**What it does:**

1. Phase 0 (reverse-engineering only): peels the service/orchestration layer away to expose aggregate-level commands; captures the root entity, related entities, value objects, commands, 1:1 domain events, read models, attributes, invariants, and external references
2. Follows an 8-step creation sequence (events → bounded contexts → entities → commands → read models → domain event schemas → entity field updates → validation)
3. Provides best practices for naming, field modeling, and entity relationships
4. Supports nested fields on commands and read models for related entity references
5. Final adjustments are made via the Phase 4 validation loop — no mid-stream confirmation step

#### `sync`

Syncs your codebase's domain model with Qlerify. Detects entities, commands, and read models in your code and ensures
they match your Qlerify workflow.

**Triggers:**

- "sync domain model"
- "update Qlerify"
- "sync entities"
- After implementing features that change domain objects

**What it does:**

1. Scans your codebase for domain objects (entities, commands, read models)
2. Compares with Qlerify workflow
3. Creates/updates/deletes to keep them in sync

#### `download`

Fast download any Qlerify data directly to files. Uses `curl + jq` to bypass AI processing, making it ~100x faster than
standard MCP tools for large data.

**Triggers:**

- "save to file"
- "download workflow"
- "export"
- Any request to save Qlerify data locally

**What it does:**

1. Fetches data directly via shell commands
2. Pipes to file without AI processing
3. ~1 second instead of 3-5 minutes for large workflows

## Usage Examples

```bash
# Invoke skills directly
/mcp-companion:workflow-creation
/mcp-companion:sync
/mcp-companion:download

# Or just ask naturally - skills trigger automatically
> create a workflow for an e-commerce order process
> sync my domain model with Qlerify
> download the Cart Microservice workflow to workflow.json
> save the swagger spec for my workflow to api.yaml
> extract the Order aggregate from shop-api and build a workflow
```
