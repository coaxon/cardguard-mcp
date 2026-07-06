# CardGuard MCP Gateway

Official gateway for CardGuard — a transaction risk evaluation engine combined with off-chain IP threat filtering and on-chain verification via Base chain USDC.

## 🚀 Quick Start

CardGuard exposes a **Streamable HTTP** MCP endpoint at `https://api.coaxon.tech/mcp`. Configure your client below — do **not** use `curl` as a local `command`; MCP clients expect either a `url` (remote) or a long-lived stdio process (local).

### Cursor

Add to `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "cardguard": {
      "url": "https://api.coaxon.tech/mcp"
    }
  }
}
```

Enterprise subscribers can bypass x402 per-call billing with a Bearer API key:

```json
{
  "mcpServers": {
    "cardguard": {
      "url": "https://api.coaxon.tech/mcp",
      "headers": {
        "Authorization": "Bearer ${env:CARDGUARD_API_KEY}"
      }
    }
  }
}
```

Docs: [cursor.com/docs/mcp](https://cursor.com/docs/mcp)

### Claude Desktop

1. Open Claude Desktop, click the **+** button next to the chat input, and select **Connectors** (or go to **Settings → Connectors**).
2. Choose **Add custom connector**.
3. Enter Server URL: `https://api.coaxon.tech/mcp`
4. Complete the prompts to add the connector.

> **Note:** Requires a **Pro / Max / Team / Enterprise** plan. Custom Connectors are currently in **Beta**.

This path connects directly to the remote Streamable HTTP endpoint — no Node.js, `npx`, or `mcp-remote` required.

Docs: [modelcontextprotocol.io/docs/develop/connect-remote-servers](https://modelcontextprotocol.io/docs/develop/connect-remote-servers)

### OpenWebUI

Requires Open WebUI **v0.6.31+**.

1. Open **Admin Settings → External Tools**.
2. Click **+ (Add Server)**.
3. Set **Type** to **MCP (Streamable HTTP)** — not OpenAPI.
4. Enter **Server URL**: `https://api.coaxon.tech/mcp`
5. Set **Auth** to **None** (free discovery tier) or **Bearer** with your Enterprise API key.
6. Save and restart Open WebUI if prompted.

Docs: [docs.openwebui.com/features/extensibility/mcp](https://docs.openwebui.com/features/extensibility/mcp/)

## 🛡️ Monetization & Access Paths
Choose the monetization tier that fits your agent architecture:

- **Web3 Pay-Per-Call (x402 V2)**: `$0.005` per call via Base chain / USDC. Zero-dependency on-chain verification via Base chain USDC.
- **Web2 Enterprise Subscription**: `$199 / month` for prepaid monthly access. — [Subscribe via Polar.sh](https://buy.polar.sh/polar_cl_wiFygLaUrvozuQxR5AGYZB6uMpOpUWqkuNGM94Txdp7)

## 🌐 Official Website & Documentation
- **API Endpoint**: `https://api.coaxon.tech/mcp`
- **Agent Card (A2A v1.0)**: `https://api.coaxon.tech/.well-known/agent-card.json`
- **Organization**: https://coaxon.tech

#mcp-server #fraud-prevention #ai-agents #security-api #cardguard
