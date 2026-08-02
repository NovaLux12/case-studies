# Case Study: Notion Tech Wiki build (2026-08-02)

**Topic:** Built a structured personal tech wiki in Notion from scratch — 16 pages, 4 sections, expanded from memory/wiki and corrected against real-world data.

**Outcome:** Live wiki with Projects, Devices, Email & Domains, Household Admin sections plus supporting reference pages. Replaced a flat knowledge base with a navigable hierarchy.

## What happened

Jack asked for a Notion-based tech wiki. Started from zero — no existing Notion workspace structure for this content. Built it in a single session using the `ntn` (Notion) skill for API writes and memory/wiki as the primary source.

### Structure

Root page **"Jack's Tech Wiki"** with 4 top-level sections:

- **Projects** — Active project inventory, tech stack, hosting
- **Devices** — MacBook, displays, peripherals, Apple ecosystem
- **Email & Domains** — Fastmail, Cloudflare DNS, Masked Emails, aliases
- **Household Admin** — Council tax, utilities, car details, estate management

Supporting pages:
- Network (GL.iNet, Tailscale, DNS, QoS)
- Technical Reference (PowerShell, Python, Bash, Go, web stack)
- Services & Accounts (recurring services, subscriptions, integrations)
- Emergency Guide (medical, household, tech)
- DNS Records (Cloudflare zone details)

### Content sourcing

Primary source: existing memory/wiki files. Secondary: web research for current council tax bands, provider contact details, and FSA recall data.

### Corrections made mid-build

Jack flagged several speculative entries:
- "TMBC" listed as council — actually his employer. Maidstone Borough Council (MBC) is correct.
- Direct debit/standing order lists — not verified against bank data. Removed.
- Mortgage provider, home/contents insurance — unknown. Removed speculative entries.
- "In 1Password" claims — Jack has no bills or account details in 1Password. Stripped all unverified references.

## Methodology

1. **Create section parents first**, then child pages with explicit `--parent page:<id>` — avoids Notion's default behaviour of creating database items instead of child pages.
2. **Strip frontmatter before pushing** — Notion does not auto-map `title:` frontmatter to the page title property; it ends up in the body.
3. **Research current data** — council tax bands, FSA recalls, provider contacts change. Pull fresh rather than relying on memory.
4. **Flag unknowns explicitly** — add a note to check the bill rather than guessing. Speculative content erodes trust in the rest of the wiki.

## Lessons

- Notion `ntn pages create` requires explicit `--parent page:<id>` to avoid creating database items in the People database.
- Frontmatter `title:` is not automatically mapped to Notion's page title property — strip before edit or PATCH separately.
- 3 pages had empty titles due to an API issue — fixed by PATCHing the title property directly directly after creation.
- Notion search can return orphaned pages from old builds — manually identify and delete duplicates.
- Speculative content in a knowledge base is worse than gaps. Gaps prompt research; speculation prompts wrong actions.

## Files

- All 16 pages created via Notion MCP (`ntn` skill)
- Source material: `memory/wiki/`, `knowledge/entities/`, web research
- No local repo — content lives in Notion workspace
