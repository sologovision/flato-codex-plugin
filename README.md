# Flato Codex Plugin

Flato MCP Operator is a Codex plugin for using Flato MCP from Codex.

It bundles:

- a Flato MCP server configuration for `https://api.flato.ai/api/mcp/editor`
- a Codex skill that tells Codex to actively read Flato MCP resources, prompts,
  design skills, brand kits, project state, feedback, and visual QA results
- marketplace metadata so Codex App can discover and install the plugin

## Install From Codex App

1. Open Codex App.
2. Open **Plugins** from the left sidebar.
3. Open the marketplace/source dropdown.
4. Select **Add more**.
5. In **Source**, enter:

```text
sologovision/flato-codex-plugin
```

or:

```text
git@github.com:sologovision/flato-codex-plugin.git
```

6. In **Git ref**, enter:

```text
main
```

7. Leave **Sparse path** empty. This repository stores
   `.agents/plugins/marketplace.json` and `plugins/flato-mcp-operator/` at the
   repository root.
8. Select **Add marketplace**.
9. Choose the **Flato** marketplace from the source dropdown.
10. Install **Flato MCP Operator**.
11. Restart Codex App or open a new Codex thread.
12. Complete Flato MCP OAuth when prompted.

## Install From Codex CLI

Codex CLI installation is optional, but useful for technical users and
automation. Installing Codex App alone does not guarantee that the `codex`
command is available in a terminal.

```bash
codex plugin marketplace add sologovision/flato-codex-plugin
codex plugin add flato-mcp-operator@flato
```

If OAuth does not start automatically:

```bash
codex mcp login flato-editor
```

## Verify

Start a new Codex thread and ask:

```text
Use Flato MCP and call flato_whoami first.
```

Expected behavior:

- Codex loads the Flato MCP Operator skill.
- Codex connects to the `flato-editor` MCP server after OAuth.
- Codex calls `flato_whoami` before editing.
- For design work, Codex should actively read Flato MCP protocol, prompt-skill,
  and brand-kit resources before writing.

## Repository Structure

```text
.agents/
  plugins/
    marketplace.json
plugins/
  flato-mcp-operator/
    .codex-plugin/
      plugin.json
    .mcp.json
    skills/
      flato-mcp-operator/
        SKILL.md
```

## Production Endpoint

The bundled MCP server points to production:

```text
https://api.flato.ai/api/mcp/editor
```

For staging, publish a separate branch or plugin variant with `.mcp.json`
pointing to:

```text
https://test1-api.flato.ai/api/mcp/editor
```

## What The Skill Enforces

For Flato MCP design work, the plugin tells Codex to:

- read `flato://protocol/creative-v1`
- read `flato://prompt-skills`
- choose and read the matching design skill resource
- check `flato://brand-kits` for brand-sensitive work
- call `flato_whoami`
- establish or create the target project
- wait until `flato_get_project_status` reports `canWrite=true`
- call `flato_get_design_context` before writing
- use explicit `pageId` and `blockId` values
- inspect `mcpFeedback` and `layoutIssueSummary` after writes
- export and inspect important output with `flato_export_to_png` and
  `flato_understand_image`

