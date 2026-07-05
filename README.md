# CardGuard MCP Gateway

Official gateway for CardGuard — a transaction risk evaluation engine combined with off-chain IP threat filtering and on-chain verification via Base chain USDC.

## 🚀 Quick Start (Cursor / Claude / OpenWebUI)
Add the following standard MCP server configuration to your settings (`~/Library/Application Support/Cursor/User/globalStorage/mcp-settings.json`, `claude_desktop_config.json`, or equivalent):

```json
{
  "mcpServers": {
    "cardguard": {
      "command": "curl",
      "args": ["-s", "https://api.coaxon.tech/mcp"]
    }
  }
}
```

## 🛡️ Monetization & Access Paths
Choose the monetization tier that fits your agent architecture:

- **Web3 Pay-Per-Call (x402 V2)**: `$0.005` per call via Base chain / USDC. Zero-dependency on-chain verification via Base chain USDC.
- **Web2 Enterprise Subscription**: `$199 / month` for prepaid monthly access. — [Subscribe via Polar.sh](https://buy.polar.sh/polar_cl_wiFygLaUrvozuQxR5AGYZB6uMpOpUWqkuNGM94Txdp7)

## 🌐 Official Website & Documentation
- **API Endpoint**: `https://api.coaxon.tech/mcp`
- **Agent Card (A2A v1.0)**: `https://api.coaxon.tech/.well-known/agent-card.json`
- **Organization**: https://coaxon.tech

#mcp-server #fraud-prevention #ai-agents #security-api #cardguard
