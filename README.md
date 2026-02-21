# Agent Viewer

A kanban board for managing multiple OpenCode agents running in tmux sessions. Spawn, monitor, and interact with agents from a single web UI.

<img width="1466" height="725" alt="Screenshot 2026-02-09 at 14 54 21" src="https://github.com/user-attachments/assets/cd31b988-f649-4e92-9844-7a1ece9aa634" />

Manage your agents from your mobile phone with Tailscale

![IMG_7782](https://github.com/user-attachments/assets/c7298d61-dd37-4d0f-8b0a-d9d1f0231782)

## Prerequisites

- [Node.js](https://nodejs.org/) (v18+)
- [tmux](https://github.com/tmux/tmux)
- [OpenCode CLI](https://docs.anthropic.com/en/docs/claude-code) (`opencode` command available in your PATH)

### Install prerequisites (macOS)

```bash
brew install node tmux
npm install -g opencode
```

## Setup

```bash
git clone <repo-url> && cd agent-viewer
npm install
```

## Usage

```bash
npm start
```

Open http://localhost:4200 in your browser.

### Configuration

| Variable | Default     | Description              |
|----------|-------------|--------------------------|
| `PORT`   | `4200`      | Server port              |
| `HOST`   | `0.0.0.0`   | Bind address (`localhost` for local-only) |

Example:

```bash
HOST=0.0.0.0 PORT=3000 npm start
```

## Architecture

Agent Viewer is a two-file application with no build step or frameworks:

| File | Purpose |
|------|---------|
| `server.js` | Express backend: agent lifecycle, tmux integration, state detection, SSE broadcasting |
| `public/index.html` | Single-file frontend: HTML, CSS, and vanilla JavaScript |

### Backend (server.js)

- **Agent Registry** — In-memory state with `.agent-registry.json` persistence. Tracks label, project path, prompt, state, and timestamps. Auto-recovers on restart.
- **State Detection** — Polls tmux pane output every 3 seconds. Classifies agents as `running`, `idle`, or `completed` via pattern-matching OpenCode's terminal UI.
- **Tmux Integration** — Spawns via `tmux new-session`, captures output via `tmux capture-pane`, sends input via `tmux send-keys`. All commands use timeouts (5–15s).
- **Auto-Discovery** — Scans tmux sessions and process trees to detect existing OpenCode instances and add them to the board.
- **SSE** — `GET /api/events` broadcasts full agent state to all clients at 3s intervals.

### Frontend (public/index.html)

- **Dual view modes** — Kanban (Running/Idle/Completed columns) and Grid (2-column card layout). View preference persisted in `localStorage`.
- **ANSI-to-HTML** — Full terminal output rendering with 16/256/24-bit color support.
- **Responsive** — Mobile tab bar, floating action button, and safe-area insets for notched devices.

## Features

### Views

- **Kanban board** — Three columns (Running, Idle, Completed). Drag cards between columns. Bulk "Clear All" for completed agents.
- **Grid view** — All agents in a responsive grid. Toggle with `V` or the header button. Ideal for scanning many agents at once.

### Agent lifecycle

- **Spawn** — Click `[+ SPAWN]` or press `N`. Enter project path (with recent-projects dropdown and folder browser) and prompt. Each agent runs in its own tmux session.
- **Auto-discovery** — Existing OpenCode tmux sessions are detected and added automatically.
- **Re-spawn** — Completed agents accept a new prompt to start a fresh task in the same project.

### Interaction

- **Send messages** — Type in the prompt field on any card and press `Ctrl+Enter` to send follow-up messages.
- **File uploads** — Drag and drop files onto a card or click `FILE`. Supports images, PDFs, text, and code files.
- **Live output** — Click `VIEW OUTPUT` for a full-screen terminal overlay with ANSI colors. Send messages and keys from the overlay.
- **Interactive prompts** — When OpenCode shows select, permission, plan, or yes/no prompts, on-card buttons let you navigate (↑/↓), confirm (Enter), or cancel (Esc) without opening the terminal.
- **Terminal attach** — Click `TERMINAL` to copy the `tmux attach` command for direct terminal access.

### Keyboard shortcuts

| Key | Action |
|-----|--------|
| `N` | Open spawn modal |
| `V` | Toggle Kanban ↔ Grid view |
| `Escape` | Close modal, output viewer, or confirm dialog |
| `Ctrl+Enter` | Send message from prompt field |

## Remote Access via Tailscale

You can access Agent Viewer from your phone (or any device) by using [Tailscale](https://tailscale.com/).

### 1. Install Tailscale on your Mac

```bash
brew install tailscale
```

Or download from [tailscale.com/download](https://tailscale.com/download).

### 2. Install Tailscale on your phone

Download the Tailscale app from the [App Store](https://apps.apple.com/app/tailscale/id1470499037) or [Google Play](https://play.google.com/store/apps/details?id=com.tailscale.ipn). Sign in with the same account.

### 3. Start the server

```bash
npm start
```

The server binds to `0.0.0.0` by default, so it's already accessible on all network interfaces including Tailscale.

### 4. Open on your phone

Find your Mac's Tailscale IP (shown in the Tailscale app or via `tailscale ip`), then visit:

```
http://<tailscale-ip>:4200
```

If you have [MagicDNS](https://tailscale.com/kb/1081/magicdns) enabled, you can use your machine name instead:

```
http://<machine-name>:4200
```
