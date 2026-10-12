# 🚀 Build & Monetize Your First MCP Server (Model Context Protocol) | MCPize 



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
| `.env` | Local environment variables (currently empty) |

The server listens on `PORT` (default `8080`).

- `GET /health` returns `{ "status": "healthy" }`
- `POST /mcp` handles MCP requests

## Tools

1. **`list_issues`** — Fetches up to 10 issues for an owner and repo. `state` can be `open`, `closed`, or `all` (default `open`). Pull requests are filtered out.
2. **`triage_issue`** — Labels an issue `bug` when the title or body contains "bug", otherwise `enhancement`.
3. **`weekly_digest`** — Counts issues updated in the last 7 days.
4. **`release_notes`** — Builds markdown from the last 10 closed pull requests that were merged.
5. **`add_comment`** — Posts a comment on an issue or pull request.

## Scripts

```bash
npm install    # install dependencies
npm run dev    # hot reload with tsx (src/index.ts)
npm run build  # compile TypeScript to dist/
npm start      # run dist/index.js
```

Set `GITHUB_TOKEN` in `.env` before calling the GitHub tools. Optionally set `PORT` (defaults to `8080`).

Connect an MCP client such as Cursor to `http://localhost:8080/mcp`.

---
