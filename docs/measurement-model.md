# Measurement model

What the platform measures, how prompts are issued, how mentions are scored, and where the limits are.

## Units

- **Organisation** with one or more **brands** (name + website) and a list of **competitors** (name + website).
- **Market**: a topic with a set of **prompts** (questions a buyer would ask an AI assistant).
- **Run**: one prompt sent to one engine at one time, producing a stored response and its parsed result.

## Issuing prompts

`visibility-tracker.queryAllEngines` sends the same prompt to each configured engine through a per-engine adapter (`openai.ts`, `anthropic.ts`, `perplexity.ts`, `gemini.ts`). Adapters normalise the answer into `AIEngineResponse` (text, citations if the engine returns them, timing). Model names are configurable per engine (`OPENAI_MODEL`, `ANTHROPIC_MODEL`, `PERPLEXITY_MODEL`, `GEMINI_MODEL`). Failures are caught by `safeQuery` and stored as placeholder responses so the run stays complete.

## Parsing a response

`parser.ts` applies the same rules to every engine:

| Signal | Rule |
|---|---|
| Brand mentioned | case-insensitive match of the brand name or its domain in the response (`parseResponseForBrand`) |
| Position | index of the first mention relative to response length |
| Citations | URLs in the response, plus bare domains without a full URL (`parseResponseForCitations`); own-domain citations flagged separately (`checkIfOwnBrand`) |
| Competitors mentioned | each competitor name or domain checked the same way (`parseResponseForCompetitors`) |
| Sentiment | rule-based positive/neutral/negative from the words around the mention (`analyzeSentiment`) |

## Scoring

`calculateVisibilityScores` aggregates parsed results per brand and engine: mention rate across prompts, average position, citation rate, sentiment mix, and share of voice against competitors. `generateVisibilityReport` produces the report the dashboard renders (`/dashboard/visibility`, `/dashboard/battle`) and the PDF export. Scores are stored per run so trends can be drawn over time.

## Discovery

1. `keyword-generator.ts` proposes seed keywords for a market.
2. `google-suggestions.ts` expands them with Google autocomplete, question variations and "people also ask".
3. `intelligent-query-generator.ts` / `query-intelligence.ts` turn keywords into natural prompts of different intents.
4. Prompts are run across engines; `brand-extractor.ts` pulls every brand and URL from the answers with sentiment context.
5. `competitor-matcher.ts` reconciles extracted names with known competitors and surfaces new ones; results feed the leaderboard, citations and gap views.

## Bot tracking

A tracking script served from the app reports hits to `/api/t`; `bot-detector.ts` identifies crawlers by user agent and IP range (`detectBotByIP`), and `bot-tracker.ts` buffers visits and writes daily files, from which summaries per site, per bot and per day are computed.

## Limits

- Engines are non-deterministic; a single run is a sample, and trends need repeated scheduled runs.
- Sentiment is rule-based and indicative, not a trained classifier.
- Citations depend on what each engine exposes; engines without citation output are scored on mentions only.
- Discovery quality depends on the seed keywords and on autocomplete availability for the locale.
