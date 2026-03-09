# sbox-api-mcp

MCP server for [S&box](https://sbox.game) game engine API documentation lookup. Indexes **1800+ types**, **15,000+ members**, and **8,000+ documentation entries** from the S&box API, providing fast fuzzy search directly from Claude Code.

## Installation

### Claude Code Plugin (recommended)

```bash
claude plugin add sofianebel/sbox-api-mcp
```

### Manual MCP Setup

```bash
claude mcp add sbox-api -- npx -y sbox-api-mcp
```

### npx (direct)

```bash
npx sbox-api-mcp
```

## Tools

| Tool | Description | Example |
|------|-------------|---------|
| `search_types` | Fuzzy search types (classes, structs, enums, interfaces) | `search_types({ query: "GameObject" })` |
| `get_type` | Full details of a specific type (methods, properties, fields) | `get_type({ name: "Sandbox.GameObject" })` |
| `search_members` | Search methods/properties across all types | `search_members({ query: "Position", kind: "property" })` |
| `list_namespaces` | List all API namespaces with type counts | `list_namespaces({ filter: "Audio" })` |
| `search_docs` | Full-text search in documentation summaries | `search_docs({ query: "play sound" })` |
| `update_api_source` | Update API data URL when S&box releases a new version | `update_api_source({ url: "https://cdn.sbox.game/releases/..." })` |

## Configuration

Environment variables (optional):

| Variable | Description | Default |
|----------|-------------|---------|
| `SBOX_API_URL` | Override the S&box API JSON URL | Latest known release |
| `SBOX_CACHE_DIR` | Override cache directory | `~/.sbox-api-mcp/` |

## Updating API Data

When S&box releases a new version, the API JSON URL changes. You can update it in two ways:

1. **From Claude Code:** Use the `update_api_source` tool with the new URL
2. **Environment variable:** Set `SBOX_API_URL` to the new URL

The URL format is: `https://cdn.sbox.game/releases/YYYY-MM-DD-HH-MM-SS.zip.json`

## How It Works

On first startup, the server downloads the S&box API JSON (~9 MB) and caches it locally in `~/.sbox-api-mcp/`. Subsequent startups use an ETag check to skip re-downloading if the data hasn't changed.

The API data is indexed in-memory using [Fuse.js](https://www.fusejs.io/) for fast fuzzy search across type names, member names, and documentation summaries.

## License

MIT
