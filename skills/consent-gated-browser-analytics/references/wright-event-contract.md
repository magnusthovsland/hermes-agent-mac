# Wright event-design contract

Apply these user-approved design conventions when working on Wright analytics. This is a specification, not proof of implementation or permission to deploy. Re-read current source and obtain action-specific approval before external writes or production release.

## Scope and semantics
- Keep Wright and Ovio in separate PostHog setups. Wright's final continuation click is intent to proceed, not successful login, payment or purchase in Ovio.
- Use `$pageview` for actual page openings, including SPA route changes. Do not treat summary-page arrival as a button action. Check existing reports before retiring legacy named `Sidevisning: ...` events.
- Use explicit actions at real user activation, not component mount, automatic defaults or page views. Preserve keyboard activation and avoid duplicate handler captures.
- Include readable names alongside stable IDs; analysts should not need database access. Names describe the entity at event time; IDs support grouping across renames.
- A user accepting an event specification does not authorize production release or resolve separate consent-policy decisions.

## Agreed ten actions
1. `Navigasjon klikket`: top-level navigation such as Våre avdelinger/Førerkort; navigation_item, placement, destination_path.
2. `Avdeling valgt`: specific office selection from directory, map or selector; office_id, office_name, office_slug, placement. Cover the actual directory links as well as package selectors; no event for automatic preselection.
3. `Førerkortklasse valgt`: license_class, license_name, transmission where relevant, placement and known office. Direct license choice in navigation should not additionally emit Navigasjon klikket for the same action.
4. `Pakke åpnet`: product_id (package product), product_name, product_type=package, known office and placement.
5. `Kursdetaljer åpnet`: office_product_id, course_name, office and placement; mapped to `Se alle datoer` inside the office-page course card, not an alias for reservation or selecting a date. The existing card renders that control when the same course group contains more than three occurrences; three dates are initially shown. Emit on expansion, not collapse. Verify current render conditions and responsive visibility before testing. The clearer name `Flere kursdatoer vist` is a proposal, not an approved rename.
6. `Prisliste åpnet`: only the expandable text/list on the office page; office_id/name/slug and applicable license filter. Emit closed-to-open, not closing; placement unnecessary while this remains the only location.
7. `Klikk: kjøp pakke`: package product_id/name, product_type=package, office and placement. Not completed purchase.
8. `Klikk: reserver kursplass`: product_type=course, office_product_id, course_name, office and course_id only if a concrete occurrence is selected.
9. `Kursdato valgt`: product_type=package|course, choice=selected|later. Include course_id/name/start_at for a concrete selection and office_product_id when available. Package flow also retains package product_id/name.
10. `Klikk: fortsett fra oppsummering`: one shared final action for both summary pages; login_method=vipps|phone distinguishes Fortsett med Vipps from Fortsett med telefonnummer. Do not collect phone numbers or login data.

All relevant actions include route as pathname. Avoid arbitrary full URL/query/input payloads. Add attribution centrally rather than independently to every button.

## Course-flow acceptance

Trace these distinct steps in order:
1. Office-page `Reserver kursplass` → `Klikk: reserver kursplass`.
2. Arrival at `/trafikkskole/<office>/kurs?opId=<office-product>` → `$pageview` with `page_type=course_selection`, whether one or several occurrences are listed.
3. Click a concrete occurrence → `Kursdato valgt` with the selected course identity.
4. Arrival at its summary → `$pageview` with `page_type=summary`.

The office-card `Se alle datoer` expansion is optional and outside this reservation sequence. Do not confuse occurrences within one course group with different course groups (`Se alle kurs`) or days within one occurrence (`Se flere kursdager`). For an expansion test, request at least four occurrences in the same QA office/course group and verify the button is visible at the tested viewport. A production screenshot establishes the control's appearance, not QA data availability. Keep that one case explicitly unverified live until suitable QA data is available.

## Summary-page payloads
- Package without selected occurrence: product_type=package, package product_id/name, office_id/name/slug, login_method. course_selection=later only for an explicit defer choice; not_applicable when course selection does not apply. Omit unknown course IDs.
- Package with selected occurrence: retain product_type=package and package product_id/name; add course_selection=selected, course_id, course_name, course_start_at, applicable office_product_id, office and login_method. A course_id must never overwrite the package product_id or change the flow into standalone course.
- Standalone course: product_type=course, office_product_id, course_id/name/start_at, course_selection=selected, office and login_method; no fabricated package ID.
- Existing outbound parameters are pakke_id plus optional kurs_id for packages, and kurs_id for standalone courses. Preserve these distinct identities in analytics.
- Already authenticated users may redirect without seeing the buttons. Do not fabricate click events for automatic redirects.

## Attribution and rollout preferences
- Exact OpenAI field is `oppref`, not `oopref`. Check the installed SDK's built-in support before adding a duplicate parser. Verify `__oppref` cookie representation using official docs/actual evidence before introducing fallback; URL and cookie support are separate.
- Preserve `utm_id` as an opaque unchanged string; do not impose campaign meaning or numeric coercion.
- Separate analytics permission from marketing identifiers, including SDK-generated properties and initial/current URL/referrer persistence. Test marketing-only withdrawal while analytics remains allowed, including pending retries.
- User accepts temporary double initial QA pageviews caused by shared CookieScript reload. Do not build a staging workaround or suppress legitimate reload views solely to eliminate that QA artifact. Address the general CMP reload as part of a separately approved production rollout with compatible PROD startup; do not switch shared settings early.
- General CMP reload and targeted reload to discard old SDK queues are different decisions. Obtain agreement on additional user-visible reload behavior rather than silently expanding it.
