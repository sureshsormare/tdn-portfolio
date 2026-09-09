# Data model

The Prisma schema (`prisma/schema.prisma`, PostgreSQL) has three groups of models.

## Shipment data

| Model | Purpose |
|---|---|
| `exp_india` | One row per export shipment: product, HS code, exporter, buyer, destination, port, quantity, unit, value, date |
| `imp_india` | One row per import shipment with the mirror fields |
| `table_periods` | The date coverage of each loaded table; drives the "latest period" defaults |
| `uploaded_data_files` | Audit of every loaded file |

Loaded by `scripts/load-csvs.mjs` (COPY, per-year files, `--year`, `--kind exp|imp`, `--truncate`).

## Reference and harmonisation

| Model | Purpose |
|---|---|
| `companies`, `company_aliases`, `quarantine_companies` | Canonical companies, their spelling variants, and names held back until reviewed |
| `countries`, `country_aliases` | Canonical countries and variants |
| `drugs`, `drug_dosage_combos`, `drug_shipment_counts`, `therapeutic_class_aliases` | Product dictionary, strength/form combinations, per-drug shipment counts, class aliases |
| `harmonization_progress` | Tracks how far each dimension has been harmonised |

## Accounts and plans

| Model | Purpose |
|---|---|
| `User`, `Account`, `Session`, `VerificationToken`, `Authenticator` | NextAuth models |
| `UserProfile`, `UserCredits` | Profile and credit balance |
| `SubscriptionType`, `SubscriptionPlan`, `SubscriptionStatus` | Plan catalogue and state |

## Query patterns

- **Search / autocomplete:** `src/lib/autocomplete.ts` and `public/autocomplete.json` serve prefix matches client-side; `api/search` resolves to canonical ids.
- **Profiles:** `api/profile/company/[slug]` and `country/[slug]` aggregate shipments by partner, product and period; `lib/profile/` shapes the response; credits are debited in `lib/profileCredits.ts`.
- **Trade flow:** `api/trade-flow/facets|flow|options|other` implement faceted filtering with server-side aggregation; the UI uses TanStack Table and virtualised rows.
- **Comtrade:** `api/comtrade/facets|footprint|hs-reference|insights|search` run against the separate Comtrade database (`COMTRADE_DATABASE_URL`).
- **Counters:** `api/stats/counters` and `lib/statsCounters.ts` cache headline totals.

Indexes for the hot paths are in `scripts/create-indexes.sql`; the Comtrade pipeline has its own in `scripts/comtrade/03_index.sql`.
