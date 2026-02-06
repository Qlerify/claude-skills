# Qlerify Claude Code Skills

Claude Code skills for integrating with [Qlerify](https://qlerify.com) - keep your domain model in sync with your
codebase.

## Prerequisites

1. **Qlerify account** with a workflow created
2. **Qlerify MCP server** configured in Claude Code with your API token

### Configure MCP Server

Add to your Claude Code MCP settings (`~/.claude/settings.local.json`):

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

Get your API token from the Qlerify UI.

## Installation

In Claude Code, run:

```bash
# Add the Qlerify marketplace
/plugin marketplace add qlerify/claude-skills

# Install the plugin
/plugin install qlerify@qlerify-skills
```

After installation, skills are available as `/qlerify:sync` and `/qlerify:download`.

## Skills

### `sync`

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

### `download`

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
/qlerify:sync
/qlerify:download

# Or just ask naturally - skills trigger automatically
> sync my domain model with Qlerify
> download the Cart Microservice workflow to workflow.json
> save the swagger spec for my workflow to api.yaml
```

