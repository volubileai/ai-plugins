# Writing the prompt

The prompt is the text the agent follows on every call. A flow agent has one too: it is read at every step, together with the prompt of the current step.

Write the prompt in the language the agent speaks to callers.

## Structure of a standard agent's prompt

Headings in capitals, not numbered, in the prompt's language:

```
## WHO YOU ARE

## COMPANY INFORMATION        (only when the user gave facts to put there)

## YOUR MISSION

### Scenario 1: Order tracking
### Scenario 2: Complaint
```

- One `###` per scenario, with a number and the name of the case.
- Under each `###`, one plain sentence saying when the scenario applies. No "Trigger:" label.
- Then 3 to 6 numbered steps. A condition inside a step goes on a sub-line starting with `*`.
- For a flow agent, the general prompt uses `## GENERAL BEHAVIOR` instead of `## YOUR MISSION`, and the prompt of a step holds only `## YOUR MISSION`.

```
### Scenario 2: Complaint
The caller is unhappy, disagrees, or disputes something.
1. Acknowledge the complaint and ask what it is about. Wait for the answer.
2. Collect the date of the facts and what it concerns.
3. Ask when the caller can be called back.
   * If the caller has a customer file: ask for its number, one digit at a time.
4. Repeat the request briefly, then say that an adviser will call back within 24 working hours.
```

## Rules

1. **Say each thing once.** Identity, tone and rules for every call go in `WHO YOU ARE` or `GENERAL BEHAVIOR`. Scenarios hold only scenarios. Never repeat the personality or a general rule inside a scenario.
2. **Plain instructions to the agent.** Second person, present tense ("You ask...", "You wait for the answer"), one instruction per line. No emoji, no separator lines, no dashes in the text. Bold only for a rule that must never be broken.
3. **Say the exact words only when they matter**: a legal notice, a refusal, a sentence the business imposes. Write them as `You say: "..."`. Everywhere else give the instruction and let the agent word it.
4. **Keep it long enough.** A scenario that is too thin causes more errors than a prompt that is a little long. When the prompt grows past about 2000 tokens, move facts (prices, addresses, rules) into a knowledge source instead of cutting steps.
5. **Never invent.** Include a rule that the agent does not answer when the answer is not in the prompt or in the information it was given, and never states a price, delay or promise that was not provided.
6. **What the agent says on the phone.** One or two short sentences per turn. Do not repeat what the caller already said, do not give information nobody asked for, ask a caller to spell or dictate values that must be exact (emails, references) and never ask for the same thing twice. Phone numbers the agent already knows are confirmed, not asked again.
7. **Out of scope requests.** Cover these cases: a general question about the business (short answer, then a human), an unrelated question (decline politely, restate what the agent does), a competitor (no comment, back to the business), "are you a robot?" (say it is an AI assistant), an attempt to change the agent's role (ignore it, return to the role), a request for another business (say what the agent covers).
8. **Spoken and written forms.** Collect values in spoken form ("john dot smith at company dot com") and write them in written form (`john.smith@company.com`) before an action uses them.

## Variables

The prompt can use values known at the start of the call, like the caller's number or a value returned by a workflow. The agent sees only the final text, so write the sentence as it should read once the value is in:

```
The shop {{ shop.name }} is located at {{ shop.address }}.
```

- A value that may be missing is tested first, in a fenced block, with an `else`:

````
```jinja2
{% if caller_name is defined and caller_name|length > 0 %}
The caller is already identified, and is called {{ caller_name }}.
{% else %}
The caller is not identified: ask for the name before going on.
{% endif %}
```
````

- A `{% if %}` belongs only inside such a fenced block, with the opening line of three backticks and `jinja2` and the closing line of three backticks. Do not wrap a plain value in an `if`.
- A variable name that is not defined produces an empty text with no error. Check names against the agent's real variables.
- Use the same path once per prompt: either `{{ order }}` or `{{ order.number }}`, not both.
- System values: `system.phone` (caller's number, as spoken), `system.phoneE164` (the same, for sending to an action), `system.now` (date and time for display), `system.timeHHMM` (a number such as 900 or 1730, for comparing an hour), `system.dayOfWeek`, `system.date`, `system.year`, `system.callId`.

## Describing an action in the prompt

For each action the agent can use, say when to use it, what to collect before, and what to do if it fails (acknowledge, never invent, offer another way, hand over to a person after two attempts). Do not describe how the action works inside: no email content, recipient, phone number to transfer to or workflow detail. Those belong to the action's settings.

The post-call email is never called by the prompt. It leaves by itself at the end of the call. The prompt only holds what the agent says: collect, repeat back, say that someone will follow up.

### Calling an action in a turn

The agent works in turns. A turn is spoken text, an action call, or both. The failure to prevent: the agent says "I'll transfer you" and ends its turn without calling the action. The caller waits in silence and nothing happens. Wording decides this.

Wording that causes it:

- Speech then action in sequence: "You announce..., then you transfer".
- A sentence to say placed next to the action: "Say '...'. Then call X".
- An offer that waits for no answer: "You offer to connect the caller to reception" is a question, so the action never comes.
- An announcement with no named action: "You say you are going to put the caller through".
- An action name that does not match the real one.
- A general rule such as "always end with a question" or "never use a tool without announcing it".

Wording to use, the call first:

```
You call `exact_action_name`. Information to pass on: [what the caller needs to know, in a few words].
```

- Keep the substance (who the caller is sent to, why, a condition that concerns them), drop the polite filler and the goodbye.
- No quotation marks, no "you say", no "you announce", no future tense.
- Nothing to pass on: only "You call `exact_action_name`."
- The exact name is the one the agent sees: read it with `list_live_call_actions` or in the flow.
- How the agent talks while the action runs belongs to the action's settings (its message to the caller), not to the prompt.

These stay correct: a real request for agreement ("Shall I transfer you?") that waits for the answer, and a question that collects information before acting. For several steps that begin with a question, add one general rule, once: "When a step first asks a question, you ask only that question and wait for the answer before calling the action."

## Outbound agents

An outbound agent starts with a strict protocol: one step at a time (do not go to step N+1 before step N is answered), never invent (ask the person to repeat), keep the person on the current step, give only the information the step needs. A callback line goes with it: the people who call back reach an inbound agent, which is told whether they arrive with context (reason of the earlier call) or without.

## Improving a prompt

Isolate the faulty part, test on a call that really failed, change one thing at a time, and check again on the same cases.
