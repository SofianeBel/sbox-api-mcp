# sbox-api-mcp Open-Source + Plugin Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Transform the local sbox-mcp-server into a published npm package and Claude Code plugin at `sofianebel/sbox-api-mcp`.

**Architecture:** Rename package to `sbox-api-mcp`, make cache directory portable via `os.homedir()`, add npm bin entry + shebang, add Claude Code plugin files, publish to npm and GitHub.

**Tech Stack:** Node.js, TypeScript, MCP SDK, Fuse.js, npm, GitHub CLI

---

### Task 1: Initialize Git Repo

**Files:**
- Modify: `C:\Users\sifly\Documents\sbox-mcp-server\.gitignore`

**Step 1: Init git and make initial commit of current working state**

```bash
cd C:/Users/sifly/Documents/sbox-mcp-server
git init
git add .gitignore package.json package-lock.json tsconfig.json src/
git commit -m "feat: initial sbox MCP server with 6 API tools"
```

Expected: clean commit with all source files.

---

### Task 2: Rename Package + Add bin Entry

**Files:**
- Modify: `C:\Users\sifly\Documents\sbox-mcp-server\package.json`

**Step 1: Update package.json**

Replace the full package.json with:

```json
{
  "name": "sbox-api-mcp",
  "version": "1.0.0",
  "description": "MCP server for S&box game engine API documentation lookup. Provides 6 tools for searching types, methods, properties, and documentation.",
  "type": "module",
  "bin": {
    "sbox-api-mcp": "dist/index.js"
  },
  "files": [
    "dist",
    ".claude-plugin",
    ".mcp.json"
  ],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build",
    "start": "node dist/index.js",
    "dev": "tsx src/index.ts"
  },
  "keywords": [
    "mcp",
    "sbox",
    "sandbox",
    "game-engine",
    "api",
    "documentation",
    "claude",
    "claude-code"
  ],
  "author": "sofianebel",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/sofianebel/sbox-api-mcp.git"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.1",
    "fuse.js": "^7.1.0",
    "zod": "^3.25.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.7.0"
  }
}
```

Key changes: `name` → `sbox-api-mcp`, added `bin`, `files`, `keywords`, `author`, `license`, `repository`, `prepublishOnly`.

**Step 2: Add shebang to index.ts**

Add `#!/usr/bin/env node` as the very first line of `src/index.ts` (before any imports). This is required for the `bin` entry to work on Unix systems.

**Step 3: Commit**

```bash
git add package.json src/index.ts
git commit -m "feat: rename to sbox-api-mcp, add bin entry and npm metadata"
```

---

### Task 3: Make Cache Directory Portable

**Files:**
- Modify: `C:\Users\sifly\Documents\sbox-mcp-server\src\data\loader.ts`

**Step 1: Replace hardcoded cache path with cross-platform home directory**

Replace lines 1-11 of `loader.ts` with:

```typescript
import { readFileSync, writeFileSync, existsSync, mkdirSync } from 'node:fs'
import { join } from 'node:path'
import { homedir } from 'node:os'
import type { SboxApiData, CacheConfig } from '../types.js'

const CACHE_DIR = process.env.SBOX_CACHE_DIR || join(homedir(), '.sbox-api-mcp')
const CONFIG_PATH = join(CACHE_DIR, 'config.json')
const DATA_PATH = join(CACHE_DIR, 'api-data.json')

const DEFAULT_URL = process.env.SBOX_API_URL || 'https://cdn.sbox.game/releases/2026-03-09-03-14-54.zip.json'
```

Changes:
- Remove `dirname`, `fileURLToPath` imports (no longer needed)
- Add `homedir` import from `node:os`
- `CACHE_DIR` uses `~/.sbox-api-mcp/` by default, overridable via `SBOX_CACHE_DIR`
- `DEFAULT_URL` overridable via `SBOX_API_URL`

The rest of the file stays identical.

**Step 2: Update .gitignore**

Remove `cache/` line (no longer relevant — cache is now in home dir). Add `stderr.log`.

```
node_modules/
dist/
stderr.log
```

**Step 3: Delete the local cache/ directory**

```bash
rm -rf cache/
rm -f stderr.log
```

**Step 4: Build and test**

```bash
npm run build
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' | node dist/index.js 2>/dev/null
```

Expected: JSON response with `serverInfo.name: "sbox-api"`. Cache should be created at `~/.sbox-api-mcp/`.

**Step 5: Commit**

```bash
git add src/data/loader.ts .gitignore
git commit -m "feat: portable cache directory (~/.sbox-api-mcp/), env var overrides"
```

---

### Task 4: Add Claude Code Plugin Files

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.mcp.json`

**Step 1: Create plugin.json**

```json
{
  "name": "sbox-api",
  "description": "S&box game engine API documentation lookup for Claude Code. Search types, methods, properties, and documentation across the entire S&box API.",
  "author": {
    "name": "sofianebel"
  }
}
```

**Step 2: Create .mcp.json**

```json
{
  "sbox-api": {
    "command": "npx",
    "args": ["-y", "sbox-api-mcp"]
  }
}
```

**Step 3: Commit**

```bash
git add .claude-plugin/plugin.json .mcp.json
git commit -m "feat: add Claude Code plugin configuration"
```

---

### Task 5: Add LICENSE and README

**Files:**
- Create: `LICENSE`
- Create: `README.md`

**Step 1: Create MIT LICENSE**

Standard MIT license with `Copyright (c) 2026 sofianebel`.

**Step 2: Create README.md**

Content should include:
- **Title + badge** — `sbox-api-mcp` with npm badge
- **What it does** — One paragraph: MCP server for S&box API, 1807 types, 15k+ members, fuzzy search
- **Installation** — 3 methods: Claude Code plugin, manual MCP add, npx
- **Tools reference** — Table of 6 tools with description and example params
- **Configuration** — Env vars: `SBOX_API_URL`, `SBOX_CACHE_DIR`
- **Updating API data** — How to use `update_api_source` tool when S&box updates
- **License** — MIT

**Step 3: Commit**

```bash
git add LICENSE README.md
git commit -m "docs: add README and MIT license"
```

---

### Task 6: Create GitHub Repo and Push

**Step 1: Create remote repo**

```bash
cd C:/Users/sifly/Documents/sbox-mcp-server
gh repo create sofianebel/sbox-api-mcp --public --source=. --description "MCP server for S&box game engine API documentation lookup" --push
```

**Step 2: Verify**

```bash
gh repo view sofianebel/sbox-api-mcp --web
```

---

### Task 7: Publish to npm

**Step 1: Verify npm auth**

```bash
npm whoami
```

If not logged in: `npm login`

**Step 2: Dry-run publish**

```bash
npm publish --dry-run
```

Verify the package contents look correct (dist/, .claude-plugin/, .mcp.json, README, LICENSE, package.json).

**Step 3: Publish**

```bash
npm publish
```

**Step 4: Verify installation works**

```bash
npx sbox-api-mcp --help 2>&1 || echo "Server starts (no --help flag, expected)"
```

---

### Task 8: Update Local Claude Code Config

**Step 1: Update local MCP to use npx instead of hardcoded path**

Update `~/.claude.json` mcpServers entry from:
```json
"sbox-api": {
  "command": "cmd",
  "args": ["/c", "node", "C:\\Users\\sifly\\Documents\\sbox-mcp-server\\dist\\index.js"]
}
```
To:
```json
"sbox-api": {
  "command": "npx",
  "args": ["-y", "sbox-api-mcp"]
}
```

**Step 2: Verify**

```bash
claude mcp list 2>&1 | grep sbox-api
```

Expected: `sbox-api: npx -y sbox-api-mcp - ✓ Connected`
