# Sanity Content Lake audit workflow

Use when asked whether the agent has Sanity access, who made recent CMS changes, or whether a robot/user performed edits. Keep the workflow read-only unless the user explicitly asks for content/schema changes.

## What to discover

1. Locate local project docs/credentials without printing secrets. Typical durable clues are README files naming project IDs, datasets, and token purposes.
2. Confirm token files exist and show metadata only: path, byte count, modified timestamp. Do not display token contents.
3. Query Content Lake for recent documents with GROQ to identify current content activity.
4. Query Sanity History API transactions to identify actual authors for recent changes.
5. List Sanity users including robots to map author IDs to human/robot names.
6. If checking whether a specific robot such as Jarvis acted, filter History API by that author ID and report latest matching transactions.

## Sanity HTTP patterns

Recent docs query:

```bash
curl -sS \
  -H "Authorization: Bearer $SANITY_AUTH_TOKEN" \
  "https://$PROJECT_ID.api.sanity.io/v2025-05-01/data/query/$DATASET?query=$(python3 - <<'PY'
import urllib.parse
print(urllib.parse.quote('*[_type != "system.group" && !(_id in path("_.**"))] | order(_updatedAt desc)[0..12]{_id,_type,_updatedAt,_createdAt,_rev,title,name,slug}'))
PY
)"
```

Recent transactions:

```bash
curl -sS \
  -H "Authorization: Bearer $SANITY_AUTH_TOKEN" \
  "https://$PROJECT_ID.api.sanity.io/v2025-02-19/data/history/$DATASET/transactions?excludeContent=true&reverse=true&limit=20"
```

Filter by author ID:

```bash
curl -sS \
  -H "Authorization: Bearer $SANITY_AUTH_TOKEN" \
  "https://$PROJECT_ID.api.sanity.io/v2025-02-19/data/history/$DATASET/transactions?excludeContent=true&reverse=true&limit=10&authors=$AUTHOR_ID"
```

Map author IDs to names/robots with the CLI:

```bash
SANITY_AUTH_TOKEN=[REDACTED]
```

## Reporting standards

- Separate **access found**, **latest activity**, and **attribution**.
- Say explicitly whether you performed only read-only checks.
- Attribute by Sanity author ID and mapped user/robot name; do not infer from document titles alone.
- If the latest activity belongs to a human but a robot has older transactions, report both the latest overall and latest robot-specific transaction.
- Never include raw token values, full secrets, or credential file contents in the final answer.

## Pitfalls

- `_updatedAt` on documents shows content recency, but not who changed it. Use History API transactions for attribution.
- Robot tokens often appear as separate Sanity users. Use `--robots` when listing users or you may miss them.
- Studio deploy tokens and content editor tokens have different purposes; do not use deploy tokens for content reads/writes when a content token exists.
