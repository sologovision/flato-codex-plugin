# Flato MCP for Codex

Use Flato MCP from Codex to create, inspect, and edit Flato design projects.

This plugin connects Codex to the production Flato MCP service:

```text
https://api.flato.ai/api/mcp/editor
```

## Requirements

- Codex App
- A Flato account
- Permission to authorize Flato MCP in your browser

## Install In Codex App

1. Open Codex App.
2. Select **Plugins** in the left sidebar.
3. Open the plugin source dropdown.
4. Select **Add more**.
5. In **Source**, enter:

```text
sologovision/flato-codex-plugin
```

6. In **Git ref**, enter:

```text
main
```

7. Leave **Sparse path** empty.
8. Select **Add marketplace**.
9. Select the **Flato** source from the source dropdown.
10. Install **Flato MCP Operator**.
11. Restart Codex App or open a new Codex thread.
12. Complete the Flato OAuth authorization when prompted.

## Update In Codex App

When a new plugin version is published:

1. Open Codex App.
2. Select **Plugins** in the left sidebar.
3. Select the **Flato** source from the source dropdown.
4. Refresh or re-add the marketplace with:

```text
sologovision/flato-codex-plugin
```

5. Reinstall or update **Flato MCP Operator**.
6. Restart Codex App or open a new Codex thread.

## Optional CLI Install

If you use Codex CLI, you can install the same plugin with:

```bash
codex plugin marketplace add sologovision/flato-codex-plugin
codex plugin add flato-mcp-operator@flato
```

If Flato OAuth does not start automatically:

```bash
codex mcp login flato-editor
```

## First Use

Open a new Codex thread and ask:

```text
Use Flato MCP and call flato_whoami first.
```

After authorization, you can ask Codex to work with Flato designs, for example:

```text
Use Flato MCP to create a one-page product launch visual.
```

```text
Use Flato MCP to edit this Flato project: https://www.flato.ai/editor/PROJECT_ID
```

```text
Use Flato MCP to revise the selected block in my open Flato editor.
```

## How It Works

The plugin gives Codex the Flato MCP server and a Flato-specific workflow. For
design work, Codex is guided to read Flato canvas fundamentals, prompt-skill
resources, brand kits, CJK font resources, and default visual themes when
relevant. It then targets real page and block IDs, applies design edits, and
verifies results before reporting completion.

## Troubleshooting

If the plugin is installed but Codex does not see `flato_*` tools, restart
Codex App and open a new thread.

If authorization does not start automatically, run:

```bash
codex mcp login flato-editor
```

If Codex says the editor is not writable, open or refresh the Flato editor page
in your browser, then ask Codex to check the project status again.
