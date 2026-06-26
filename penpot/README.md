# Penpot + OpenCode MCP Integration

Design-driven development: use OpenCode agents to create and manipulate Penpot designs via MCP.

## Architecture

```
┌─────────────┐     MCP Stream      ┌──────────────────┐
│  OpenCode   │ ◄──────────────────► │  Penpot MCP      │
│  (AI Agent) │    localhost:9001    │  Plugin          │
└─────────────┘                     └──────┬───────────┘
                                           │
                                    ┌──────▼───────────┐
                                    │  Penpot Frontend │
                                    │  localhost:9001   │
                                    └──────────────────┘
```

## Docker Compose

The Penpot stack runs 6 services:

| Service | Image | Port | Purpose |
|---------|-------|------|---------|
| `penpot-frontend` | `penpotapp/frontend:2.16` | `9001` | Web UI + MCP stream endpoint |
| `penpot-backend` | `penpotapp/backend:2.16` | — | API, auth, persistence |
| `penpot-mcp` | `penpotapp/mcp:2.16` | `4401` | MCP plugin relay |
| `penpot-exporter` | `penpotapp/exporter:2.16` | — | SVG/PNG export rendering |
| `penpot-postgres` | `postgres:15` | — | Database |
| `penpot-valkey` | `valkey/valkey:8.1` | — | WebSocket pub/sub |

### Key Flags

```yaml
PENPOT_FLAGS: disable-email-verification enable-smtp enable-prepl-server disable-secure-session-cookies enable-mcp
```

`enable-mcp` is required for the MCP plugin to function.

### Quick Start

```bash
cd C:\sources\personal-source\docker\penpot
docker compose up -d
```

Access Penpot at [http://localhost:9001](http://localhost:9001).

---

## OpenCode MCP Configuration

**Config path:** `~/.config/opencode/opencode.json`

```json
{
  "mcp": {
    "penpot": {
      "type": "remote",
      "url": "http://localhost:9001/mcp/stream?userToken=<YOUR_TOKEN>",
      "enabled": true
    }
  }
}
```

### Obtaining the User Token

1. Open Penpot at `http://localhost:9001`
2. Open a design file
3. Launch the **MCP Plugin** from the Plugins panel
4. Click **Connect** — the plugin displays a connection URL
5. Extract the `userToken` query parameter from that URL

The token is a JWE-encrypted session credential. It changes per session — update the config when reconnecting.

### Verifying Connectivity

After restarting OpenCode, run a quick test:

```
check penpot mcp
```

The agent should respond: `connected: true, penpotVersion: "penpot available"`.

---

## Agent Capabilities via Penpot MCP

| Capability | Tool |
|------------|------|
| Create boards, shapes, text | `penpot_execute_code` (JS via Plugin API) |
| Export designs to PNG/SVG | `penpot_export_shape` |
| Inspect API types | `penpot_penpot_api_info` |
| Navigate pages, search shapes | `penpotUtils.*` helpers |
| Apply flex/grid layouts | `board.addFlexLayout()` / `board.addGridLayout()` |
| Manage design tokens | `penpot.library.local.tokens` |
| Create/use components | `penpot.library.local.createComponent()` |
| Variant groups | `penpot.createVariantFromComponents()` |

### Design System Used (Ecomerge Demo)

| Property | Value |
|----------|-------|
| Heading font | Playfair Display (700) |
| Body font | Inter (400/700) |
| Primary accent | `#5B4CF0` (purple) |
| Secondary accent | `#FF6B6B` (coral) |
| Dark | `#1A1D26` |
| Light bg | `#F8F8FC` |

---

## Files

```
penpot/
├── docker-compose.yaml    # Penpot stack definition
└── README.md              # This file
```
