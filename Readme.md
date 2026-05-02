# AMRIT MCP Server

A Model Context Protocol (MCP) server that gives AI coding tools (Claude Code, Cursor) deep contextual knowledge of the AMRIT platform — its architecture, repos, conventions, and documentation.

This is a proof-of-concept built as part of a C4GT DMP 2026 proposal for the AMRIT Agentic AI Coding Framework issue.

## What it does

Once connected to Claude Code or Cursor, any AI session automatically has access to:

- **`search_amrit_docs`** — search indexed AMRIT documentation by natural language query
- **`list_amrit_docs`** — see what's been indexed
- **`get_amrit_context`** — get a platform overview (architecture, repos, stack) at the start of any session
- **`generate_jira_ticket`** — generate a structured JIRA ticket from a plain English description
- **`reindex_docs`** — re-scan docs after adding new files

## Setup

### 1. Install dependencies

```bash
npm install
```

### 2. Build

```bash
npm run build
```

### 3. Add your docs

Create a `docs/` folder in the project root and drop in any `.md` or `.txt` files about AMRIT. Good starting points:

- Copy README files from AMRIT repos
- Export Confluence pages as markdown
- Paste architecture notes

```
amrit-mcp/
  docs/
    architecture-overview.md
    hwc-api-readme.md
    spring-boot-conventions.md
    sdlc-guide.md
```

Or point to a different directory by setting the environment variable:

```bash
set AMRIT_DOCS_DIR=C:\path\to\your\amrit-docs
```

### 4. Connect to Claude Code

Add this to your Claude Code MCP config file.

**Config file location on Windows:**

```
%APPDATA%\Claude\claude_desktop_config.json
```

**Config to add:**

```json
{
  "mcpServers": {
    "amrit": {
      "command": "node",
      "args": ["C:\\path\\to\\amrit-mcp\\dist\\index.js"],
      "env": {
        "AMRIT_DOCS_DIR": "C:\\path\\to\\amrit-mcp\\docs"
      }
    }
  }
}
```

Replace the paths with your actual paths.

### 5. Connect to Cursor

In Cursor settings → MCP, add a new server:

- **Command:** `node`
- **Args:** `["C:\\path\\to\\amrit-mcp\\dist\\index.js"]`
- **Env:** `AMRIT_DOCS_DIR=C:\path\to\amrit-mcp\docs`

## Usage examples

Once connected, in any Claude Code or Cursor session:

```
"How does beneficiary registration work in AMRIT?"
→ Uses search_amrit_docs automatically

"Get me AMRIT context before we start"
→ Calls get_amrit_context, primes the session

"Generate a JIRA ticket for: add pagination to the beneficiary search API in HWC"
→ Calls generate_jira_ticket with component=HWC
```

## How it works

The server uses a simple keyword-based chunk search over indexed markdown files. Each file is split on headings into chunks. Queries are scored by term frequency with a bonus for heading matches.

This is intentionally simple for the prototype — production would use embeddings (e.g. via a local model or OpenAI) for semantic search. The architecture is identical; just swap the `search()` method in `DocIndexer`.

## Project structure

```
src/
  index.ts       — MCP server, tool definitions, document indexer
docs/            — Drop AMRIT documentation here
dist/            — Compiled output (after npm run build)
```

## Roadmap (full proposal scope)

- [ ] Semantic search via embeddings instead of keyword matching
- [ ] GitHub API connector — index AMRIT repos directly without manual copy-paste
- [ ] Confluence connector — pull pages via Confluence REST API
- [ ] JIRA connector — read/write tickets from agent context
- [ ] Code review skill — review PRs against AMRIT coding standards
- [ ] Contribution guide for adding new skills and MCP connectors
