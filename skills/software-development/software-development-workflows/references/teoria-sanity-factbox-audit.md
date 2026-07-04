# Teoria Sanity content/fact-box audit notes

Use for Teoria CMS/front-end investigations involving fact boxes, published vs draft content, and attribution.

## Access/context

- Teoria frontend repo observed at `~/repos/teoria-frontend`.
- Teoria frontend Sanity project ID: `450l1d3m`.
- Dataset used for production content: `production`.
- Local token path observed: `~/clawd-shared/credentials/sanity_token_teoria` (do not print token contents).
- Wright-Web is a different Sanity/org context; do not assume Wright-Web CMS controls Teoria pages.

## Frontend data flow

In `app/api/sanity.server.ts`, Teoria creates a Sanity client roughly as:

```ts
createClient({
  projectId: "450l1d3m",
  dataset: process.env.SANITY_DATASET ?? "development",
  apiVersion: process.env.SANITY_API_VERSION ?? "v2025-06-24",
  token: process.env.SANITY_API_TOKEN,
  useCdn: true,
  perspective: "published",
})
```

Class landing pages query:

```groq
*[_type == "undersideKlasse" && driverClass == $klasse][0]
```

Fact boxes are rendered directly from manual CMS fields:

```ts
faktaboks.fakta.map((f) => ({
  key: getLocalizedField(f.key, locale),
  value: getLocalizedField(f.value, locale),
  description: getLocalizedField(f.description, locale),
}))
```

So if Sanity contains wrong facts, the site displays wrong facts; there is no frontend validation against Statens vegvesen.

## Query patterns

### Published and draft documents

```bash
SANITY_AUTH_TOKEN=[REDACTED]
PROJECT=450l1d3m
DATASET=production
```

```groq
*[_id in ["<docId>", "drafts.<docId>"]]{
  _id,_type,_updatedAt,_rev,driverClass,title,
  faktaboks{title,fakta[]{_key,key,value,description}}
} | order(_id asc)
```

Use `perspective=raw` to see both published and draft; `perspective=published` to see only what production frontend should read.

### History API

Sanity History API returns newline-delimited JSON, not a single JSON array. Parse line-by-line:

```python
raw = urllib.request.urlopen(req).read().decode()
for line in raw.splitlines():
    if line.strip():
        tx = json.loads(line)
```

Then map `author` IDs with:

```bash
SANITY_AUTH_TOKEN=[REDACTED]
```

## Fact-box pitfalls

- Sanity Studio may show red validation warnings on each fact row even when the numeric facts are correct. One observed cause: `description` had only Norwegian (`no`) values like `-`, while other localized fields had both `no` and `en`.
- The live website may still show the published document even if Studio is currently editing a draft; always compare published vs draft before diagnosing.
- Do not conflate validation errors with factual errors. Check both schema/localization completeness and authoritative sources.
- Treat pass/fail percentages as marketing/statistical claims, not fixed rule facts. For Teoria messaging use “40 % stryker”, not the 44 % figure from the communications-platform PDF.

## Klasse B observed values

For `undersideKlasse` driverClass `B`, document `6a24f81d-441b-4d61-a613-a0fa7c6213bb`, observed published fact-box values were:

- Antall spørsmål: `45`
- Riktige svar: `38`
- Maks feil: `Maks 7 feil`
- Tid: `90 minutter`
- Tidligste alder: `17,5 år`
- Pris teoriprøve: `480 kr`
- Består: `Ca. 60 %` (marketing/statistical approximation)
- Karantenetid ved stryk: `2 uker`

Statens vegvesen pages are the preferred source for current rule/fee facts. Re-check live sources before editing/publishing, because fees and rules can change.

## Recommended durable model

Prefer a central Teoria fact source per driver class instead of duplicating manual facts across class/gratis/FAQ pages:

```txt
driverClass: B
numberOfQuestions: 45
requiredCorrect: 38
maxErrors: 7
testDurationMinutes: 90
earliestAgeYears: 17.5
testFeeNok: 480
waitingPeriodAfterFailDays: 14
failRateApproxPercent: 40
sourceUrl: Statens vegvesen
sourceCheckedAt: YYYY-MM-DD
```
