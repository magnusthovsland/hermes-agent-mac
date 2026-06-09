User's Telegram identity for Hermes gateway allowlisting: username sungam81, numeric user ID 7575580066. User set up a new Hermes Telegram bot named Edith / Edithsungam_bot and does not want Hermes to reuse OpenClaw Telegram credentials.
§
Infinity Drift develops digital products for the traffic-school/training market. Main products: Ovio and Teoria. Relevant tools/platforms include GitHub, AWS, Vercel, Vipps/MobilePay, Link Mobility, SendGrid, Datadog, PostHog, OpenSearch, Airtable, Sanity/CMS, Notion. Hermes Notion files: `~/.hermes/notion/infinity-drift/`; token path: `~/.hermes/credentials/notion_token`.
§
Ovio is Infinity Drift's traffic-school SaaS OS covering schools, pupils/guardians, companies, teachers, vehicles/resources, scheduling/booking, follow-up, payments/finance/reporting, and integrations such as Statens vegvesen, NAF/ENAF, Vipps/MobilePay, Visma, and Kravia.ai.
§
Ovio production backend runs in AWS ECS/Fargate eu-north-1, cluster `wright`, service `wis-main-api`. For backend hang/downtime, first operational action is ECS Force new deployment, not RDS restart. Important integrations: Statens vegvesen, NAF/ENAF, Vipps/MobilePay, Visma.net, Visma Business GraphQL, and Kravia.ai.
§
Teoria is Infinity Drift's theory-learning platform using Airtable + Sanity/CMS/Vercel, with simple wording, plausible alternatives, and consistent question structure. For NO→EN use British/international English for Norway theory-test learners; preserve Norwegian law/actors; terms include give way, driving licence, road user, pedestrian crossing, overtaking; convert promille to % BAC.
§
Hermes Google Drive default folder for Magnus deliverables is `Dokumenter Hermes`, folder ID `1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`; local pointer stored at `~/.hermes/google_drive_hermes_folder.json`. Current reused OpenClaw Google token has Drive scopes only, not Docs/Sheets native edit scopes.