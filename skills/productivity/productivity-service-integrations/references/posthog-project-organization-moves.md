# PostHog project moves between organizations

Authoritative source: https://posthog.com/docs/settings/projects#moving-projects-between-organizations

## Same-region move

When source and destination organizations are in the same PostHog region (EU→EU or US→US), the move is self-service and free for all plan levels:

1. Open project settings.
2. Find **Move project**.
3. Select the destination organization.
4. Confirm.

This transfers project ownership without physically relocating event data.

Requirements:

- The operator must be owner/admin of the source organization.
- The operator must be a member of the destination organization.
- The source organization must have at least two projects.
- A free organization limited to one project may need a temporary upgrade/billing setup to create a placeholder second project; it can be downgraded afterward.

Access consequence: source-organization members lose access after the move unless they are also members of the destination organization. Verify personal API keys, service users, destinations, and automations afterward. Avoid claiming every ingestion token changes; verify project and API behavior after the move.

## Cross-region move

EU↔US is a physical data migration. It is available only on Scale or Enterprise and must be coordinated through in-app PostHog support/engineering.

## Assessment format

Lead with the branch that determines difficulty:

- same region + source has 2+ projects → a few-minute UI operation;
- same region + source has only 1 project → simple move but temporary upgrade/placeholder may be required;
- cross-region → support-led migration on Scale/Enterprise.

Before any write, inspect organization names, regions, membership, role, project count, billing plan, integrations, and key ownership read-only. Do not initiate the move without explicit approval.