---
name: n8n-workflow-architect-lite
description: Plans n8n workflows before any JSON is written (trigger, steps, data shape, credentials, failure plan, test cases) and checks n8n workflow JSON with a bundled offline validator for JSON syntax errors, broken or id-keyed connections, duplicate node names and ids, missing credential placeholders, likely hardcoded secrets, retry settings outside n8n's limits and unreachable nodes. Use when the user is designing an n8n automation, wants to check n8n workflow JSON before importing it, or gets an n8n import error about invalid structure, duplicate node names or connections.
license: MIT. See LICENSE.txt
compatibility: Uses the open Agent Skills format supported by Claude Code, Cursor, Codex and others. The validator needs Python 3.8+ (standard library only, no network access).
metadata:
  author: Quiet Engine Co.
  version: "1.1.0"
  tags: "n8n,workflow-automation,n8n-json,validator,planning"
---

# Workflow Architect for n8n (Lite)

Two jobs: plan an n8n workflow properly before building it, and check workflow JSON before it is imported.

## Guardrails
1. Never put API keys, tokens or passwords in workflow JSON. Use credential placeholders (`"id": "REPLACE_WITH_CREDENTIAL_ID"`) and tell the user which credential to connect.
2. Say "validated" when only the validator ran. Do not claim a workflow works unless it was imported and run.
3. Do not import, publish or run anything in the user's n8n unless they ask.

## Plan
Follow `references/planning.md` and fill in its plan template with the user: outcome sentence, trigger (plus a Manual Trigger with mock data for testing), data shape in and out, steps, credentials, failure plan, dedupe key, dry-run switch, test cases and assumptions. Ask at most 3 focused questions, state the rest as assumptions, and get the user's OK on the plan.

## Check workflow JSON
```bash
python3 scripts/validate_workflow.py workflow.json            # human-readable
python3 scripts/validate_workflow.py --json workflow.json     # machine-readable
python3 scripts/validate_workflow.py --strict workflow.json   # warnings fail too
```
Use `-` to read the workflow from stdin. Exit codes: 0 = no errors, 1 = errors (or warnings with `--strict`), 2 = file unreadable or not valid JSON.

How to use the result:
1. Fix every ERROR first. Common ones: `json-syntax` (line and column given), `duplicate-name`, `bad-connection-target`, `bad-connection-source` (often connections keyed by node id instead of name), `missing-credentials`, `hardcoded-secret`.
2. Then warnings: `unreachable` nodes, `retry-clamped` (n8n limits retries to 2-5 tries and up to 5000 ms between them), `no-sticky-notes`, `no-trigger`.
3. INFO lines list credential placeholders the user must connect after import.
4. Re-run until clean, then report the final output to the user, plus what they must set up after import.

This lite edition has no node-version table, so the validator skips typeVersion checks and says so in its output.

## Files
- `references/planning.md`: planning procedure and template
- `scripts/validate_workflow.py`: the validator (Python 3 standard library, offline)
