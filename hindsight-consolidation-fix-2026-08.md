# Hindsight Consolidation Fix — 76× Throughput Recovery

_Date: 2026-08-12_
_System: OpenClaw hindsight memory layer_
_Issue: internal performance bottleneck_

## What happened

Hindsight consolidation was running at batch 48 with a 120s timeout. On a backlog of ~464 memories, it was timing out repeatedly — effectively stuck. The consolidated throughput was too slow to clear the queue in any reasonable window.

## Root cause

Two levers were misconfigured for the workload:

1. **Batch size too large.** At 48 memories per LLM call, each consolidation request hit token limits and took long enough to approach the timeout ceiling. Large batches also amplified variance — one slow response meant the whole batch failed.
2. **Timeout too short for the batch size.** At 120s, the timeout was tighter than the actual distribution of LLM response times for large batches. The system retried, but retries on timeouts just added load without making progress.

A third factor was that reasoning/thinking was left enabled by default. For consolidation — a relatively mechanical summarization task — the extra reasoning tokens increased cost and latency without commensurate quality gain.

## Fix

Three coordinated changes:

- **Batch 48 → 12.** Smaller batches fit comfortably within the LLM’s working context, reduced per-request variance, and let the system parallelize better across the queue.
- **Timeout 120 → 300.** With smaller batches, the actual response time distribution dropped well below 300s, giving enough headroom to absorb slow runs without retry storms.
- **Disable thinking/reasoning.** Set `LLM_EXTRA_BODY` reasoning off for consolidation runs. The task benefits more from throughput than from chain-of-thought depth.

## Result

- **76× faster consolidation** — from roughly 1 minute per memory down to ~1s/memory.
- **464-backlog cleared in ~6 minutes** with zero timeouts.
- Both previously stuck operations completed automatically.

## Lessons

- **Batch size and timeout are coupled.** Changing one without the other is incomplete. If you shrink batches, you can usually shrink timeout too; if you grow batches, you must grow timeout proportionally.
- **Reasoning defaults are not free.** OpenClaw’s gateway defaults reasoning ON for many providers. For high-volume, low-creativity tasks (consolidation, indexing, tagging), turn it off explicitly.
- **Watch for stuck-queue symptoms.** A backlog that isn’t shrinking, combined with timeout errors in logs, is a signal to inspect batch/timeout/reasoning before adding retries or throwing compute at it.
- **Retry storms mask configuration problems.** Adding retries to a misconfigured pipeline just generates more failing work. Fix the config first; retries are for transient faults, not systematic slowness.

## Source

Verified live on 2026-08-12. Bank PATCH applied to live config; both stuck ops completed; 464-backlog cleared in ~6 min, zero timeouts.
