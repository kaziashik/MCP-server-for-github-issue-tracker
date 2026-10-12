# 🚀 Build & Monetize  MCP Server (Model Context Protocol) | MCPize 



## 📚 What You’ll Learn

This small projet covers:

- ✅ What the Model Context Protocol (MCP) is and how it works.
- ✅ How to build a GitHub Issue Tracker using TypeScript and Octokit.
- ✅ Connecting your local MCP server to AI clients like Cursor.
- ✅ Deploying your server for free using the MCPize platform.
- ✅ Setting up AI-generated pricing plans to monetize your developer tools.

---

## 🧑‍💻 Tech Stack

- [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) (`@modelcontextprotocol/sdk`) with Streamable HTTP
- TypeScript (ES2022, NodeNext, strict) and Node.js 20+
- [Octokit](https://github.com/octokit/rest.js) for the GitHub API
- Express 5 for `/health` and `POST /mcp`
- Zod for tool input validation
- MCPize for deployment (`mcpize.yaml`, Dockerfile)

---

## What this project is

`github-issue-tracker` is a small MCP server. Almost all of the code lives in `src/index.ts`. It authenticates to GitHub with `GITHUB_TOKEN` and exposes five tools an AI client can call.

| Path | Role |
|---|---|
| `src/index.ts` | Server, five tools, Express routes |
| `package.json` | Scripts and dependencies |
| `tsconfig.json` | TypeScript config |
| `mcpize.yaml` | MCPize deploy manifest (TypeScript, HTTP) |
| `Dockerfile` | Multi-stage Node 22 image, port 8080 |
| `.env` | Local `GITHUB_TOKEN` (never commit this file) |
| `.cursor/mcp.json` | Cursor connection to this server |

The server listens on `PORT` (default `8080`).

- `GET /health` returns `{ "status": "healthy" }`
- `POST /mcp` handles MCP requests

## Tools

1. **`list_issues`** — Fetches up to 10 issues for an owner and repo. `state` can be `open`, `closed`, or `all` (default `open`). Pull requests are filtered out.
2. **`triage_issue`** — Labels an issue `bug` when the title or body contains "bug", otherwise `enhancement`.
3. **`weekly_digest`** — Counts issues updated in the last 7 days.
4. **`release_notes`** — Builds markdown from the last 10 closed pull requests that were merged.
5. **`add_comment`** — Posts a comment on an issue or pull request.

## Setup

You need Node.js 20 or newer.

### 1. Install dependencies

From the project folder:

```bash
npm install
```

That installs:

| Package | Used for |
|---|---|
| `@modelcontextprotocol/sdk` | MCP server and Streamable HTTP transport |
| `@octokit/rest` | GitHub API (issues, labels, comments, pull requests) |
| `express` | HTTP server for `/health` and `/mcp` |
| `zod` | Tool input validation |
| `dotenv` | Loads `GITHUB_TOKEN` from `.env` |
| `chalk` | Colored startup logs |
| `typescript`, `tsx`, `@types/node`, `@types/express` | Build and dev reload |

### 2. Create a GitHub token

1. Open [fine-grained personal access tokens](https://github.com/settings/tokens?type=beta) while signed in to GitHub.
2. Click **Generate new token**.
3. Name it `MCP-server-for-github-issue-tracker`.
4. Set an expiration (30 or 90 days is enough for local use).
5. Under **Repository access**, choose **Only select repositories** and pick the repos this server should use.
6. Under **Permissions**, set:
   - **Issues:** Read and write (list, label, comment)
   - **Pull requests:** Read-only (release notes)
   - **Metadata:** Read-only (GitHub adds this automatically)
7. Generate the token and copy it once. GitHub will not show it again.

### 3. Put the token in `.env`

Create `.env` in the project root:

```bash
GITHUB_TOKEN=github_pat_your_token_here
```

`PORT` is optional. It defaults to `8080`.

`.env` is listed in `.gitignore`. Do not commit the token, and do not paste it into chat.

### 4. Start the server

```bash
npm run dev
```

Leave this terminal open. A working start looks like:

```text
MCP Server running on http://localhost:8080
  Health: http://localhost:8080/health
  MCP:    http://localhost:8080/mcp
```

Other scripts:

```bash
npm run build  # compile TypeScript to dist/
npm start      # run the compiled server
```

## Connect the server in Cursor

This server speaks HTTP, so Cursor does not start it. `npm run dev` must already be running.

1. In the project root, create `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "github-issue-tracker": {
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

2. Open **Cursor Settings → MCP**.
3. Find **github-issue-tracker** and turn the project source on.
4. Click **Reload**.

It is connected when the Streamable HTTP error is gone and these tools are enabled: `list_issues`, `triage_issue`, `weekly_digest`, `release_notes`, `add_comment`.

`GET /mcp` returns **405 Method Not Allowed** on purpose. Cursor opens that request looking for a server-sent event stream. This server only accepts `POST`.

## Test the server from Cursor

1. Confirm the process is up: open `http://localhost:8080/health`. It should return `{"status":"healthy"}`.
2. In Cursor MCP settings, click **Reload** on `github-issue-tracker`.
3. Open a new Agent chat and ask:

```text
Use the github-issue-tracker list_issues tool for owner kaziashik and repo MCP-server-for-github-issue-tracker
```

A connected server returns an issue list, or `No issues found.` A missing or invalid `GITHUB_TOKEN` returns a GitHub authentication error instead. Restart `npm run dev` after you change `.env`, then reload the server in Cursor.

---
