# Cloudflare Toolkit

Comprehensive Cloudflare MCP integration for Claude Code with 5 MCP servers and guidance skills.

## Features

- **Documentation Search** - Semantic search across Cloudflare docs
- **Workers Bindings** - Manage KV, R2, D1, and Hyperdrive resources
- **Workers Builds** - Monitor deployments and read build logs
- **Observability** - Query logs, metrics, and debug Workers
- **DNS Analytics** - Monitor DNS performance and configuration

## Installation

```bash
claude plugin install florin-popa-marketplace/cloudflare-toolkit
```

Or add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["florin-popa-marketplace/cloudflare-toolkit"]
}
```

## Prerequisites

- Node.js (for npx)
- Cloudflare account with OAuth access

## MCP Servers

| Server | URL |
|--------|-----|
| Documentation | https://docs.mcp.cloudflare.com/mcp |
| Workers Bindings | https://bindings.mcp.cloudflare.com/mcp |
| Workers Builds | https://builds.mcp.cloudflare.com/mcp |
| Observability | https://observability.mcp.cloudflare.com/mcp |
| DNS Analytics | https://dns-analytics.mcp.cloudflare.com/mcp |

## Authentication

On first use of each MCP server, a browser window opens for Cloudflare OAuth authentication. This is a one-time setup per session.

## Skills

### cloudflare-docs
Guidance for searching Cloudflare documentation effectively.

### workers-bindings
Reference for managing Workers storage bindings (KV, R2, D1, Hyperdrive).

### workers-builds
Workflows for monitoring deployments and debugging build failures.

### workers-observability
Patterns for querying logs, analyzing metrics, and debugging Workers.

### dns-analytics
Guidance for DNS performance monitoring and troubleshooting.

## Common Workflows

### Post-Deployment Verification
1. Check build status with Workers Builds tools
2. Review build logs for errors/warnings
3. Query observability for runtime errors
4. Check DNS analytics if using custom domains

### Debugging Worker Issues
1. Query observability for recent errors
2. Use observability_keys to discover available fields
3. Filter by specific criteria to narrow down
4. Review build logs if issue correlates with deployment

## Sources

- [Cloudflare MCP Servers](https://developers.cloudflare.com/agents/model-context-protocol/mcp-servers-for-cloudflare/)
- [GitHub: cloudflare/mcp-server-cloudflare](https://github.com/cloudflare/mcp-server-cloudflare)
