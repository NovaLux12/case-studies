# Case Study: Provider split under plan pressure (2026-08)

**Topic:** When a coding-plan provider hit 2% remaining, the migration off it wasn't a single swap — it was a deliberate split of consumers across two providers based on traffic shape and prompt-cache keying.

**Outcome:** Default model + five core cron jobs moved to one provider; the highest-concurrency daemon moved to a second provider at double its concurrency. All consumers verified live post-migration. The split is the lesson, not the swap.

## What happened

A primary LLM provider's coding plan drained to 2% remaining, forcing a migration of the default model and cron workloads off it. The reflex is to move everything to one replacement provider. The decision here was to segment instead.

### The split

- **Default model + 5 core crons → Provider A (cheap, steady).** The daily-memory, self-improving, email-indexer, morning-briefing, and item-monitoring jobs all moved to a budget provider running the default model.
- **Hindsight daemon → Provider B (headroom), concurrency 2 → 4.** The memory-retain/consolidation daemon moved to a different provider and had its concurrency doubled.

### Why not just one provider

Two independent reasons:

1. **Prompt-cache keys are per-endpoint.** The model's prompt cache (DeepSeek-style prefix caching) is keyed per API endpoint. The daemon's retain/consolidation prompts do not share prefixes with the cron briefing prompts — so forcing them onto the same provider would buy zero cache hits. The cross-provider "loss" wasn't a loss; there was no shared prefix to lose.

2. **Traffic shape differs.** The daemon is a bursty, high-concurrency consumer. It went to the provider with no platform concurrency cap (headroom). The crons and chat are cheap steady traffic — they went to the budget provider. Crucially, the budget provider (Provider A) is the one with the *hard account-level concurrency ceiling* that had caused 429s before; putting the bursty daemon there was the mistake the split avoided.

### What stayed put (deliberately)

The exhausted provider kept its small, non-plan-drawing workloads: provider definitions, model alias decoders, a TTS voice pipeline (a different product, not the coding plan), and a plugin enable-list. No active LLM consumer on the drained plan remained.

## Verification

- Gateway serving the new default, requests returning 200
- Both agent primaries set to the new default
- Zero cron overrides left on the old provider
- Daemon connection verified on the second provider at concurrency 4, no 429/401

## Lessons

- **When a plan drains, segment consumers — don't consolidate onto one provider.** Match each consumer's traffic shape to provider headroom. Bursty high-concurrency work belongs on the provider with no platform cap; cheap steady work belongs on the budget provider.
- **Prompt caches key per-endpoint.** Forcing consumers onto one provider only helps if their prompts actually share prefixes. If they don't, multi-provider is free.
- **Know which provider has the hard ceiling.** The account-level concurrency cap lives somewhere specific — usually the budget provider. Putting the bursty consumer there repeats the 429 history. The headroom provider is the one without a platform cap on paid models.
- **A drained plan isn't all-or-nothing.** Small utility workloads on a different price axis (TTS, provider blocks) can stay; only the LLM consumers drawing on the depleted plan need to move.

## Files / artefacts

- OpenClaw config: model + per-agent overrides, five cron payload model overrides
- Daemon: provider endpoint, API key env target, concurrency 2 → 4
