# Ovio task → integration relation

Session learning: Magnus wants Ovio tasks to be linkable directly to operational integrations.

## Schema pattern

For `Ovio – Oppgaver`, keep a relation property named exactly:

- `Integrasjon`

It should point to:

- `Ovio – Integrasjoner`
- database ID: `3616841f-dd02-81f0-9182-de3d2c1ab09b`

This lets each task page map to one or more integrations such as Vipps/MobilePay, Visma, Kravia.ai, NAF, SMS/email, etc.

## Workflow

1. Before changing schema, retrieve the current `Ovio – Oppgaver` database and verify whether `Integrasjon` already exists.
2. If absent, patch the tasks database properties with a Notion relation to `Ovio – Integrasjoner`.
3. Verify by retrieving the tasks database again:
   - property exists;
   - type is `relation`;
   - target database ID matches `Ovio – Integrasjoner`.
4. If the user points to a specific task screenshot/title, query the task and verify the new property appears on that page.
5. Do not auto-create a missing integration row unless the user explicitly asks. If a task references an integration not present in `Ovio – Integrasjoner` (e.g. Scrive/e-signering), report that and suggest the likely new integration name.

## Notion API shape

Using the standard databases API (`Notion-Version: 2022-06-28` in the current Ovio scripts):

```json
{
  "properties": {
    "Integrasjon": {
      "relation": {
        "database_id": "3616841f-dd02-81f0-9182-de3d2c1ab09b",
        "type": "single_property",
        "single_property": {}
      }
    }
  }
}
```

## User-facing explanation

Keep the report short and concrete:

- field added: `Integrasjon`;
- field type: relation;
- target: `Ovio – Integrasjoner`;
- whether the example task currently has any integration selected;
- if a relevant integration is missing, suggest creating it rather than silently guessing.
