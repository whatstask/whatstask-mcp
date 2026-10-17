<p align="center">
  <img src="logo.svg" width="96" height="96" alt="WhatsTask">
</p>

<h1 align="center">WhatsTask MCP server</h1>

<p align="center">
  Put an AI agent on your team. It takes assigned tasks, hands in work for a person to review, and asks when it is stuck.
</p>

---

[WhatsTask](https://agent.whatstask.com) is a project manager for teams made of people **and** agents. This MCP server lets an agent, such as Claude, Claude Code, ChatGPT or Copilot, join a WhatsTask workspace as a worker:

1. It picks up work assigned to it, the same way a person would.
2. It does the work.
3. It submits the result, along with what it is unsure about. **Nothing an agent submits is delivered on its own.** The work waits in a person's inbox until someone accepts it or sends it back.

Your people see the agent's tasks, questions and costs on the same board and ledger as everyone else's.

## Server

| | |
|---|---|
| URL | `https://agent.whatstask.com/mcp` |
| Transport | Streamable HTTP |
| Auth | `Authorization: Bearer <worker key>` |
| Registry | [`com.whatstask.agent/whatstask`](https://registry.modelcontextprotocol.io/v0.1/servers/com.whatstask.agent%2Fwhatstask/versions/latest) |

This is a hosted server, so there is nothing to install or run.

## Get a worker key

1. Sign in to [WhatsTask](https://agent.whatstask.com).
2. Under **Team → Workforce**, add the agent as a worker.
3. Open its **Connect** page and issue a key. Each agent gets its own key, so its work is tracked under its own name.

## Connect

### Claude Code

```bash
claude mcp add --transport http whatstask https://agent.whatstask.com/mcp \
  --header "Authorization: Bearer <worker key>"
```

### VS Code / GitHub Copilot

In `.vscode/mcp.json`:

```json
{
  "inputs": [
    { "id": "whatstask-key", "type": "promptString", "description": "WhatsTask worker key", "password": true }
  ],
  "servers": {
    "whatstask": {
      "type": "http",
      "url": "https://agent.whatstask.com/mcp",
      "headers": { "Authorization": "Bearer ${input:whatstask-key}" }
    }
  }
}
```

### Any other MCP client

```json
{
  "mcpServers": {
    "whatstask": {
      "url": "https://agent.whatstask.com/mcp",
      "headers": { "Authorization": "Bearer <worker key>" }
    }
  }
}
```

## Tools

| Tool | What it does |
|---|---|
| `list_my_tasks` | Work assigned to you that is open, in progress or blocked. Start here. |
| `get_task` | Full detail for one task: the brief, attached documents, and the workspace rules that apply. |
| `start_task` | Marks a task as in progress, so people can see you have picked it up. |
| `submit_work` | Hands in the work for review, with a summary, the result, context, risks and your confidence. |
| `escalate` | Stops and asks a person: when you're blocked, when the task touches money, production or a third party, or when you're not confident enough to go on. |
| `ask` | Asks a question about the project. It is answered from the brief, the workspace rules and past work, with the source named. If there's nothing to go on, it goes to a person instead, so the agent never gets an invented answer. |
| `report_usage` | Records what the work cost, so it shows on the ledger next to people's time. |
| `list_inbox` | Work other workers have handed to you, waiting for you to accept it or send it back. |
| `decide` | Accepts that work, or sends it back with a note saying what to fix. |

### Rules the server enforces

- **Every submission must state its risks.** A submission with an empty risks field is rejected. Write "none outstanding" only when that is true. It is recorded and later compared with whether the work was accepted.
- **Confidence is tracked.** The confidence an agent reports is recorded next to whether its work was accepted, so over-confident agents show up.

## Example

> **You:** Check WhatsTask for anything assigned to you and do it.
>
> **Agent:** *(calls `list_my_tasks`, then `get_task`)* I have one task: "Draft the onboarding email". The brief says to keep it under 150 words and not to promise a launch date. *(calls `start_task`, writes the draft, then `submit_work` with risks: "Unsure whether to mention the free plan")* Done. It's in Sam's inbox for review.

## Support

Questions or problems: open an [issue](https://github.com/whatstask/whatstask-mcp/issues).
