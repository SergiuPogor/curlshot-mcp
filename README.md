# CurlShot MCP server

Let your AI agent take screenshots of web pages. [CurlShot](https://curlshot.com) runs a hosted [Model Context Protocol](https://modelcontextprotocol.io) server, so there is nothing to install: point your MCP client at the endpoint and add your API key.

```
https://curlshot.com/api/mcp
```

- **Transport:** streamable HTTP (JSON-RPC 2.0 over `POST`). The server is stateless, so a `GET` answers `405`.
- **Auth:** `Authorization: Bearer YOUR_API_KEY`
- **Protocol versions:** `2025-06-18`, `2025-03-26`, `2024-11-05`

This repository holds the documentation, the [`server.json`](server.json) manifest (MCP Registry format) and the license. The server itself runs at curlshot.com.

## Tools

| Tool | What it does | Needs a key |
| --- | --- | --- |
| `take_screenshot` | Renders a public web page and returns it as an image, a PDF, or a video of the page scrolling. Each successful call uses one screenshot of your quota. Cache hits and failed renders are free. | Yes |
| `list_devices` | Lists the device presets (phones, tablets, desktops) that `take_screenshot` accepts in `viewport_device`. | No |

`take_screenshot` arguments:

| Argument | Meaning |
| --- | --- |
| `url` (required) | Full address of the page, starting with `http://` or `https://`. |
| `format` | `png`, `jpeg` (default), `webp`, `pdf`, `mp4`, `webm` or `gif`. The video formats record the page scrolling from top to bottom, on plans that include video. |
| `full_page` | Capture the whole page, not only the first screen. |
| `viewport_device` | A device preset such as `iphone_15_pro` (see `list_devices`). |
| `viewport_width`, `viewport_height` | Browser window size in pixels (default 1280 x 1024). |
| `dark_mode` | Ask the page for its dark theme. |
| `block_ads`, `block_cookie_banners` | On by default. |
| `selector` | Capture only the first element matching this CSS selector. |
| `wait_until`, `delay` | When the page counts as loaded, plus extra seconds to wait. |
| `cache` | Reuse a stored copy of the same request. |

Small results come back inline as an image the model can see. Results over 6 MB come back as a download link valid for about a day.

## Get an API key

1. Sign up at [curlshot.com/register](https://curlshot.com/register). The free plan includes 100 screenshots a month, no card needed.
2. Create a separate API key for your agent in the dashboard, so you can revoke it on its own.
3. Leave **Require signature** off for that key. MCP calls are not signed.

Keep the key in your client configuration, not in a prompt. The examples below use `YOUR_API_KEY` as a placeholder.

## Client setup

### Claude Code

```bash
claude mcp add --transport http curlshot https://curlshot.com/api/mcp \
  --header "Authorization: Bearer YOUR_API_KEY"
```

### Claude Desktop

Claude Desktop starts local (stdio) servers from its config file, so connect through the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) proxy. Edit `claude_desktop_config.json` (Settings → Developer → Edit Config):

```json
{
  "mcpServers": {
    "curlshot": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://curlshot.com/api/mcp",
        "--transport",
        "http-only",
        "--header",
        "Authorization: Bearer YOUR_API_KEY"
      ]
    }
  }
}
```

Restart Claude Desktop after saving. This needs Node.js.

### Cursor

Add to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "curlshot": {
      "url": "https://curlshot.com/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

### VS Code (GitHub Copilot)

Add to `.vscode/mcp.json`. VS Code asks for the key once and stores it securely:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "curlshot-api-key",
      "description": "CurlShot API key",
      "password": true
    }
  ],
  "servers": {
    "curlshot": {
      "type": "http",
      "url": "https://curlshot.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${input:curlshot-api-key}"
      }
    }
  }
}
```

### Gemini CLI

This repository is also a [Gemini CLI extension](https://geminicli.com/extensions). Install it and enter your API key when asked:

```bash
gemini extensions install https://github.com/SergiuPogor/curlshot-mcp
```

The key is stored as the `CURLSHOT_API_KEY` setting and sent as `Authorization: Bearer ...`.

### Agent skill (no MCP)

For agents that use [Agent Skills](https://agentskills.io) instead of MCP, [`skills/curlshot/SKILL.md`](skills/curlshot/SKILL.md) teaches the agent to call the HTTP API directly with `curl`, reading the key from `CURLSHOT_API_KEY`:

```bash
npx skills add SergiuPogor/curlshot-mcp
```

### Other clients

Any client that supports remote servers over streamable HTTP works. Most take a config like this:

```json
{
  "mcpServers": {
    "curlshot": {
      "type": "http",
      "url": "https://curlshot.com/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

Clients that only start local servers can use the `mcp-remote` command shown for Claude Desktop.

## Try it with curl

```bash
curl -X POST "https://curlshot.com/api/mcp" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "take_screenshot",
      "arguments": { "url": "https://example.com", "format": "webp" }
    }
  }'
```

## Limits and pricing

- Free plan: 100 screenshots a month, no card. Paid plans start at $19 a month. See [pricing](https://curlshot.com/pricing).
- Only successful renders count. Cache hits and failed renders are free.
- The MCP endpoint accepts 120 requests per minute per IP address, on top of your plan's own limits.
- When the monthly limit is reached, the tool returns an error with an upgrade link instead of an image.

## Links

- Website: [curlshot.com](https://curlshot.com)
- Docs: [curlshot.com/docs](https://curlshot.com/docs)
- MCP guide for AI agents: [curlshot.com/docs/guides/ai-agents](https://curlshot.com/docs/guides/ai-agents)
- Pricing: [curlshot.com/pricing](https://curlshot.com/pricing)
- Server card: [curlshot.com/.well-known/mcp.json](https://curlshot.com/.well-known/mcp.json)

## License

[MIT](LICENSE)
