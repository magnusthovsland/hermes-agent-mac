# Notion project/task dashboard patterns

Session-derived guidance for organizing project dashboards when a master task database and a master project database already exist.

## Recommended model

Use two master databases:

- `Projects` database: one row/page per project.
- `Tasks` database: one row/page per task.
- `Tasks.Project`: relation to `Projects`.

Avoid separate per-project task databases or manually duplicated board pages. They fragment task data and make status/properties inconsistent.

## Project-specific task views

The preferred Notion UI pattern is:

1. Open a project page inside the `Projects` database.
2. Turn on **Full width** for the project page/template.
3. Add a **Linked view of database** pointing to the master `Tasks` database.
4. Filter the linked view by the project relation:
   - `Project` contains the current project page / template page.
5. Recreate the same useful layouts as the master task database:
   - Board grouped by `Status`
   - Table/list for all tasks
   - Timeline/calendar if dates exist
   - Mine/open/bug/finished views as needed
6. Save this as the default project template so every project page gets the same task dashboard.

This gives one project overview database plus project pages with task boards filtered to that project, without duplicating tasks.

## Notion limitations to remember

- Database view tabs belong to one database only. A view tab under `Projects` cannot directly show rows from `Tasks` as if it were a native project-database view.
- Use linked database views inside project pages for cross-database dashboards.
- Notion's public API can inspect and update databases/pages/properties/blocks, but database view configuration and database templates are largely UI-only. Do not promise to create board/table/timeline views or project templates via API unless the currently available API/tooling explicitly exposes view/template mutation.

## Layout pitfall

A linked/inline database inside a normal Notion page can appear visually shifted far to the right because Notion centers the page content container. This is especially visible with the sidebar open and kanban boards: the first column may start hundreds of pixels away from the sidebar.

When diagnosing screenshots, distinguish:

- full-page database: usually left-oriented close to the sidebar and uses database layout directly;
- normal page with inline/linked database: constrained/centered page container unless **Full width** is enabled.

For kanban/task dashboards, prefer full-page databases for the master task database and **Full width** project pages/templates for embedded linked views.