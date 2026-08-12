# Provider Split — Umans → OpenCode Go

_Date: 2026-08-12_
_System: OpenClaw LLM provider configuration_
_Context: model provider audit + key migration_

## What happened

During a routine provider audit, the active LLM stack had drifted: Umans was still the default utility model and the Hindsight daemon LLM, despite OpenCode Go having been added as a faster, cheaper alternative with better DeepSeek V4 Flash availability. The split was also masking a stale-key bug — the gateway was reading `OPENCODE_API_KEY` from `gateway.systemd.env` instead of the 1Password exec-provider, so a key rotation on 2026-08-11 silently didn't apply.

## What we did

1. **Migrated utility model:** `utilityModel` → `opencode-go/deepseek-v4-flash` (main + tmbc-work), with `openrouter/auto-beta` and `openrouter/free` as fallbacks.
2. **Migrated Hindsight LLM:** Moved from Umans `deepseek-v4-flash-0731` to `opencode-go/deepseek-v4-flash` via an exec SecretRef key. Verified live with a retain call at 08:45 on 2026-08-12.
3. **Removed stale env override:** Deleted `OPENCODE_API_KEY` from `gateway.systemd.env` so the 1Password exec-provider is the single source of truth.
4. **Cleaned up aliases:** Tidied provider aliases in config (e.g., `DeepSeek V4 Flash` → canonical form).
5. **Raised maxTokens:** Overrode StepFun plugin's hardcoded `maxTokens: 65536` for models that need more output headroom.

## Why it matters

- **Env files override config silently.** The gateway kept using the stale OpenCode key for hours after rotation because `gateway.systemd.env` takes precedence over the config's `apiKey`. This is the kind of bug that shows up as "the new key doesn't work" when the real problem is the old key is still being used.
- **Provider aliases drift.** Without a canonical alias list, the same model appears under multiple names across agents, making cost tracking and fallback reasoning harder.
- **Hindsight consolidation is rate-limited by batch + timeout, not model choice.** The consolidation fix (batch 48→12, timeout 120→300, thinking off) was the real unlock; the model migration made it cheaper at the new throughput.

## Lessons

- **Env files are configuration too.** When rotating any API key, grep all env sources (`gateway.systemd.env`, shell rc, systemd unit files) before declaring the rotation complete.
- **Verify live, don't trust config.** A config patch that looks correct can be silently overridden by an env var. Always do a probe call after rotation.
- **One provider, one key source.** If a key can come from 1Password, env, or config, the system will eventually read from the wrong one. Pick one and delete the others.

## Source

Verified live on 2026-08-12. OpenCode Go renewed; Umans now unused by Hindsight + utility model.
