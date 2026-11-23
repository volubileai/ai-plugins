# Actions and integrations

An action is something the agent does besides talking. Actions run at three moments: before the call (while it rings), during the call, and after the call.

## What the platform does not do

- No SMS, and no free-form email. The only built-in send is the post-call email. Anything else goes through an integration with another tool. When the user wants it, ask which tool they use, and offer an integration only if the integration tools are listed. Never present a send as built in.
- No sending during a call. The post-call email leaves after the call, and the agent never triggers it.

## Actions during the call

Use `list_live_call_action_types` to see the types that can be created and the settings each one takes. List the agent's actions with `list_live_call_actions` and read one with `get_live_call_action`.

| Type | Use | Settings the tool checks |
| --- | --- | --- |
| `TRANSFER_CALL` | Send the caller to a person or another number | A description, at least one destination (phone number and description each), a ring time of 5 to 120 seconds |
| `HANG_UP` | End the call under precise conditions | A description, the minimum number of attempts (1 to 5), the call status to record |
| `WAIT` | Pause the agent (hold, or a phone menu that is still talking) | A description |
| `DTMF` | Press keypad tones, to move through a phone menu | A description |

- A standard agent has at most one `HANG_UP`, one `WAIT` and one `DTMF`. Change the existing one with `update_live_call_action`.
- A new action is created switched off. The user switches it on in the Volubile app. Say so in the hand-over.
- A transfer created without opening hours has every day empty, so it is probably never offered. Ask for the days and the hours (`HH:mm`, end after start, two time ranges per day at most) and put them in.
- Phone numbers are in international format, like `+33612345678`. A transfer over a phone connection (SIP) can only use a connection that already exists in the workspace.
- The agent speaks before running some actions (transfer, hang up, knowledge search, workflow) unless `answerBeforeExecution` is `false`. Set it to `false` when the action should run without an introduction.
- Test an action with a real call before relying on it.
- Keep the agent's prompt free of action settings: see `reference/writing-the-prompt.md`.

A flow agent has no `create_live_call_action`: transfer, hang up, wait, keypad tones and knowledge search are added as steps or global nodes with `edit_truepath`. See `reference/call-flow-agents.md`.

A knowledge search uses a knowledge source from `list_rag_datasources`. Creating or replacing a knowledge source happens in the Volubile app, and a new import replaces the old content of that source.

## Before the call

`list_pre_call_actions` shows what runs while the call rings. Only workflows can be created here (see below). A workflow that runs before the call has 30 seconds, and what it returns becomes variables of the call.

## After the call: the email

`list_post_call_mail_actions`, `create_post_call_mail_action` and `update_post_call_mail_action`.

- Settings: `name`, then `config` with `subject` (required), `to` (at least one address), `cc` and `bcc` (empty lists if none), `tags` (the labels that trigger the email, any of them) and `statuses` (call statuses that must match, an empty list if none).
- The body of the email is always the call summary.
- `tags` are labels of the agent's classifiers (read with `get_agent_post_processing`). An unknown label is refused. Write the labels first, and have the user publish them before switching the email on.
- The subject can use: `tags` (the call's labels, comma separated: `{{ tags.split(',')[0] }}` is the first), `extractors['name']['field']`, `context.name`, `watchedVariables`, `summary`, `phone`, `startTime`, `duration`.
- An address is a lowercase address, or a `{{ variable }}` that holds one, such as an extracted email.
- The email is created switched off. An active email changes for the next calls once updated: confirm before changing one.

## Integrations (workflows)

Available only when the integration tools are listed. On a plan without integrations, the tools answer with a one-sentence refusal and a link: relay it and stop.

A workflow connects the agent to another tool or web service. It has three kinds by the moment it runs: `PRE_CALL` (before the call, its reply sets call variables), `LIVE_CALL` (when the agent calls it during the call), `POST_CALL` (after the call, with the call's data, its reply is ignored).

Order of work:

1. `list_webhook_actions` to see what exists, with each workflow's state and the values the agent sends.
2. `create_webhook_action` with the trigger. For `LIVE_CALL`, also give `liveCallTrigger`: a name, a description of when and why the agent calls it, and the values the agent sends (name, type `string`, `integer` or `boolean`, required or not). The action starts switched off with an empty workflow.
3. If the workflow needs an account of another app (a calendar, a CRM): `get_connection_link` with the app's `pieceName` from `ap_research_pieces`. Give the link to a person who can open this workspace in the Volubile app and sign in to the account to connect. It should be the business account, not a personal one. When they confirm, find the connection with `ap_list_connections`.
4. Build the steps. Look up apps with `ap_research_pieces` and `ap_get_piece_props`, read the flow with `ap_flow_structure`, add or change steps with `ap_add_step` and `ap_update_step`, and check with `ap_validate_flow`.
5. Test. `ap_test_flow` and `ap_test_step` run the workflow for real, so a step that creates a booking creates it. Tell the user which steps will act on their tools, and ask before each test. The test needs sample values for the trigger: pass them at the top level, not inside a `body`. Read the outcome with `ap_get_run`.
6. Publish with `ap_lock_and_publish`. This switches the workflow on at once: confirm first.
7. Hand-over: switch the action on in the Volubile app if the agent is a standard agent. On a flow agent, a published `LIVE_CALL` workflow needs no switch: link it with a workflow node in `edit_truepath`.

Rules that trip people up:

- A workflow reads the values the trigger sends directly as `trigger['output'].<name>`. Never `trigger['output'].body.<name>`.
- `update_webhook_action` replaces the whole `LIVE_CALL` trigger and renames the workflow. It changes the draft: calls see it after a publish.
- A step that uses an app account needs that connection to exist first (step 3).
- A workflow node of a flow agent needs a published workflow and gives every required value the workflow expects, and no unknown ones.
- Describe an integration to the user in plain words (what it reads or writes, in which tool), and note any account or permission the user's IT must provide.
