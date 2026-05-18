# 🔴 Redlib MCP Server

[![Docker Pulls](https://img.shields.io/docker/pulls/alfafadock/mcp-redlib?label=docker%20pulls)](https://hub.docker.com/r/alfafadock/mcp-redlib)
[![Docker Image Size](https://img.shields.io/docker/image-size/alfafadock/mcp-redlib/latest)](https://hub.docker.com/r/alfafadock/mcp-redlib)

A **Model Context Protocol (MCP) server** that enables AI agents to interact with Reddit through your private **Redlib** instance. No Reddit API keys required — just a running Redlib instance!

Serves tools over **HTTP (Streamable HTTP transport)** — deploy it anywhere and connect any MCP client.

---

## 📑 Table of Contents

- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Configuration](#-configuration)
- [Available Tools](#️-available-tools)
- [Integration with AI Clients](#-integration-with-ai-clients)
  - [Claude Desktop](#claude-desktop)
  - [Hermes Agent (Native MCP)](#hermes-agent-native-mcp)
  - [Cursor](#cursor)
  - [VS Code / GitHub Copilot](#vs-code--github-copilot)
  - [Any HTTP MCP Client](#any-http-mcp-client)
- [Docker Compose](#-docker-compose)
- [Security: Default vs Hardened](#-security-default-vs-hardened)
- [Development](#-development)
- [Example Usage with AI](#-example-usage-with-ai)
- [Contributing](#-contributing)
- [License](#-license)

---

## ✨ Features

- 🔒 **Privacy-First** — Uses your self-hosted Redlib, no tracking or API keys
- 🛠️ **3 Powerful Tools** — Search posts, get hot posts, fetch full post details with comments
- 🌐 **HTTP MCP Endpoint** — Streamable HTTP transport, no stdio dependency
- 🐳 **Docker Ready** — Exposes port `3000`, deploy anywhere
- 🔌 **Works with Any HTTP MCP Client** — Claude Desktop, Hermes, Cursor, VS Code, and more
- 📊 **Structured Output** — Returns clean JSON instead of raw HTML

---

## 📋 Prerequisites

Before using this MCP server, you need:

1. **Redlib Instance** — A running Redlib instance (default: `http://localhost:8080`)
   - [Redlib GitHub](https://github.com/redlib-org/redlib)
   - [Redlib Docker Setup](https://github.com/redlib-org/redlib#docker)

2. **MCP Client** — Any MCP client that supports HTTP transport:
   - [Claude Desktop](https://claude.ai/download)
   - [Hermes Agent](https://hermes-agent.nousresearch.com)
   - [Cursor](https://cursor.sh/)
   - [VS Code](https://code.visualstudio.com/) with GitHub Copilot
   - Any MCP-compatible client

---

## 🚀 Quick Start

### Option 1: Docker (Recommended)

```bash
# Pull and run the default version
docker run -d --rm \
  -p 3000:3000 \
  -e REDLIB_URL=http://localhost:8080 \
  --name redlib-mcp \
  alfafadock/mcp-redlib:latest
```

The MCP endpoint will be available at `http://localhost:3000/mcp`.

### Option 2: Hardened Docker (Security-Focused)

```bash
# Uses non-root user and minimal privileges
docker run -d --rm \
  -p 3000:3000 \
  --cap-drop=ALL \
  --security-opt no-new-privileges:true \
  -e REDLIB_URL=http://localhost:8080 \
  --name redlib-mcp-hardened \
  alfafadock/mcp-redlib:hardened
```

### Option 3: Local Development

```bash
# Clone and setup
git clone https://github.com/tdussmann/redlib-mcp-server.git
cd redlib-mcp-server

# Install dependencies
npm install

# Build
npm run build

# Run (starts HTTP server on port 3000)
npm start
```

---

## 🐳 Docker Compose

<details>
<summary>Click to expand Docker Compose setup</summary>

Create `docker-compose.yml`:

```yaml
services:
  redlib-mcp:
    image: alfafadock/mcp-redlib:latest
    container_name: redlib-mcp
    ports:
      - "3000:3000"
    environment:
      - REDLIB_URL=http://redlib:8080   # Adjust if Redlib is on a different host
      - PORT=3000
    restart: unless-stopped
```

If running alongside a Redlib container in the same compose file:

```yaml
services:
  redlib:
    image: quay.io/redlib/redlib:latest
    container_name: redlib
    ports:
      - "8080:8080"
    restart: unless-stopped

  redlib-mcp:
    image: alfafadock/mcp-redlib:latest
    container_name: redlib-mcp
    ports:
      - "3000:3000"
    environment:
      - REDLIB_URL=http://redlib:8080
    depends_on:
      - redlib
    restart: unless-stopped
```
</details>

---

## 🔧 Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `REDLIB_URL` | `http://localhost:8080` | URL of your Redlib instance |
| `PORT` | `3000` | Port for the HTTP MCP server |
| `MCP_ENDPOINT` | `/mcp` | HTTP path for the MCP endpoint |

### Example: Custom Redlib Port (e.g., 8085)

```bash
docker run -d --rm \
  -p 3000:3000 \
  -e REDLIB_URL=http://localhost:8085 \
  -e PORT=3000 \
  alfafadock/mcp-redlib:latest
```

### Example: Custom Port and Endpoint

```bash
docker run -d --rm \
  -p 8081:8081 \
  -e PORT=8081 \
  -e MCP_ENDPOINT=/redlib \
  -e REDLIB_URL=http://localhost:8080 \
  alfafadock/mcp-redlib:latest
```

Server available at `http://localhost:8081/redlib`.

---

## 🛠️ Available Tools

### 1. `search_reddit`
Search Reddit posts using your private Redlib instance.

**Parameters:**
- `query` (required) — Search query string
- `subreddit` (optional) — Limit search to specific subreddit

**Example:**
```json
{
  "query": "rust programming",
  "subreddit": "rust"
}
```

**Returns:** JSON with post IDs, titles, authors, scores, and comment counts.

---

### 2. `get_subreddit_hot`
Get hot posts from a specific subreddit.

**Parameters:**
- `subreddit` (required) — Subreddit name (without r/)
- `limit` (optional) — Number of posts (default: 25)

**Example:**
```json
{
  "subreddit": "rust",
  "limit": 10
}
```

---

### 3. `get_post`
Get a specific post with its comments.

**Parameters:**
- `subreddit` (required) — Subreddit name
- `postId` (required) — Reddit post ID (from search results)

**Example:**
```json
{
  "subreddit": "rust",
  "postId": "abc123"
}
```

**Returns:** Full post body, score, and up to 10 top comments.

---

## 🔌 Integration with AI Clients

All integrations use the **HTTP MCP endpoint** at `http://localhost:3000/mcp` (adjust host/port for your deployment).

### Claude Desktop

Edit `~/.config/claude/claude_desktop_config.json` (Linux/macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "redlib": {
      "url": "http://localhost:3000/mcp"
    }
  }
}
```

**For remote deployment:**
```json
{
  "mcpServers": {
    "redlib": {
      "url": "http://your-server-ip:3000/mcp"
    }
  }
}
```

**After updating:** Restart Claude Desktop. You should see a hammer icon (🔨) indicating MCP tools are available.

*Reference: [Claude Desktop MCP Docs](https://modelcontextprotocol.io/docs/develop/connect-local-servers)*

---

### Hermes Agent (Native MCP)

Add to your `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  redlib:
    url: "http://localhost:3000/mcp"
```

If your Redlib container is networked alongside Hermes Agent, use the container name/host:

```yaml
mcp_servers:
  redlib:
    url: "http://redlib-mcp:3000/mcp"
```

For remote deployments with authentication:

```yaml
mcp_servers:
  redlib:
    url: "http://your-server:3000/mcp"
    headers:
      Authorization: "Bearer your-token-here"
```

**After updating:** Restart Hermes Agent. Tools will appear with the `mcp_redlib_` prefix.

---

### Cursor

Add to `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project level):

```json
{
  "mcpServers": {
    "redlib": {
      "url": "http://localhost:3000/mcp"
    }
  }
}
```

---

### VS Code / GitHub Copilot

Create `.vscode/mcp.json` in your project:

```json
{
  "servers": {
    "redlib": {
      "url": "http://localhost:3000/mcp"
    }
  }
}
```

---

### Any HTTP MCP Client

The server speaks the standard **MCP Streamable HTTP transport** (protocol version 2025-03-26). Configure any compliant MCP client with:

```
URL: http://<host>:3000/mcp
```

---

## 🔒 Security: Default vs Hardened

| Feature | Default (`latest`) | Hardened (`hardened`) |
|---------|-------------------|---------------------|
| **User** | root | Non-root (mcpuser) |
| **File Ownership** | root | mcpuser |
| **Build Stages** | Single | Multi-stage (smaller) |
| **Runtime Caps** | Default | Requires `--cap-drop=ALL` |
| **Exposed Port** | 3000 | 3000 |
| **Use Case** | Development, testing | Production |

### Using Hardened Image

```bash
docker run -d --rm \
  -p 3000:3000 \
  --cap-drop=ALL \
  --security-opt no-new-privileges:true \
  -e REDLIB_URL=http://localhost:8080 \
  alfafadock/mcp-redlib:hardened
```

---

## 💻 Development

### Project Structure

```
redlib-mcp-server/
├── src/
│   └── index.ts              # HTTP MCP server (Streamable HTTP transport)
├── dist/                      # Compiled JavaScript (gitignored)
├── Dockerfile                 # Default Docker image
├── Dockerfile.hardened        # Hardened Docker image
├── docker-compose.yml         # Production compose
├── package.json
├── tsconfig.json
└── README.md
```

### Build from Source

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run locally (HTTP server on port 3000)
npm start

# Development mode with hot-reload
npm run dev
```

### Build Custom Docker Images

```bash
# Default version
docker build -t redlib-mcp-server .

# Hardened version
docker build -f Dockerfile.hardened -t redlib-mcp-server:hardened .
```

### Run Custom Build

```bash
docker run -d --rm \
  -p 3000:3000 \
  -e REDLIB_URL=http://localhost:8080 \
  redlib-mcp-server
```

### Test the Endpoint

```bash
# Health check — should return 400 (no session) or MCP initialization response
curl -v http://localhost:3000/mcp

# Send MCP initialize request
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0.0"}}}'
```

---

## 📝 Example Usage with AI

Once connected to your AI client (e.g., Claude), you can:

```
User: "Search Reddit for 'home lab setup' and summarize the top results"

AI uses search_reddit tool →
Returns structured JSON with posts →
AI summarizes the findings for you
```

```
User: "Get the full post and comments for that Rust tutorial I searched earlier"

AI uses get_post with postId →
Returns post body + comments →
AI provides detailed analysis
```

---

## Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

---

## License

MIT — see the [LICENSE](./LICENSE) file for details.
