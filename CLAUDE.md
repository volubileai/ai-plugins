# ai-plugins

This repository is public. Everything in it is published in the Claude directory and the OpenAI directory (ChatGPT and Codex), and anyone can read it.

## Never commit

- A secret: API key, token, password, client secret, private key, connection string, `.env` file, test account credentials.
- An internal address: only `https://mcp.volubile.ai/v1/mcp` and the public `volubile.ai` pages may appear. No preprod, localhost or internal host.
- Customer data: customer or agent names, IDs, phone numbers, emails, transcripts or any call content, even as an example.
- Internal material: staff names or emails, internal tools and processes, ticket links, anything written for Volubile staff rather than for customers.

## Before every commit

1. Read the diff with the list above in mind.
2. Run the scan and check every hit by hand:

   ```bash
   git grep -nIiE 'secret|token|passw|api[_-]?key|bearer|private key|client_id|preprod|localhost|127\.0\.0\.1|@volubile\.ai|[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-'
   ```

3. Run `claude plugin validate .` and fix what it reports.

## Layout

- `.claude-plugin/plugin.json`: the Claude manifest.
- `.codex-plugin/plugin.json`: the OpenAI manifest, for ChatGPT and Codex.
- `.mcp.json`: the server both manifests point at.
- `skills/volubile/`: the skill, written for customers (Admin, Editor or Viewer of a Volubile workspace).

Raise `version` in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `metadata.version` in `skills/volubile/SKILL.md` on every release.
