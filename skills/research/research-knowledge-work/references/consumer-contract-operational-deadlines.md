# Operational deadline terminology in consumer contracts

Use this reference when a product must calculate cancellation, rescheduling, notice, or booking deadlines using an industry term whose everyday or legal meaning varies (for example `virkedag`, `business day`, or `working day`).

## Core lesson

Do not replace an established industry term merely because dictionaries or statutes use it inconsistently. If the industry and users already say `virkedag`, preserve that vocabulary and remove ambiguity through an explicit contractual definition and a concrete calculated deadline.

A strong pattern is:

> I disse avbestillingsvilkårene betyr virkedag mandag til fredag, med unntak av offentlige hellig- og høytidsdager. Lørdag og søndag regnes ikke som virkedager.

Scope the definition to **these terms** rather than claiming a universal legal definition.

## Legal/research discipline

1. Check whether the deadline is contractual or imposed by mandatory legislation.
2. Do not assume a word has one universal legal meaning. Different statutes can define or operationalize the same word differently.
3. A contractual definition generally cannot redefine a mandatory statutory deadline.
4. For consumer terms, assess both clarity and substantive reasonableness:
   - Markedsføringsloven § 22: balance and clarity in consumer contract terms.
   - Avtaleloven § 36: unreasonable terms can be set aside or changed.
   - Avtaleloven § 37: ambiguity in non-negotiated consumer terms is interpreted in the consumer's favour.
5. Treat the cancellation fee and the day-definition as separate legal questions. A clear calendar rule does not by itself make a full-fee clause reasonable.
6. Phrase conclusions as a grounded product/legal assessment, not a formal legal opinion, and recommend one legal review when standard terms will be distributed to many businesses.

Useful primary sources:

- Markedsføringsloven § 22: https://lovdata.no/dokument/NL/lov/2009-01-09-2/KAPITTEL_5#%C2%A722
- Avtaleloven §§ 36–37: https://lovdata.no/dokument/NL/lov/1918-05-31-4/KAPITTEL_3#%C2%A736
- Angrerettloven § 6 (example of context-specific deadline rules): https://lovdata.no/dokument/NL/lov/2014-06-20-27/KAPITTEL_1#%C2%A76
- Bokmålsordboka, `virkedag`: https://ordbokene.no/nob/bm/virkedag
- Bokmålsordboka, `hverdag`: https://ordbokene.no/nob/bm/hverdag
- Bokmålsordboka, `arbeidsdag`: https://ordbokene.no/nob/bm/arbeidsdag

## Product-model pattern

Represent the deadline as structured data, not only free text:

```text
count: 1
unit: CONTRACTUAL_BUSINESS_DAY
included_weekdays: MONDAY–FRIDAY
excluded_calendar: Norwegian public holidays
cutoff_local_time: 12:00
timezone: Europe/Oslo
policy_version: ...
computed_deadline_at: ...
```

A fixed local cutoff such as `kl. 12.00 én virkedag før` is not the same as a rolling `24 hours before` rule.

Calculation:

1. Exclude the appointment date.
2. Walk backward by calendar date.
3. Count only the contractually included days.
4. Skip listed public holidays and explicit school exceptions.
5. Set the cutoff on the resulting date in the business's timezone.
6. Snapshot the policy and computed deadline on the booking so later policy changes do not silently alter old contracts.

## UI and contract wording

Settings can use the established compact term:

```text
Avbestillingsfrist: [1] virkedag før, kl. [12:00]
```

Always pair it with the definition and a generated example.

The customer-facing booking should lead with the concrete result:

> Avbestillingsfrist: fredag 14. august kl. 12.00.

Then show the rule secondarily. This minimizes disputes without forcing the industry to learn replacement terminology.

Keep **when** and **how** separate:

- deadline rule
- accepted cancellation channels
- whether notice must be sent, received, or registered

Prefer `mottatt og registrert` over the vague `må skje` when channel timing matters. Provide a timestamped self-service channel where possible.

## Edge cases to force into requirements

- Saturday and Sunday
- movable public holidays
- appointment immediately after Easter/Christmas
- Christmas Eve and New Year's Eve (not automatically public holidays)
- school-specific closure dates
- appointments held on weekends
- timezone and daylight-saving transitions
- rescheduling
- policy changes after booking
- failed/unsupported cancellation channels
- whether a full fee is mandatory or discretionary

## Pitfalls

- Do not overrule established domain language with a newly coined product term unless the user explicitly wants terminology change.
- Do not state that `virkedag` universally includes or excludes Saturday.
- Do not bury the definition only in long terms; show the exact calculated deadline on each booking.
- Do not tie contractual deadlines directly to mutable teacher schedules or opening hours without versioning.
- Do not infer that widespread industry practice automatically makes every fee clause legally reasonable.
