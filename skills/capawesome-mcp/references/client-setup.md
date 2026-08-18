# MCP Client Setup

Per-client configuration for the hosted Capawesome MCP server.

Replace `YOUR_TOKEN` with the API token created in the [Capawesome Cloud Console](https://console.cloud.capawesome.io/settings/tokens). For a documentation-only setup, drop the `Authorization` header entirely and use `https://mcp.capawesome.io/mcp` as the URL.

All examples use `?toolsets=all`, which registers the documentation tools plus every Capawesome Cloud toolset. Adjust the query parameters as described in `toolsets.md`.

## Claude Code

Run in the project root:

```bash
claude mcp add --transport http capawesome "https://mcp.capawesome.io/mcp?toolsets=all" \
  --header "Authorization: Bearer YOUR_TOKEN"
```

Add `--scope user` to make the server available in every project instead of just the current one:

```bash
claude mcp add --scope user --transport http capawesome "https://mcp.capawesome.io/mcp?toolsets=all" \
  --header "Authorization: Bearer YOUR_TOKEN"
```

Project scope writes the server to `.mcp.json` in the project root, which is usually tracked by Git. Use `--scope user` whenever a token is involved, so the token stays out of the repository.

Verify with:

```bash
claude mcp list
```

Remove with:

```bash
claude mcp remove capawesome
```

## Claude Desktop

The Claude Desktop config file only starts local commands, so use [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) to reach the hosted server.

Open the config file via **Settings → Developer → Edit Config** and add the `capawesome` entry to `mcpServers`:

```json
{
  "mcpServers": {
    "capawesome": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.capawesome.io/mcp?toolsets=all",
        "--header",
        "Authorization:${AUTH_HEADER}"
      ],
      "env": {
        "AUTH_HEADER": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

The header value comes from `env` because a header with spaces is not passed through reliably on the command line.

Restart Claude Desktop after saving.

## Cursor

Add the `capawesome` entry to `.cursor/mcp.json` in the project root, or to `~/.cursor/mcp.json` to use the server in every project:

```json
{
  "mcpServers": {
    "capawesome": {
      "url": "https://mcp.capawesome.io/mcp?toolsets=all",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

`.cursor/mcp.json` in a project is tracked by Git. Use `~/.cursor/mcp.json` whenever a token is involved, or add `.cursor/mcp.json` to `.gitignore`.

Restart Cursor after saving.

## VS Code

Add the `capawesome` entry to `.vscode/mcp.json` in the project root:

```json
{
  "servers": {
    "capawesome": {
      "type": "http",
      "url": "https://mcp.capawesome.io/mcp?toolsets=all",
      "headers": {
        "Authorization": "Bearer ${input:capawesome-token}"
      }
    }
  },
  "inputs": [
    {
      "id": "capawesome-token",
      "type": "promptString",
      "description": "Capawesome Cloud API token",
      "password": true
    }
  ]
}
```

VS Code prompts for the token the first time the server starts and stores it securely, so the file stays safe to commit.

Restart VS Code after saving.

## Other MCP Clients

Any client that supports a remote MCP server over HTTP works. Configure:

- **Transport**: HTTP
- **URL**: `https://mcp.capawesome.io/mcp?toolsets=all`
- **Header**: `Authorization: Bearer YOUR_TOKEN`

For a client that only starts local commands, wrap the server with `mcp-remote` as shown in the Claude Desktop section.
