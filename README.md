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

## Connect an MCP client

This server speaks HTTP, so the client does not start it. `npm run dev` must already be running.

To connect Cursor, add this file at `.cursor/mcp.json` in the project root. The `url` is the MCP endpoint the client calls:

```json
{
  "mcpServers": {
    "github-issue-tracker": {
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

That block is the whole client connection. `github-issue-tracker` is the name Cursor shows, and `http://localhost:8080/mcp` is this server.

Then, in Cursor:

1. Open **Cursor Settings → MCP**.
2. Find **github-issue-tracker** and turn the project source on.
3. Click **Reload**.

It is connected when the Streamable HTTP error is gone and these tools are enabled: `list_issues`, `triage_issue`, `weekly_digest`, `release_notes`, `add_comment`.

`GET /mcp` returns **405 Method Not Allowed** on purpose. Cursor opens that request looking for a server-sent event stream. This server only accepts `POST`.

## Test the server from Cursor

1. Confirm the server is up. Local: `http://localhost:8080/health`. Hosted: `https://mcp-server-for-github-issue-tracker.mcpize.run/health`. Both should return `{"status":"healthy"}`.
2. In Cursor MCP settings, click **Reload** on `github-issue-tracker`.
3. Open a new Agent chat and paste one prompt at a time.

A connected server returns tool output. A missing or invalid `GITHUB_TOKEN` returns a GitHub authentication error. After you change `.env`, restart `npm run dev` and reload the server in Cursor. On MCPize, set the secret and run `mcpize deploy` again.

These three only read GitHub:

```text
Use the github-issue-tracker list_issues tool for owner kaziashik and repo MCP-server-for-github-issue-tracker. Include open, closed, and all issues.
```

```text
Use the github-issue-tracker weekly_digest tool for owner kaziashik and repo MCP-server-for-github-issue-tracker.
```

```text
Use the github-issue-tracker release_notes tool for owner kaziashik and repo MCP-server-for-github-issue-tracker.
```

`list_issues` returns issue titles, or `No issues found.` `weekly_digest` returns a count for the last 7 days. `release_notes` returns merged pull requests, or `No recent merged PRs found.`

These two change the repository. Create a test issue on GitHub first, then replace `1` with that issue number.

```text
Use the github-issue-tracker triage_issue tool for owner kaziashik, repo MCP-server-for-github-issue-tracker, and issue number 1.
```

```text
Use the github-issue-tracker add_comment tool for owner kaziashik, repo MCP-server-for-github-issue-tracker, and issue number 1. Comment body: MCP server test comment.
```

`triage_issue` adds the label `bug` when the title or body contains "bug", otherwise `enhancement`. `add_comment` posts that text on the issue.

## Deploy on MCPize

`mcpize.yaml` already tells MCPize how to build and start this server.

```bash
npm install -g mcpize
mcpize login
mcpize secrets set GITHUB_TOKEN
mcpize deploy
```

`mcpize login` opens the browser. `mcpize secrets set GITHUB_TOKEN` asks for the token in the terminal. Do not put that token in the repo.

After a successful deploy:

- Gateway: `https://mcp-server-for-github-issue-tracker.mcpize.run`
- Health: `https://mcp-server-for-github-issue-tracker.mcpize.run/health`
- MCP: `https://mcp-server-for-github-issue-tracker.mcpize.run/mcp`
- Marketplace: `https://mcpize.com/mcp/mcp-server-for-github-issue-tracker`

## Connect Cursor to the live link

Yes. After deploy you can use the hosted server instead of `localhost`. Cursor needs the MCP path, not the site root:

`https://mcp-server-for-github-issue-tracker.mcpize.run/mcp`

Put that URL in `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "github-issue-tracker": {
      "url": "https://mcp-server-for-github-issue-tracker.mcpize.run/mcp"
    }
  }
}
```

Then:

1. Open **Cursor Settings → MCP**.
2. Reload **github-issue-tracker**.
3. Stop the local `npm run dev` process. Cursor is now calling the hosted server.
4. Test with the same `list_issues` prompt.

If a tool returns a GitHub authentication error, the cloud server does not have the token yet. Set it, then deploy again:

```bash
mcpize secrets set GITHUB_TOKEN
mcpize deploy
```

Check the deployment anytime with `mcpize status` and `mcpize logs`.

---
