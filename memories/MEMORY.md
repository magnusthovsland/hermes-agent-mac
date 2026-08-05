User's Telegram identity for Hermes gateway allowlisting: username sungam81, numeric user ID 7575580066. User set up a new Hermes Telegram bot named Edith / Edithsungam_bot and does not want Hermes to reuse OpenClaw Telegram credentials.
§
Ovio is Infinity Drift's traffic-school SaaS. Analytics requirement: each tenant owns its GA4 purchase tracking; preserve ad attribution from the school's domain into Ovio and avoid Wright-hardcoded tags. Magnus uses «virkedag» for cancellation deadlines, defined Monday–Friday (Saturday excluded).
§
Ovio production backend: AWS ECS/Fargate eu-north-1, cluster `wright`, service `wis-main-api`. For hangs/downtime, first action is ECS Force new deployment, not RDS restart.
§
Teoria is Infinity Drift's theory-learning platform using Airtable + Sanity/CMS/Vercel. Teoria PostHog: EU project `29869` (`Teoria - Prod`), credential at `~/.hermes/credentials/posthog_teoria.json` (chmod 600; never expose keys). Style: simple wording, plausible alternatives, consistent structure; NO→EN uses British/international English, preserves Norwegian actors, converts promille to % BAC. Teoria ads should say “40% stryker”, not 44%.
§
Hermes Google Drive default folder for Magnus deliverables is `Dokumenter Hermes`, folder ID `1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`; local pointer stored at `~/.hermes/google_drive_hermes_folder.json`. Current reused OpenClaw Google token has Drive scopes only, not Docs/Sheets native edit scopes.
§
For Wright-Web release workflow, Magnus may ask for fixes to be pushed directly to QA for testing while creating a separate PR against prod/main for developer review and production promotion.
§
Infinity Drift develops traffic-school products Ovio and Teoria using GitHub, AWS, Vercel, PostHog, Airtable, Sanity and Notion. Notion files: `~/.hermes/notion/infinity-drift/`; token: `~/.hermes/credentials/notion_token`. Notion area `Produkt og utvikling → Nettsider` is for website maintenance; its task board should stay simple and standalone, without Ovio/other database relations or integrations.