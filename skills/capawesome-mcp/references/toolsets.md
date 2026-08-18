# Toolsets

The Capawesome MCP server groups its tools into toolsets. Select them with the `toolsets` query parameter on the server URL.

Every tool carries MCP annotations that mark it as read-only or destructive, which is what clients use to decide when to ask for confirmation before running a tool.

## Selecting Toolsets

| URL | Registers |
| --- | --------- |
| `https://mcp.capawesome.io/mcp` | `docs` only. No token needed. |
| `https://mcp.capawesome.io/mcp?toolsets=all` | `docs` plus every Capawesome Cloud toolset. Requires a token. |
| `https://mcp.capawesome.io/mcp?toolsets=cloud` | Every Capawesome Cloud toolset, without `docs`. Requires a token. |
| `https://mcp.capawesome.io/mcp?toolsets=docs,cloud-apps,cloud-app-builds` | Only the named toolsets. |
| `https://mcp.capawesome.io/mcp?toolsets=all&readonly=true` | Read-only tools only. Nothing can be created, changed, or deleted. |

Every tool takes up context in the client, and fewer tools makes it pick the right one more often. Name the specific toolsets needed for the task instead of registering all 85 tools, and keep `docs` for documentation search.

Quote the URL when passing it on the command line, otherwise the shell interprets the query parameters.

## Documentation

Registered by default. Needs no token.

| Toolset | Tools | Description |
| ------- | ----- | ----------- |
| `docs` | 3 | `search_docs`, `get_doc_page`, and `list_blog_posts` — search and read the Capawesome documentation and blog. |

## Capawesome Cloud

Opt in with `?toolsets=cloud` or by name. Each requires an API token. A toolset named `cloud-app-*` acts on a single app, so its tools all take an app ID.

| Toolset | Tools | Description |
| ------- | ----- | ----------- |
| `cloud-app-automations` | 5 | Manage automations that build on Git events. |
| `cloud-app-builds` | 5 | Trigger app builds and share them. |
| `cloud-app-certificates` | 5 | Manage signing certificates and provisioning profiles. |
| `cloud-app-channels` | 5 | Manage live update channels. |
| `cloud-app-configurations` | 5 | Manage native configurations. |
| `cloud-app-deployments` | 4 | Deploy builds to channels and app stores, and roll back. |
| `cloud-app-destinations` | 5 | Manage app store destinations. |
| `cloud-app-devices` | 5 | Inspect devices and control which update they receive. |
| `cloud-app-environments` | 5 | Manage build environments, variables, and secrets. |
| `cloud-app-repository` | 3 | Link an app to a Git repository, and see which one it uses. |
| `cloud-apps` | 4 | Create, list, and update apps. |
| `cloud-git` | 6 | Manage the Git connections of an organization and browse their repositories. |
| `cloud-jobs` | 4 | Track, cancel, and diagnose builds and deployments. |
| `cloud-license-keys` | 6 | Manage license keys for Capawesome Insiders and Enterprise SDKs. |
| `cloud-organizations` | 10 | Identify the account, list organizations (and the IDs other tools need), manage members and invitations. |
| `cloud-teams` | 5 | Manage teams and their apps and members. |

Toolset names and tool counts change as the server grows. For the current list, call `get_doc_page` with `https://capawesome.io/docs/ai/mcp/` and read the Toolsets and Tool reference sections.

## Recipes

Pick the smallest set that covers the task.

| Task | Toolsets |
| ---- | -------- |
| Look up plugin, CLI, or Cloud documentation | `docs` |
| Ship a live update | `docs,cloud-apps,cloud-app-channels,cloud-app-deployments,cloud-jobs` |
| Build a native app in the cloud | `docs,cloud-apps,cloud-app-builds,cloud-app-certificates,cloud-app-environments,cloud-jobs` |
| Publish to the App Store or Google Play | `docs,cloud-apps,cloud-app-builds,cloud-app-destinations,cloud-app-deployments,cloud-jobs` |
| Debug a failed build or deployment | `docs,cloud-jobs,cloud-app-builds,cloud-app-deployments` |
| Set up Git-triggered automations | `docs,cloud-apps,cloud-git,cloud-app-repository,cloud-app-automations` |
| Investigate why a device is not updating | `docs,cloud-apps,cloud-app-channels,cloud-app-devices` |
| Shared or unattended setup | Any of the above plus `readonly=true` |

## Frequently Used Tools

| Tool | Toolset | Use for |
| ---- | ------- | ------- |
| `search_docs` | `docs` | Finding the documentation page for a plugin, CLI command, or Cloud feature. |
| `get_doc_page` | `docs` | Reading a full page before writing code against an API. |
| `cloud_get_current_user` | `cloud-organizations` | Verifying the token works and identifying the account. |
| `cloud_list_organizations` | `cloud-organizations` | Getting the organization ID that most other tools require. |
| `cloud_list_apps` | `cloud-apps` | Getting the app ID that every `cloud-app-*` tool requires. |
| `cloud_create_app_build` | `cloud-app-builds` | Starting a build from a Git ref or a ZIP URL. |
| `cloud_create_app_deployment` | `cloud-app-deployments` | Deploying a build to a channel or a store destination. Deploying an earlier build is how a rollback is performed. |
| `cloud_diagnose_job` | `cloud-jobs` | Fetching the log tail and an AI failure summary for a failed job. |
| `cloud_probe_app_device` | `cloud-app-devices` | Checking which live update a specific device would receive right now. |
