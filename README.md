# Cold Email Copy Grader

A Claude Code skill that grades your cold email campaigns on a 0-100 scale. Based on patterns from 1,000+ real B2B cold email campaigns.

Paste your draft email sequences and targeting details. Get back a score, risk flags, benchmark comparisons, and full copywriting rewrites when your copy needs work.

No API keys, no database, no external tools. Everything runs from the patterns baked into the skill.

## What You Get

- **Score (0-100)** broken down by copywriting quality, targeting, and personalization
- **Risk flags** that catch the mistakes most likely to tank your campaign
- **Benchmark comparison** against top-performing campaigns
- **Full rewrites** when your score is below 65 — subject lines, body copy, and follow-up sequences
- **Quick grade mode** for a fast gut check (letter grade + 3 bullets)

## The Most Important Finding

Generic AI personalization — where AI describes what the prospect's company does — has a **71% poor rate**. It performs **4x worse than no personalization at all**.

Meanwhile, targeted product-focused AI — where AI calculates your value for them — has a **63.9% top-performer rate**.

The difference:
- Bad: "I noticed {{company_name}} helps customers with [AI-generated description]..."
- Good: "{{company_name}} could save {{savings}} annually by switching to [our approach]..."

This single insight is worth more than most cold email courses.

## Installation

### Claude Code (CLI)

Copy the skill to your Claude Code skills directory:

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills/cold-email-copy-grader

# Copy the skill files
cp SKILL.md ~/.claude/skills/cold-email-copy-grader/
cp -r examples ~/.claude/skills/cold-email-copy-grader/
```

Then use it in Claude Code:

```
/cold-email-copy-grader
```

Or just paste your email copy and say "grade this cold email."

### Manual Use (Any LLM)

You can also use `SKILL.md` as a system prompt or context document with any LLM. The scoring methodology, benchmarks, and rewrite guidelines work independently of Claude Code. Copy the contents of `SKILL.md` into your system prompt and paste your campaign copy.

## Usage

### Full Grade

Provide your email sequences, targeting info, and merge variables:

```
Grade this campaign:

Target: CEOs at restaurants, 1-50 employees, US

Sequence 1:
Subject: quick question about {{company_name}}?
Body: {{first_name}}, saw {{company_name}} has a {{rating}}-star rating on Google...
[rest of email]

Sequence 2:
[follow-up]
```

You'll get a full report with score breakdown, risk flags, benchmarks, and rewrites if needed.

### Quick Grade

For a fast gut check:

```
Quick check on this email:

Subject: Partnership opportunity
Body: Hi {{first_name}}, I wanted to reach out about...
```

Returns a letter grade, 3 bullet points, and one recommendation.

## Example

See [examples/before-and-after.md](examples/before-and-after.md) for a full worked example showing a mediocre campaign (score: 38) getting rewritten to above average (score: 66).

**Before**: Generic subject, passive CTA, bump-only follow-up, 2 sequences
**After**: Question subject, observation opener, value-add follow-ups, 4 sequences with breakup email

## What's Inside

The scoring is based on three weighted components:

| Factor | Weight | What It Measures |
|--------|--------|-----------------|
| Copywriting | 40% | Subject line patterns, body structure, CTAs, sequence flow |
| Targeting | 35% | Decision-maker concentration, industry focus, audience-copy alignment |
| Variables | 25% | Personalization type and quality (the generic AI trap lives here) |

Plus a risk assessment system that catches 15 anti-patterns, from spam triggers to the "AI Personalization Trap."

## Key Benchmarks

| What | Top Performers | Average | Poor |
|------|---------------|---------|------|
| Subject line | Question format, <42 chars | Standard, 47 chars | Generic, 54+ chars |
| Body length | ~73 words | ~71 words | ~70 words (length isn't the issue) |
| Sequences | 4-5 | 3 | 1-2 |
| Opener | Observation (14%) | Question (11%) | Generic greeting (4%) |
| CTA | Question (13%) | Soft ask (11%) | Passive (6%) |
| Follow-ups | Value-add (13%) | Mixed | Bump-only (5%) |

## Where This Data Comes From

These patterns were extracted from analysis of 1,000+ real B2B cold email campaigns across restaurants, healthcare, real estate, professional services, technology, manufacturing, finance, and other industries. The campaigns ranged from excellent (top 10%) to poor, providing clear signal on what separates winners from losers.

The analysis covered subject line patterns, body structure, personalization strategies, sequence design, targeting approaches, and — most importantly — the impact of different types of AI personalization on campaign performance.

## License

MIT License. See [LICENSE](LICENSE).

## Credits

Built by [GrowthEngineX](https://growthengine-x.com). Data from real campaigns run across dozens of industries.
