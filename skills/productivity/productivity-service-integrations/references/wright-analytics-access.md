# Wright analytics access

## Stable project references
- GitHub: `WrightTrafikkskole/Wright-Web`.
- GTM QA: `GTM-MDCP8Q8` on `qa.wright.no`.
- GTM PROD: `GTM-NTMPCTS2` on `wright.no`.
- Google account designated for GTM work: `magnus@infinitydrift.no`. Verify current container visibility; login and permission state can change.
- PostHog EU: project `49059` is `Wright.no PROD`; `50123` is `Wright.no QA`.
- Wright PostHog credentials: `~/.hermes/credentials/posthog_wright.json`. Never print contents. A project-scoped personal key may reject project-list endpoints; use key metadata and exact project endpoints.

## Native Chrome access
- browser_exec may use a separate/headless session invisible to the user. Do not tell the user to log into a visible window unless an actual native browser window was opened.
- Use native Chrome AppleScript for task-scoped DOM reads when desktop capture fails. Rediscover tab/window IDs each time a reference becomes stale; never inspect unrelated personal tabs.
- Chrome requires the user to enable **Vis → Utvikler → Tillat JavaScript fra Apple Events** for AppleScript DOM access. Ask rather than silently changing browser settings; the user can disable it after work.
- Google Ads authorization does not imply GTM OAuth scopes or container access. Verify both authentication and exact container permissions.
- Temporary tokens are not durable access guarantees. Never store token values in skills or memory, and recommend revocation after the task.

## CookieScript variable regression test
- Test missing cookie, rejection, JSON-string categories, array categories, absent categories, malformed JSON and partial withdrawal.
- In GTM sandbox JSON.parse, invalid input returns undefined. Validate parsed objects before reading `.action` and parsed arrays before `.join`.
- The observed variable starts with an empty-string fallback, so `.includes()` is safe when the cookie is absent. The real format hazard is unconditionally JSON-parsing `categories` even when already an array.
- Diagnose the **CookieScript Consent State variable template** separately from the **CookieScript CMP tag template**.
