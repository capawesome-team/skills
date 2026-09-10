# MCP Client Setup

Per-client configuration for the hosted Capacitor MCP server.

The server needs no account and no token, so no configuration carries a secret and every file below is safe to commit:

```
https://capacitor-mcp.capawesome.io/mcp
```

Register the server under the name `capacitor` in every client.

## Claude Code

Run in the project root:

```bash
claude mcp add --transport http capacitor https://capacitor-mcp.capawesome.io/mcp
```

Add `--scope user` to make the server available in every project instead of just the current one:

```bash
claude mcp add --scope user --transport http capacitor https://capacitor-mcp.capawesome.io/mcp
```

Project scope writes the server to `.mcp.json` in the project root, which shares it with everyone working on the repository.

Verify with:

```bash
claude mcp list
```

Remove with:

```bash
claude mcp remove capacitor
```

## Claude Desktop

Tell the user to open **Settings → Connectors → Add custom connector** and enter:

- **Name**: `Capacitor`
- **URL**: `https://capacitor-mcp.capawesome.io/mcp`

The same connector works on [claude.ai](https://claude.ai).

## Cursor

Add the `capacitor` entry to `.cursor/mcp.json` in the project root, or to `~/.cursor/mcp.json` to use the server in every project:

```json
{
  "mcpServers": {
    "capacitor": {
      "url": "https://capacitor-mcp.capawesome.io/mcp"
    }
  }
}
```

Restart Cursor after saving.

## VS Code

Add the `capacitor` entry to `.vscode/mcp.json` in the project root:

```json
{
  "servers": {
    "capacitor": {
      "type": "http",
      "url": "https://capacitor-mcp.capawesome.io/mcp"
    }
  }
}
```

Restart VS Code after saving.

## Windsurf

Add the `capacitor` entry to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "capacitor": {
      "serverUrl": "https://capacitor-mcp.capawesome.io/mcp"
    }
  }
}
```

Restart Windsurf after saving.

## Zed

Add the `capacitor` entry to the Zed `settings.json`:

```json
{
  "context_servers": {
    "capacitor": {
      "url": "https://capacitor-mcp.capawesome.io/mcp"
    }
  }
}
```

Restart Zed after saving.

## Other MCP Clients

Any client that supports a remote MCP server over HTTP works. Configure:

- **Transport**: HTTP
- **URL**: `https://capacitor-mcp.capawesome.io/mcp`
- **Header**: none

A client that only starts local commands can run the [`@capawesome/capacitor-mcp`](https://www.npmjs.com/package/@capawesome/capacitor-mcp) package, which proxies stdio to the hosted server:

```json
{
  "mcpServers": {
    "capacitor": {
      "command": "npx",
      "args": ["-y", "@capawesome/capacitor-mcp"]
    }
  }
}
```

This requires Node.js 22 or later. The proxy takes no configuration; `CAPACITOR_MCP_URL` overrides the endpoint for local development only.
