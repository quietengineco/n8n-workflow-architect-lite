# Workflow Architect for n8n (Lite)

Free agent skill: plan n8n workflows before building them, and check workflow JSON offline before you import it.

## What it does
- **Planning procedure** your agent follows with you: trigger, steps, data shape, credentials, failure plan, test cases, in a fill-in template.
- **Validator** (`scripts/validate_workflow.py`, Python 3 standard library, no network): JSON syntax with line/column, workflow structure, connection references (including connections keyed by node id), duplicate node names and ids, missing credential placeholders, likely hardcoded secrets, retry settings outside n8n's limits, unreachable nodes, missing sticky notes.

## Use
```bash
python3 scripts/validate_workflow.py my-workflow.json
```
Or ask your agent: "Plan an n8n workflow that ..." / "Check this n8n JSON before I import it."

## What the lite edition does not include
No node reference or typeVersion table (so no typeVersion checks), no worked examples, no error-handling snippets, no debug checklist. Those are in the full **Workflow Architect for n8n** skill.

## Requirements
An Agent Skills client (Claude Code, Cursor, Codex and others) and Python 3.8+.

Also runs piped, with no other files: `cat scripts/validate_workflow.py | python3 - my-workflow.json`.

Support: quietenginecompany@gmail.com

What was tested: the validator is unit-tested with automated checks (Python 3.8 and 3.13); not run in a live agent and not clicked through by a person.

## Full version
The full **Workflow Architect for n8n** skill adds a node reference and typeVersion table, worked example workflows, error-handling snippets and a debug checklist: https://quietenginecompany.com/products/

## Install
Copy this folder to `~/.claude/skills/` (Claude Code), `~/.cursor/skills/` (Cursor) or `~/.agents/skills/` (Codex).

---
Made with AI assistance. Independent project; not affiliated with or endorsed by n8n. MIT licensed (see `LICENSE.txt`).

Made by Quiet Engine Co. · https://quietenginecompany.com
