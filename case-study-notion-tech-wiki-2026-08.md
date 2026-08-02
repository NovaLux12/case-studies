# Case Study: Personal tech wiki build in Notion (2026-08)

**Topic:** Built a structured personal tech wiki in Notion from scratch — 16 pages, 4 sections, expanded from existing notes and corrected against real-world data.

**Outcome:** Live wiki with Projects, Devices, Digital Services, and Household Admin sections plus supporting reference pages. Replaced a flat knowledge base with a navigable hierarchy.

## What happened

Requested a Notion-based tech wiki. Started from zero — no existing Notion workspace structure for this content. Built it in a single session using the `ntn` (Notion) skill for API writes and existing notes as the primary source.

### Structure

Root page **"Tech Wiki"** with 4 top-level sections:

- **Projects** — Active project inventory, tech stack, hosting
- **Devices** — Personal computers, displays, peripherals, ecosystem
- **Digital Services** — Email provider, DNS, aliases, integrations
- **Household Admin** — Council tax, utilities, property details, estate management

Supporting pages:
- Network (router, mesh/VPN, DNS, QoS)
- Technical Reference (scripting languages, web stack)
- Services & Accounts (recurring services, subscriptions, integrations)
- Emergency Guide (medical, household, tech)
- DNS Records (zone details)

### Content sourcing

Primary source: existing personal notes. Secondary: web research for current council tax bands, provider contact details, and recall data.

### Corrections made mid-build

Several speculative entries flagged and removed:
- Wrong council named — corrected to the actual local council.
- Direct debit/standing order lists — not verified against bank data. Removed.
- Mortgage provider, home/contents insurance — unknown. Removed speculative entries.
- "In password manager" claims — no bills or account details stored there. Stripped all unverified references.

## Methodology

1. **Create section parents first**, then child pages with explicit `--parent page:<id>` — avoids Notion's default behaviour of creating database items instead of child pages.
2. **Strip frontmatter before pushing** — Notion does not auto-map `title:` frontmatter to the page title property; it ends up in the body.
3. **Research current data** — council tax bands, recall data, provider contacts change. Pull fresh rather than relying on memory.
4. **Flag unknowns explicitly** — add a note to check the bill rather than guessing. Speculative content erodes trust in the rest of the wiki.

## Lessons

- Notion `ntn pages create` requires explicit `--parent page:<id>` to avoid creating database items in the People database.
- Frontmatter `title:` is not automatically mapped to Notion's page title property — strip before edit or PATCH separately.
- 3 pages had empty titles due to an API issue — fixed by PATCHing the title property directly directly after creation.
- Notion search can return orphaned pages from old builds — manually identify and delete duplicates.
- Speculative content in a knowledge base is worse than gaps. Gaps prompt research; speculation prompts wrong actions.

## Files

- All 16 pages created via Notion MCP (`ntn` skill)
- Source material: existing notes, web research
- No local repo — content lives in Notion workspace
