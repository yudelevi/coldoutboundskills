---
name: disco-like
description: Find lookalike companies via DiscoLike's 65M+ business domain database. Search by seed domains ("find companies like clay.com and apollo.io") or natural-language ICP text ("B2B cold email outreach"). Supports negation domains and country/region filtering. Use when you already know 3-10 reference companies and want hundreds more that look like them. Outputs CSV ready for /blitz-list-builder or the email waterfall.
---

# Disco-Like

Lookalike company discovery. Give it 3-10 seed domains you know are a good fit; it returns hundreds of similar companies by domain, industry, and business characteristics. Useful for expanding from a small known-good list to a much bigger TAM without manual research.

## When to use

- You have 3-10 customer domains you love, want "more like these"
- You want to expand a small client list into a full TAM
- You have an ICP description but don't want to manually build Prospeo filters
- Competitive / adjacent-market expansion

## When NOT to use

- You need PEOPLE, not companies (use Prospeo or Blitz after this)
- Your ICP is extremely narrow or nascent (<5 seed examples exist)
- Budget is tight — DiscoLike charges credits per net-new domain (90-day dedupe); check the dashboard for current rate

## Two search modes

### Mode A — Seed domains (most common)

```bash
npx tsx scripts/discover.ts --domains "clay.com,apollo.io,outreach.io" --country US --limit 500 --out lookalikes.csv
```

DiscoLike finds companies with similar characteristics (industry mix, employee count range, business type, tech stack) to your seeds.

### Mode B — Natural-language ICP

```bash
npx tsx scripts/discover.ts --text "B2B SaaS companies selling outbound sales software to RevOps teams" --country US --out lookalikes.csv
```

`--text` routes to DiscoLike's `icp_prompt` wizard, which extracts filters (country, employee range, category, tech stack), generates clean ICP text, and synthesizes its own seed domain set, then runs discovery — all in one call. With a sharp ICP description this is typically **more precise than 2-3 hand-picked seeds**, because the wizard pulls a wider, better-balanced seed set from the index plus structured filters you'd otherwise have to hand-build. The script logs the `X-Applied-Filters` response header so you can see what got extracted and iterate.

### Hybrid mode

```bash
npx tsx scripts/discover.ts --domains "clay.com" --text "outbound automation" --country US --out lookalikes.csv
```

Combines both — starts from seeds, expands via the ICP wizard.

## Negation (exclude existing customers / competitors)

```bash
npx tsx scripts/discover.ts \
  --domains "clay.com,apollo.io" \
  --negation-domains "yourcompany.com,yourbigcustomer.com" \
  --country US \
  --out lookalikes.csv
```

Always include your own domain + existing customers + known-unfit competitors. Saves enrichment cost downstream.

## Inputs

- `DISCOLIKE_API_KEY` (env) — from DiscoLike dashboard (https://app.discolike.com/account/management/keys)
- Either `--domains` or `--text` (at least one required)
- Optional: `--negation-domains`, `--country`, `--limit`, `--max-companies`
- `--country` accepts ISO-3166-1 alpha-2 codes (US, GB, DE, ...) AND region aliases (`EU`, `EMEA`, `APAC`, `DACH`, `NORDICS`, `LATAM`, `MENA`, `BENELUX`, `CEE`, `ANZ`, `ASEAN`, `GCC`) which auto-expand server-side.

## Outputs

CSV with columns: `domain, company_name, industry, headcount_range, headcount, location_country, location_state, location_city, linkedin_url, similarity, score, description, source`

All rows have `source=discolike` so you can mix with other list-builder outputs without collisions.

## Cost

DiscoLike bills credits **only for net-new domains** — anything your account discovered in the last 90 days re-pulls for free. That makes iteration cheap: re-running the same `--text` or seed list with tweaked filters mostly hits the dedupe cache. The per-credit rate and your monthly allotment depend on plan; see https://app.discolike.com for current pricing. To forecast a fresh pull, run a small `--limit 20` sample and read the `X-Total-Count` header (the script logs it).

## Required step: Qualify with /icp-prompt-builder

**This is a required step. Do not skip it.**

Before pulling 5,000 companies, run DiscoLike on a small sample (50-100), then invoke `/icp-prompt-builder`:
1. Evaluate which of the 50 are actually good ICP fits
2. Refine your ICP description / negation list based on what DiscoLike returned
3. Only then scale to 5,000+

(DiscoLike also has a built-in `/v1/validate/icp` endpoint that does this server-side with structured fit / confidence / reasoning per domain — handy if you want to skip eyeballing.)

**Why required:** DiscoLike lookalike results are only as good as your seed domains. If 80% of the first 50 are wrong, you need to change seeds, not pay to pull more. A wrong-seeded 10K pull burns credits AND cascades into wasted email-finder fees downstream. Qualifying the first 50 catches bad seeds before they become expensive.

**Cheap to iterate.** Because of the 90-day net-new dedupe, re-running the same query with tweaked `--text`, filters, or negations mostly re-hits domains you already paid for — so refining your inputs after the sample costs roughly nothing in credits.

## Recommended flow

1. `/icp-onboarding` → nail down seed companies (your best 5 customers)
2. `/disco-like --domains="seed1,seed2,..." --limit=100 --out=sample.csv` → sample run
3. `/icp-prompt-builder` → score the sample, tune ICP prompt
4. If sample quality is high, scale: `/disco-like ... --max-companies=5000 --out=full.csv`
5. `/blitz-list-builder --domains-file=full.csv` → find decision-makers at each
6. `/email-waterfall` → fill in emails
7. Upload to Smartlead

## API details (reference)

**Base URL:** `https://api.discolike.com/v1`

**Auth:** `x-discolike-key` header

**Endpoints:**

| Method | Path | Purpose |
|---|---|---|
| GET | `/discover` | Lookalike + filtered discovery (paginated via `max_records` + `offset`) |
| GET | `/count` | Pre-count for **structured filters only** (no `domain`, no `icp_text`) |
| GET | `/bizdata?domain=X` | Detailed data for a single domain |

**Data returned per company:**
- `domain`, `name`, `description`, `similarity` (0-100), `score` (digital footprint 0-800)
- `industry_groups` (weighted dict — script takes top industry)
- `employees` (range string like "51-200")
- `address` (country, state, city)
- `social_urls` (script extracts LinkedIn company URL)

**Rate limit:** Conservative — script throttles at 5 concurrent, 10 req/sec. No 429s observed on normal runs.

## Common gotchas

- **Seed domains must be clean bare domains.** `clay.com` works, `https://clay.com/` doesn't.
- **`--text` runs the ICP wizard, not raw semantic match.** The wizard auto-extracts filters and seed domains from your description. If you specifically want raw semantic matching, use `--icp-text` instead (maps to the lower-level `icp_text` parameter).
- **No people data.** DiscoLike is company-level. Always chain with Blitz or Prospeo for contacts.
- **Regional coverage is good.** EU / EMEA / APAC / DACH / NORDICS / LATAM are first-class region codes that auto-expand. Don't manually OR member countries; pass the region.
- **`/count` cannot pre-count a vector query.** It only accepts structured filters (country, category, employee_range, tech_stack, phrase_match, etc.) — passing `domain` or `icp_text` returns 422. To gauge a lookalike's universe, run `/discover` with `--limit 20` and read `X-Total-Count` instead.
- **Default results can be 50 near-clones of one archetype.** If your ICP is broad and your top result dominates the vector space, the default `variance=UNRESTRICTED` will return many similar companies. Pass `--variance MEDIUM` (or `MID_HIGH`) to spread results across the ICP — useful for TAM building where you want range, not 100 versions of the same company.

## Scripts

- `scripts/discover.ts` — main search + CSV output
- `scripts/count.ts` — structured-filter precount (no `domain` / `icp_text`)
- `scripts/bizdata.ts` — single-domain lookup

## What to do next

**Run `/icp-prompt-builder`** on your 50-company sample (required step above). Then either:
- `/blitz-list-builder` to find owner contacts at each filtered domain, OR
- `/list-quality-scorecard` directly if this is companies-only and you'll enrich another way

**Or wait:** if the 50-sample ICP fit was poor (<40% matches), don't scale. Change your seed domains and re-run with better inputs.

## Related skills

- `/icp-onboarding` — defines the seed domains you'll use
- `/icp-prompt-builder` — quality-check the first 50 results before scaling
- `/blitz-list-builder` — chain to find contacts at each discovered company
- `/email-waterfall` — fill missing emails after Blitz
- `/cold-email-starter-kit` → `06-list-building-prospeo.md` for broader list-building patterns
