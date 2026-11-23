# Call flow agents

A call flow agent is built as a graph of steps instead of one prompt. The tools call this mode TRUEPATH: `create_agent` with `builderMode` set to `TRUEPATH`, then `edit_truepath`. A flow agent still has a general prompt, read at every step on top of the step's own prompt: write it with `update_agent_prompt`.

## When to choose one

Choose a flow agent when the call needs a real branch:

- different behavior open and closed hours, chosen by the clock;
- several sites, brands or queues, chosen by the number called or a value collected;
- a path that depends on data fetched during the call;
- several distinct paths, or chained integrations.

A call that follows one path, however many topics it covers, stays a standard agent. A flow agent adds delay to each turn: do not propose it by reflex. When you propose one, show the user the map of steps (each step, its role in a sentence, its transitions and their conditions) and get approval before writing.

## Building blocks

| Block | Kind | Use |
| --- | --- | --- |
| Prompt (`LLM`) | step | A part of the conversation with its own prompt |
| Say (`SAY`) | step | A fixed sentence. Every flow starts with one: it holds the greeting |
| Router (`ROUTER`) | step | Branches on conditions, such as the hour or a value |
| List (`LIST`) | step | A fixed table to look up (postcodes, references) |
| Transfer list (`TRANSFER_CALL_LIST`) | step | Choose one recipient among several, with hours per line |
| Transfer (`TRANSFER_CALL`) | step and global action | A step sends to one number. A global action offers several named numbers |
| Hang up (`HANG_UP`) | step and global action | End the call |
| Workflow (`WEBHOOK`) | step and global action | Call an integration |
| Knowledge (`RETRIEVE_DOCUMENTS`) | global action | Search a knowledge source |
| Keypad (`DTMF`), Wait (`WAIT`) | global action | Phone menus and hold |

Steps are linked to each other. Global actions can be called from prompts but are not linked to each other.

Use a List step instead of a knowledge search for exact matches. A List step never has an LLM transition and always returns to the Prompt step that called it: when the flow must continue elsewhere afterwards, split that Prompt step in two.

## Rules for writing the graph

- The first step is a Say step with the greeting. It is linked to the main step by a direct transition. The agent's own greeting field is not used by a flow agent.
- A transition leaving a Prompt step is an LLM transition, with a name and a condition prompt. A transition leaving any other step is direct or a logic condition. The tool refuses the other combinations.
- A step has only one direct transition leaving it.
- Name every transition: the name is its label in the app.
- The start step and the main step must be active. A router needs at least one transition leaving it.
- A router branch is a logic condition: an OR of ANDs of triples `firstValue`, `operator`, `secondValue`. For open hours, use the operators `NOW_IN_TIMESLOT` and `NOW_NOT_IN_TIMESLOT`: they read the clock, and the time ranges go in `secondValue`. Do not compare `{{ system.now }}`, which is display text. Compare a time with `system.timeHHMM`, a number.
- Give every step and global node a readable name and a position, or they stack on one line with their type as the name.
- Changing a transition means `removeEdge` then `addEdge` in the same list: they are applied together. Removing a step does not remove its transitions: remove them in the same list.
- Variables used in a prompt must be produced earlier in the flow (collected by a transition, returned by a workflow, set by a List step). The tool refuses a name nothing produces.
- A name that becomes a tool name (an LLM transition or a global action) must be unique, and cannot become `hang_up` or `main-prompt`.

## Writing

`edit_truepath` takes a list of operations applied all together, and saves one draft. Its description lists the operations and their fields: follow it. Read the graph first with `get_agent_truepath`, and read a step with `get_truepath_node`. Change only a step's prompt with `updateNodePrompt`. The user publishes the draft in the Volubile app.
