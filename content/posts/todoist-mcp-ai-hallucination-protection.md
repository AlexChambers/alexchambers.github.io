+++
date = '2025-12-15T10:57:21+01:00'
draft = false
title = 'Adding Hallucination Protection to Todoist MCP'
+++

The official Todoist MCP server doesn't show task names when it operates on them. It just reports back opaque IDs. Within 10 seconds of using it, I knew this was a problem.

## The Problem

I run my life on Todoist using a GTD (Getting Things Done) setup. Each Area of Focus (Career, Health, Home Management, etc.) is a top-level project, with sections for active tasks, Waiting For, and Someday-Maybe. It works well, but it means I have a lot of tasks across a lot of projects.

I've been using Claude Code with MCP (Model Context Protocol) to manage those tasks. It's great for bulk operations, like "close all the grocery items" or "move these tasks to the Waiting For section."

The issue is that the original server operates on opaque IDs. When Claude closes a task, it reports something like `closed task 8234567890`. Did it get the right one? No idea. I found myself manually cross-referencing IDs to verify what actually happened. That's not a workflow, that's babysitting.

And if Claude hallucinates or misremembers which ID maps to which task, you've just closed (or deleted) the wrong thing with no way to know until you check.

## The Fix: Verification Parameters

My fork adds a simple but effective safeguard. For any destructive operation (update, close, delete, move), the server requires human-readable verification:

```typescript
// Before: just an ID
close-task({ taskId: "8234567890" })

// After: ID plus verification
close-task({
  taskId: "8234567890",
  taskName: "Buy milk",
  projectName: "Home Management"
})
```

The server validates that the task name and project name actually match the ID before executing. If Claude has the wrong ID, the operation fails with a clear error instead of silently destroying the wrong data.

It's not bulletproof. Claude could hallucinate consistent but incorrect details. But in practice, it catches the common failure mode: stale or confused IDs from earlier in a conversation.

## Token Efficiency

Once the verification stuff was working, I started noticing how often Claude Code was compacting the conversation. The original server exposes a lot of tools (labels, productivity stats, full project CRUD) that I never use. Each tool definition eats context window space, and I was burning tokens on tools I'd never call.

My fork disables the tools I don't need and filters response payloads to return only essential fields. Less bloat, fewer compactions, lower token costs.

## Using It

If you want to try it:

```bash
git clone https://github.com/AlexChambers/todoist-mcp
cd todoist-mcp
npm install && npm run build
claude mcp add todoist-mcp -e TODOIST_API_KEY=your_key -- node $(pwd)/build/index.js
```

You'll need a Todoist API key from Settings > Integrations > Developer.

## Should You Use This or the Official One?

Doist has since released [todoist-ai](https://github.com/Doist/todoist-ai), which is more comprehensive. If you want full coverage of the Todoist API and trust your LLM to handle IDs correctly, use that.

If you prefer explicit verification on destructive operations, my fork is there. It's opinionated and minimal, which is how I like my tools.

The code is at [github.com/AlexChambers/todoist-mcp](https://github.com/AlexChambers/todoist-mcp).
