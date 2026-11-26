# Labels and collected data

After each call, Volubile can put labels on it and collect data from it. Both are called post-call processing in the tools: `get_agent_post_processing` reads it and `replace_agent_post_processing` writes it as a draft. Nothing changes for callers until the user publishes the draft in the Volubile app.

## Classifiers (labels)

A classifier puts one label on a call, such as the reason for the call or how it ended. Labels show in the call list and on the call page, and they can trigger a post-call email.

- **A classifier has a name, one short description and a list of labels.** The description is one sentence of under 200 characters starting with a verb ("Classifies the reason for the call."). It holds no label definitions.
- **Each label has its own name and description.** The description says what can be seen in the call, in one or two sentences, with synonyms and edge cases.
- **Always add a catch-all label** ("Other calls") with a description like "Any call that fits no other label of this classifier."
- **Names use letters, digits, spaces and `. - [ ] ( )`**. Classifier and label names also accept an apostrophe. Quotation marks and other symbols are refused.
- **No label appears twice**, inside a classifier or across the agent's classifiers. Names are compared without regard to upper or lower case.
- **Two classifiers exist on every agent**, about how the call ended and how satisfied the customer was (read their exact names with `get_agent_post_processing`). Do not create another one for the same idea: adapt its labels.
- **A post-call email is triggered by labels of one classifier**, any of them (an OR). It cannot require a label of one classifier AND a label of another. When an outcome needs a combination, make it one label of one classifier.
- **No hierarchy**: a classifier never depends on another. When a distinction only makes sense in one context, put the context in the labels.
- The order of labels is not kept. Nothing may rely on it.
- A default label can be set for calls where nobody spoke (`defaultNoInteractionClassification`): it must name a classifier and one of its labels that exist.

Example: a classifier "Call reason" with the labels "Opening hours", "Appointment", "Complaint" and "Other calls", each with its own description.

## Extractors (collected data)

An extractor collects one piece of information during the call: a name, a phone number, a reference, a date.

- **Name:** unique, in letters, digits, spaces and `. - [ ] ( )`. No apostrophe.
- **Description:** what to extract, the format expected, where it appears and how the caller says it. "Customer file number, dictated one digit at a time" is better than "file number". For a phone number, "Customer phone number in international format".
- **Type:** general information (free text) or date. A **group** gathers related extractors under one name (an address, a booking). A group has sub-extractors and no type of its own.
- No two extractors share a name, upper or lower case ignored.

## Writing it

- `replace_agent_post_processing` **replaces everything**. Read with `get_agent_post_processing` first and send everything that must stay: a block left out is emptied.
- Send back the `id` of each existing item you keep, or it loses its identity. Leave the `id` out only for a new item.
- The server rebuilds the system classifiers: send nothing for them.
- Write labels before any post-call email that uses them. The user publishes the labels before switching the email on.
- Send the whole object as JSON, not as text.
