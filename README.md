# Volubile

Volubile builds voice AI agents that answer your phone calls. This package connects Claude, ChatGPT and Codex to your Volubile account so you can review calls and manage your agents in a conversation.

## What it does

- Lists calls and reads their transcripts and event logs, to understand why an agent answered the way it did.
- Counts calls over a period.
- Reads and edits the prompt, voice, initial message, post-call processing and conversation flow of an agent.
- Creates agents and manages their live and post-call actions.
- Lists the voices and knowledge sources available to your account.

A bundled skill teaches the assistant how to use these tools: which one to call, in which order, and how to read call logs.

## Sign in

The first time you use it, you are sent to the Volubile sign-in page. Each operator signs in with their own Volubile account, no shared key or password is stored in this package. The assistant only sees what your account can see in the Volubile app.

On Claude (chat, Cowork), add the Volubile connector from the Connectors tab of the plugin and sign in. On Team and Enterprise plans, an Owner adds it first. On ChatGPT Enterprise and Edu, a workspace admin must enable it first.

## Roles

Your role in Volubile decides what the assistant can do for you:

| Role | What the assistant can do |
| --- | --- |
| Viewer | Read-only on calls: list calls, read transcripts and event logs, count calls. List voices and knowledge sources. |
| Editor | Everything a Viewer can do, plus read and edit agents (prompt, voice, initial message, post-call processing, conversation flow), create agents and manage their actions. |
| Admin | Everything an Editor can do. Settings reserved to admins in the Volubile app stay reserved to them. |

Some tools depend on the options of your plan, for example integrations. A tool you cannot use is not listed.

## Data

The package contains text files only and runs no code on your machine. The assistant sends your requests to the Volubile server at `https://cts.eu.volubile.ai/v1/mcp`, and nothing is sent anywhere else. Call data, including transcripts, stays in your Volubile account and is returned to the conversation only when you ask for it.

## Support

Questions or problems: https://support.volubile.ai

Website: https://volubile.ai

## License

MIT, see `LICENSE`.
