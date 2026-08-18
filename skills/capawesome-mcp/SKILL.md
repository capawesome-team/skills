---
name: capawesome-mcp
description: "Guides the agent through connecting an MCP client to the hosted Capawesome MCP server, which exposes always-current Capawesome documentation search and the Capawesome Cloud management API. Covers setup for Claude Code, Claude Desktop, Cursor, and VS Code, API token creation, toolset selection, read-only mode, secret handling, verification, and troubleshooting. Do not use for installing Capacitor plugins, migrating apps or plugins to a newer version, running Capawesome CLI commands, or MCP servers other than Capawesome."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capawesome-mcp
---

# Capawesome MCP Server

Connect an MCP client to the hosted Capawesome MCP server for always-current documentation search and Capawesome Cloud management.

The server is hosted by Capawesome — there is nothing to install and nothing to keep up to date:

```
https://mcp.capawesome.io/mcp
```

It serves two things:

- **Documentation** — search and read the current Capawesome documentation and blog. No token required. This is always more up to date than any reference file bundled with a skill.
- **Capawesome Cloud** — manage apps, builds, deployments, channels, devices, environments, certificates, and jobs. Requires an API token.

## Prerequisites

1. An MCP client: Claude Code, Claude Desktop, Cursor, VS Code, or any other client that implements the [Model Context Protocol](https://modelcontextprotocol.io).
2. For Capawesome Cloud tools only: a [Capawesome Cloud](https://cloud.capawesome.io) account and an API token.

## Agent Behavior

- **Guide step-by-step.** Walk the user through the process one step at a time. Never present multiple unrelated questions at once.
- **Auto-detect before asking.** Detect the MCP client from the project and environment before asking the user which one they use.
- **Never ask for a token unless Cloud tools are wanted.** The documentation tools work without authentication. Only start the token flow when the user wants Capawesome Cloud management.
- **Never write a token into a file that is tracked by Git.** See [Handling Tokens](#handling-tokens).

## Procedures

### Step 1: Determine Which Toolsets Are Needed

Ask the user which of the following they want, and register only that:

1. **Documentation only** — search and read the Capawesome documentation. No token needed. Use `?toolsets=docs` (the default when no query parameter is given).
2. **Documentation and Capawesome Cloud** — the full surface. Requires a token. Use `?toolsets=all`. This is what most setups want.
3. **A specific subset** — read `references/toolsets.md` and pick the toolsets matching the user's task. Fewer tools take up less context and make the client pick the right tool more often.

If the user is unsure, recommend option 2.

Additionally, ask whether the server should be **read-only**. Appending `?readonly=true` registers read-only tools only, so nothing can be created, changed, or deleted. Recommend it for shared, unattended, or CI setups.

### Step 2: Create an API Token

Skip this step if the user chose documentation only in Step 1.

1. Tell the user to open the [Capawesome Cloud Console](https://console.cloud.capawesome.io/settings/tokens) and create an API token.
2. Warn the user that the token is shown **only once** and must be copied immediately.
3. **Wait** for the user to confirm they have the token before continuing.

A token acts on behalf of the account that created it, so the server can do whatever that account can do across its organizations and apps. Requests go through the same Cloud API as the Capawesome CLI, which means the same permissions, network restrictions, and rate limits apply.

### Step 3: Detect the MCP Client

Detect the client instead of asking, by checking the project in this order:

1. `.mcp.json` or `.claude/` in the project root → **Claude Code**
2. `.cursor/mcp.json` or `.cursor/` in the project root → **Cursor**
3. `.vscode/` in the project root → **VS Code**

If none of these exist, or if more than one matches, ask the user which client to configure.

### Step 4: Add the Server

Read `references/client-setup.md` and apply the section for the detected client. Build the server URL from the choices made in Step 1:

| Choice | URL |
| ------ | --- |
| Documentation only | `https://mcp.capawesome.io/mcp` |
| Documentation and Capawesome Cloud | `https://mcp.capawesome.io/mcp?toolsets=all` |
| Capawesome Cloud only | `https://mcp.capawesome.io/mcp?toolsets=cloud` |
| Specific toolsets | `https://mcp.capawesome.io/mcp?toolsets=docs,cloud-apps,cloud-app-builds` |
| Read-only | Append `&readonly=true`, or `?readonly=true` when there is no other parameter |

Always quote the URL when passing it on the command line, otherwise the shell interprets the query parameters.

Omit the `Authorization` header entirely for a documentation-only setup.

### Step 5: Restart the Client and Verify

1. Tell the user to restart their MCP client so it picks up the new server.
2. Verify the documentation tools by asking the client to run `search_docs` with a query such as `live update rollback`.
3. If Cloud tools were registered, verify them by asking the client to run `cloud_get_current_user`. It returns the account the token belongs to.
4. If either call fails, go to [Error Handling](#error-handling).

### Step 6: Use the Server

Once connected, prefer the MCP tools over bundled reference files and over model knowledge whenever the topic is Capawesome:

- **`search_docs`** — search the documentation and blog by keyword. Start here; snippets are short by design.
- **`get_doc_page`** — read a full page as Markdown, using a URL from `search_docs`. Read the whole page before writing code against a plugin API, a CLI command, or a Cloud workflow — search snippets regularly omit required configuration steps.
- **`list_blog_posts`** — list the most recent blog posts, for announcements and release notes.
- **`cloud_*`** — manage Capawesome Cloud. Read `references/toolsets.md` for the toolset each tool belongs to.

## Handling Tokens

A few Cloud tools accept sensitive values — environment secrets, signing certificates, and app store credentials. Anything passed to a tool becomes part of the conversation and is sent to the AI provider, and it may end up in chat history or logs outside the user's control.

Apply these rules:

- **Never commit a token.** `.mcp.json`, `.cursor/mcp.json`, and `claude_desktop_config.json` in a project are tracked by Git. Prefer a setup that keeps the token out of the file: the VS Code `inputs` prompt, an environment variable, or the user-scoped configuration.
- **Set production secrets with the Capawesome CLI or the Console**, not through MCP tools. Use the MCP server for reading and for values the user would be fine seeing in a transcript.
- **If a token may have leaked**, tell the user to revoke it in the [Capawesome Cloud Console](https://console.cloud.capawesome.io/settings/tokens) and create a new one.

## Error Handling

- **Server not listed after setup**: The client was not restarted. Restart it. In Claude Code, run `claude mcp list` to confirm the server is registered.
- **`401 Unauthorized`**: The token is missing, malformed, or revoked. Verify the header value is `Bearer <TOKEN>` including the space, and that the token still exists in the Console.
- **Documentation tools work but no `cloud_*` tools appear**: The URL is missing `?toolsets=all` (or a `cloud-*` toolset), or the `Authorization` header was not sent. Check the URL and the header.
- **Only read tools appear**: The URL contains `readonly=true`. Remove it to register write tools.
- **Shell reports "no matches found" or drops the query parameters**: The URL was not quoted on the command line. Wrap it in double quotes.
- **Claude Desktop cannot reach the server**: The Claude Desktop config file only starts local commands. Use `mcp-remote` as shown in `references/client-setup.md`.
- **`403 Forbidden` on Cloud tools**: The account lacks permission for that organization or app, or an organization network restriction blocks the request. Verify with `cloud_get_current_user` and `cloud_list_organizations`.
- **Rate limited**: The same rate limits as the Cloud API apply. Retry after a short wait.

## Related Skills

- **`capawesome-cloud`** — For setting up native builds, live updates, and app store publishing.
- **`capawesome-cli`** — For the Capawesome CLI, which covers the same Cloud API from the terminal and CI/CD.
- **`capacitor-plugins`** — For installing and configuring Capacitor plugins, including the Capawesome plugins documented by this server.
- **`capacitor-expert`** — For a broad Capacitor reference covering plugins, framework integration, and Capawesome Cloud.
