# Ovio project task views — Notion workflow notes

Use this when building or troubleshooting project-specific task dashboards in Notion, especially for Infinity Drift/Ovio.

## Durable pattern

- Keep one master task database: `Ovio – Oppgaver`.
- Keep one project database: `Ovio – Prosjekter`.
- Tasks should have `Prosjekt` as a relation to `Ovio – Prosjekter`.
- Project pages should contain linked views of `Ovio – Oppgaver` filtered to the current project.
- Avoid separate per-project child databases unless intentionally creating independent task stores; they look like boards but fragment the task model.

## API vs UI boundary

Notion API can safely manage:

- page/database/data source metadata
- properties, including relations
- pages/items and page blocks
- child databases created inside pages
- text scaffolds/callouts/instructions on project pages

Notion API should not be assumed to manage these UI objects:

- linked database blocks/views of an existing database
- database view tabs
- board/table/timeline layout settings
- group-by/sort/visible-property view settings
- database templates with linked views filtered to “this page”

When the desired outcome is a filtered linked view, use API to prepare the page scaffold and relation model, then give precise UI steps for the linked view.

## Side peek pitfall

When a row in a Notion database opens in side peek, Full width affects only the side panel and the page still feels cramped. Tell the user to open the page as a full page first:

- click the expand/open-as-page icon in the side peek header, or
- right-click the database row and choose Open as full page.

Only then set `••• → Full width` and add linked views.

## Manual linked view steps

For each project page:

1. Open the project as a full page.
2. Enable `Full width`.
3. Under `Oppgaver`, type `/linked`.
4. Choose `Linked view of database`.
5. Select `Ovio – Oppgaver`.
6. Set layout to `Board`.
7. Set `Group by` to `Status`.
8. Add filter: `Prosjekt contains <this project>`.
9. Save for all when Notion asks about saving the view configuration.
10. Rename the view to `Board` or `Oppgaver · Board`.

Optional duplicate views:

- `Table`: same project filter, table layout.
- `Mine`: same project filter + `Ansvarlig contains Me`.
- `Ferdig`: same project filter + `Status = Ferdig`.

## Board column order

For boards grouped by `Status`, column order follows the order of options in the `Status` property on the master task database. Set the global status option order once, e.g.:

`Innboks → Backlog → Prioritert backlog → Denne uken → Pågår → QA → Ferdig`

This affects all linked views grouped by that property, which is usually desirable for consistency.

## Communication pitfall

When the user asks why a Notion layout looks wrong, inspect the specific spatial issue before giving generic Full width advice. In this session the key issue was a large gap between the Notion sidebar and the first board column caused by a centered page container/inline database context, not merely “the page is not full width.”