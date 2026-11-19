# Sharesight MCP for Unraid

Unraid Community Applications template for [lethek/sharesight-mcp](https://github.com/lethek/sharesight-mcp), an unofficial MCP (Model Context Protocol) server that lets AI assistants such as Claude read your [Sharesight](https://www.sharesight.com) portfolio through the Sharesight v3 API.

This is a community template. It is not affiliated with Sharesight or the sharesight-mcp authors. Questions about the server itself belong on the [project repo](https://github.com/lethek/sharesight-mcp); template questions go to the support link in the template.

## What you need first

- **Sharesight API access.** Email support@sharesight.com to request it. Sharesight may require a particular plan.
- Your API application's **Client ID** and **Client Secret**.

## Quick start

1. Install **Sharesight-MCP** from Community Applications.
2. Enter your **Sharesight Client ID** and **Sharesight Client Secret**.
3. Set **MCP Auth Token** to a long random string. On the Unraid terminal you can run `openssl rand -hex 32`. The container refuses to start without it.
4. Start the container. Check `http://SERVER:3000/healthz` returns OK.
5. Point your MCP client at `http://SERVER:3000/mcp` with the header `Authorization: Bearer YOUR_TOKEN`.

Example for Claude Code:

```bash
claude mcp add --transport http sharesight http://SERVER:3000/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

## Read-only by default

The upstream server defaults to read **and** write. This template defaults **Read-only mode** (`MCP_READ_ONLY`) to `true`, which leaves only the read tools and removes everything that creates, changes or deletes data. Set it to `false` only if you want your AI assistant to be able to edit Sharesight.

## Settings

| Setting | Variable | Default | Notes |
|---|---|---|---|
| MCP port | container `3000` | `3000` | Change the host side if 3000 is taken. |
| Sharesight Client ID | `SHARESIGHT_CLIENT_ID` | | Required. |
| Sharesight Client Secret | `SHARESIGHT_CLIENT_SECRET` | | Required. Masked. |
| MCP Auth Token | `MCP_AUTH_TOKEN` | | Required. Masked. |
| Read-only mode | `MCP_READ_ONLY` | `true` | Upstream default is `false`. |
| Cursor secret | `MCP_CURSOR_SECRET` | random per start | Set it so paging cursors survive restarts. |
| Max response characters | `MCP_MAX_RESPONSE_CHARS` | `30000` | Minimum 5000. |
| Holdings cache (seconds) | `MCP_CACHE_TTL_SECONDS` | `60` | `0` disables. |
| Attachments folder | path `/attachments` (read-only) | | Optional. |
| Attachments root | `MCP_DATA_ROOT` | empty | Set to `/attachments` to enable attachment uploads. |
| Max attachment size | `MCP_MAX_ATTACHMENT_BYTES` | `10485760` | Bytes. |
| Sharesight API base URL | `SHARESIGHT_BASE_URL` | `https://api.sharesight.com` | Leave as is. |

## Security

This container holds credentials that can read (and, if you allow it, change) your financial records.

- Keep it on your LAN or a VPN such as Tailscale.
- If you publish it through a reverse proxy, add strong authentication in front of it.
- Treat the auth token like a password and rotate it if it leaks.
- Sharesight requires TLS 1.3 from 30 September 2026. The image is built on Node 24, which supports it.
