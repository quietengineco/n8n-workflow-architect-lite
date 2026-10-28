# Planning an n8n workflow (before writing any JSON)

Fill in this plan with the user first. Ask only for what you cannot infer; propose defaults for the rest and mark them as assumptions. Show the finished plan and get a "yes" before building.

## 1. Outcome (one sentence)
"When <event>, take <data>, do <transformation/decision>, and put the result in <destination>."
If you cannot write this sentence, the request is not clear enough yet. Ask.

## 2. Trigger
Pick exactly one production trigger, plus a Manual Trigger for testing.

| Situation | Trigger node | Notes |
|---|---|---|
| Another system can call a URL | Webhook (`n8n-nodes-base.webhook`) | Decide method (usually POST), path, and response mode: answer immediately (`onReceived`), after the last node (`lastNode`), or from a Respond to Webhook node (`responseNode`). |
| A person submits data | Form Trigger (`n8n-nodes-base.formTrigger`) | Built-in hosted form. |
| On a timetable | Schedule Trigger (`n8n-nodes-base.scheduleTrigger`) | Write the cron or interval and the timezone (workflow settings, `timezone`). |
| An app event (new row, new email, new issue) | That app's trigger node | Needs a credential. Polling triggers check on an interval. |
| Called by another workflow | Execute Workflow Trigger (`n8n-nodes-base.executeWorkflowTrigger`) | Define the input fields the caller must send. |
| Handle failures of other workflows | Error Trigger (`n8n-nodes-base.errorTrigger`) | See the n8n docs on error workflows. |

Always add a Manual Trigger ("Run test") that feeds mock data into the same first processing node, so the workflow can be run without the real event.

## 3. Data shape
Write the item shape at three points: what the trigger emits, what the middle steps need, and what the destination expects.

```
trigger out:   { body: { email: string, name?: string, items: [{ sku, qty }] } }
after clean:   { email (lowercase), name, item_count: number, received_at: ISO string }
destination:   Sheet "Leads" columns: email | name | item_count | received_at
```

Rules of thumb:
- n8n passes a list of **items**; each item is `{ "json": { ... } }`. Most nodes run once per item.
- Decide where one item becomes many (Split Out) or many become one (Aggregate, Summarize, Code with "run once for all items").
- Note which fields can be missing or empty and what should happen then (default, skip, or fail).
- Note volume: rows per run, payload size, API rate limits. Over ~100 calls per run, plan batching (Loop Over Items) and waits.

## 4. Steps
List each node in order, with its type, its job in one line, and its branch. Example:

```
1. Run test (Manual Trigger)        -> 3
2. New order (Webhook POST /orders) -> 3
3. Normalize (Edit Fields / Set)    trim, lowercase email
4. Valid? (If)                      true -> 5, false -> 7
5. Append row (Google Sheets)       creds: googleSheetsOAuth2Api
6. Reply 200 (Respond to Webhook)
7. Reply 400 (Respond to Webhook)
```

Prefer built-in nodes over Code when a node does the job (Set, If, Filter, Switch, Merge, Sort, Remove Duplicates, Split Out, Aggregate, Summarize). Use Code for real logic that would otherwise take 4+ nodes. Keep each Code node short and commented.

## 5. Credentials
List every external service, the n8n credential type the node uses, and who will create it. Never put keys, tokens or passwords in the workflow JSON. In the JSON, reference credentials with a placeholder:

```json
"credentials": { "slackApi": { "id": "REPLACE_WITH_CREDENTIAL_ID", "name": "Slack account" } }
```

The user picks the real credential on that node after import. For HTTP Request calls to an API without a dedicated node, use Authentication = Generic credential type (Header Auth, Basic Auth, OAuth2) instead of typing the key into a header.

## 6. Failure plan
For each external call, decide:
- **Retry?** Transient errors (timeouts, 429, 5xx): turn on Retry On Fail.
- **Then what?** Stop the run (default), continue with the error as data, or route to an error output.
- **Who hears about it?** A shared error workflow (Error Trigger) that alerts someone.
- **Re-runs:** if the workflow runs twice on the same input, does it create duplicates? If so, dedupe on a key.

## 7. Test plan
- Mock data that covers: a normal item, a duplicate, a missing/invalid field, and an empty input.
- Expected result for each, written down before running.
- A "dry run" switch (a boolean in a Config/Set node, checked by an If node before any write or send) so the user can run end to end without side effects.

## 8. Plan template (copy, fill, show the user)

```
Workflow: <name>
Outcome: When ..., take ..., do ..., put ...
Trigger: <node> (+ Manual Trigger "Run test" with mock data)
Data in:  {...}
Data out: {...}
Steps:
  1. ...
Credentials needed: <service> -> <credential type> (user creates)
Failure plan: retries on <nodes>; error branch <...>; error workflow <yes/no>
Dedupe key: <field or none>
Dry run: <Config.dry_run, default true>
Test cases: normal / duplicate / invalid / empty -> expected ...
Assumptions: ...
```

## 9. Then build, then check
1. Write the workflow JSON (one top-level object: `name`, `nodes`, `connections`, `settings`).
2. Add sticky notes: overview (what it does, how to test, how to go live) and setup (credentials, IDs to replace).
3. Run `python3 scripts/validate_workflow.py <file>.json` and fix every error. Explain any warning you keep.
4. Tell the user exactly what to replace after import (credentials, sheet IDs, channel names, URLs) and how to run the test.
