# Tools

Open this file before the first write, or when unsure what a tool does to the agent. The description each tool carries in the tool list is the final word: when it differs from this page, follow the tool.

The Volubile app is the web application where the user manages agents. Tool descriptions call it the back office.

## Who sees what

A tool the user's role or plan does not allow is missing from the tool list. A Viewer sees only the call and reference tools. An Editor or an Admin sees the agent tools too.

| Group | Tools | Viewer | Editor, Admin |
| --- | --- | --- | --- |
| Calls | `list_calls`, `count_calls`, `get_call` | yes | yes |
| Call event logs | `list_call_event_logs`, `get_call_event_log` | only if the plan includes call event logs | same |
| Reference data | `list_voices`, `list_rag_datasources` | yes | yes |
| Agent reading | `get_agent`, `get_agent_prompt`, `get_agent_post_processing`, `get_agent_voice`, `get_initial_message`, `get_agent_truepath`, `get_truepath_node`, `get_truepath_edge`, `get_truepath_global_node` | no | yes |
| Agent writing | `create_agent`, `update_agent_name`, `update_agent_prompt`, `edit_agent_prompt`, `update_agent_voice`, `update_initial_message`, `replace_agent_post_processing`, `edit_truepath` | no | yes |
| Actions | `list_pre_call_actions`, `list_live_call_actions`, `get_live_call_action`, `list_live_call_action_types`, `create_live_call_action`, `update_live_call_action`, `list_post_call_mail_actions`, `create_post_call_mail_action`, `update_post_call_mail_action` | no | yes |
| Integrations | `list_webhook_actions`, `create_webhook_action`, `update_webhook_action`, `get_connection_link`, and the `ap_` tools | no | yes, `get_connection_link`, `update_webhook_action` and the `ap_` tools only if the plan includes integrations |

`create_webhook_action` can be listed on a plan without integrations. It then answers with a refusal sentence: relay it as it is.

Two things have no tool here. Activating or deactivating an agent is done in the Volubile app, in the agent's settings, by an Admin of the workspace (an Editor cannot). Test scenarios are not available to workspace users. Send the user to the Volubile app or to https://support.volubile.ai.

## What each write does

**Saved as a draft.** Each call saves a new version of the agent. Nothing reaches callers until the user publishes the version in the Volubile app. These tools take `expectedVersion`.

| Tool | Changes |
| --- | --- |
| `update_agent_prompt` | The whole general prompt |
| `edit_agent_prompt` | Parts of the general prompt, by text replacement |
| `update_agent_voice` | The voice and, optionally, the languages |
| `update_initial_message` | The greeting of a standard agent |
| `replace_agent_post_processing` | All labels and collected data, replaced in full |
| `edit_truepath` | The conversation flow of a flow agent, including each step's prompt |

**Applied at once.**

| Tool | Effect |
| --- | --- |
| `create_agent` | Creates an active agent with the workspace's default voice, no actions and nothing published. Needs a plan that includes outbound agents when the type is OUTBOUND. |
| `update_agent_name` | Renames the agent now. Up to 100 characters. |
| `create_live_call_action` | Creates an action switched off. The user switches it on in the app. Standard agents only: a flow agent gets the same capabilities as global nodes with `edit_truepath`. |
| `update_live_call_action` | Replaces an action. An active action changes from the next call. On an active transfer, destinations and phone connection cannot change: the user switches the action off in the app first, or you create a new action. |
| `create_post_call_mail_action` | Creates a post-call email switched off. |
| `update_post_call_mail_action` | Replaces the name and settings. An active email changes for the next calls. |
| `create_webhook_action` | Creates an action switched off and an empty workflow for it. The user finishes and switches it on in the app. |
| `update_webhook_action` | Renames the workflow and replaces the trigger the agent sees. The change is in the workflow's draft: calls use it once the workflow is published. |

**Workflow tools (`ap_` tools).** Reading tools change nothing. `ap_add_step`, `ap_update_step` and `ap_delete_step` change the workflow's draft. `ap_lock_and_publish` publishes the workflow and switches it on at once. `ap_test_flow` and `ap_test_step` run the workflow for real in a test environment: a step that writes to the user's calendar or CRM writes there. Ask before testing such a step. See `reference/actions-and-integrations.md`.

## Guards on writes

- Every write on an agent takes `expectedAgentName`: the exact name returned by the last `get_agent` (upper and lower case do not matter). After `create_agent` or `update_agent_name`, use the new name.
- The draft tools above take `expectedVersion`: `latestConfigurationVersion` from the last `get_agent`, or the version the previous draft write returned. One call makes one version, whatever the number of changes inside it. Never work out the next version yourself.
- When a tool answers that the name or version is out of date, read the agent again, show the change again and ask again.
- Send object arguments as JSON objects, not as strings that contain JSON.
- A refused write names the faulty field (for a list, its position, like `operations[1]`) and lists every problem of that call together. Fix them all and send the whole list again.

## Reading what the read tools leave out

- `get_agent_truepath` is an overview: no step prompts. Read a step with `get_truepath_node` or `get_truepath_global_node`, and the condition of a transition with `get_truepath_edge`.
- `list_live_call_actions` gives each action's id, name, status and type. Read one in full with `get_live_call_action`. A workflow action's details come from `list_webhook_actions`.
- `get_agent_prompt` says when the prompt has sections only Volubile admins can edit. Those sections are kept when the prompt is written back. Jinja blocks come fenced with three backticks and `jinja2`: keep the fences when writing the prompt back.
- `list_voices` lists at most 50 voices and gives the total. When the total is higher, filter by `locales` (all of the agent's languages) and `provider`.
- `update_agent_voice` keeps the agent's languages unless `locales` is given. Every language after the first adds a second speech recognizer to each call and raises the cost, so add one only when the user asks for it.

## Errors

| Answer | Meaning |
| --- | --- |
| `Resource not found` | The id is wrong, or the agent belongs to another workspace. Say so and ask the user to check the id. |
| `Access denied` | The user's role or workspace does not allow it. |
| `Invalid argument: ...` | The text says what to fix. Out of date name or version: read again. |
| A sentence mentioning the plan or support | A refusal. Relay it as it is with its link. |
| `Internal error` | Say it failed, do not retry in a loop, point to https://support.volubile.ai. |
