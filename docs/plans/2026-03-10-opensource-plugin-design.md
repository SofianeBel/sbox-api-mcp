# Design: sbox-api-mcp — Open-source + Plugin Claude Code

## Overview

Transform the existing local MCP server into a published npm package (`sbox-api-mcp`) and Claude Code plugin. GitHub repo: `sofianebel/sbox-api-mcp`.

## Installation Methods

```bash
# Claude Code plugin (one click)
claude plugin add sofianebel/sbox-api-mcp

# Manual via npm
claude mcp add sbox-api -- npx -y sbox-api-mcp
```

## Changes Required

### 1. Portable Cache Directory

Current: relative path `../../cache` from dist/data/loader.js — breaks when installed via npm.

New: `~/.sbox-api-mcp/` via `os.homedir()`, overridable with `SBOX_CACHE_DIR` env var.

```
~/.sbox-api-mcp/
├── config.json     # { url, etag, lastFetched }
└── api-data.json   # Cached 8.6 MB API JSON
```

### 2. npm Package

- Rename to `sbox-api-mcp`
- Add `"bin": { "sbox-api-mcp": "dist/index.js" }`
- Add shebang `#!/usr/bin/env node` to index.ts
- Publish to npm registry

### 3. Claude Code Plugin

```
.claude-plugin/plugin.json
{
  "name": "sbox-api",
  "description": "S&box game engine API documentation lookup for Claude Code",
  "author": { "name": "sofianebel" }
}

.mcp.json
{
  "sbox-api": {
    "command": "npx",
    "args": ["-y", "sbox-api-mcp"]
  }
}
```

### 4. Environment Variables

- `SBOX_API_URL` — Override default API JSON URL
- `SBOX_CACHE_DIR` — Override cache directory

### 5. Files to Add

- `README.md` — Installation, usage, tools reference, contributing
- `LICENSE` — MIT
- `.claude-plugin/plugin.json` — Plugin metadata
- `.mcp.json` — MCP server config for plugin

## Final Repo Structure

```
sbox-api-mcp/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
├── src/
│   ├── index.ts
│   ├── types.ts
│   ├── data/
│   │   ├── loader.ts
│   │   └── indexer.ts
│   └── tools/
│       ├── search-types.ts
│       ├── get-type.ts
│       ├── search-members.ts
│       ├── list-namespaces.ts
│       ├── search-docs.ts
│       └── update-source.ts
├── dist/
├── package.json
├── tsconfig.json
├── .gitignore
├── LICENSE
└── README.md
```

## What Does NOT Change

- 6 MCP tools (search_types, get_type, search_members, list_namespaces, search_docs, update_api_source)
- Fuse.js indexing logic
- ETag-based caching mechanism
- S&box JSON format parsing
