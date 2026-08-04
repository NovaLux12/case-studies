# Case Study: The hindsight daemon that had two owners (2026-08)

**Topic:** A daemon on port 9077 crashed into an endless restart loop because it had two owners — a plugin that auto-spawned its own instance on every gateway start, and a systemd unit that could never bind but kept being restarted. The first diagnosis was wrong; the fix only looked like it worked.

**Outcome:** The redundant systemd unit was retired, the plugin became the single owner, and the daemon stabilised. The real lesson is about verifying *who actually owns a resource* before debugging the copy you think you control.

## What happened

A personal analytics daemon came up as an infinite restart loop: `[Errno 98] address already in use`, `status=3/NOTIMPLEMENTED`, endless `Restart=always` retries on the same port.

### First diagnosis (wrong)

The restart error pointed at the port being occupied. The first theory was a stale manually-started instance holding it. The stale processes were killed. The port freed briefly — and the loop came back within about a minute.

The tell was that the loop *recovered on its own*: killing the wrong owner only bought ~60 seconds before the port was occupied again and the unit resumed failing. That should have registered as "something is respawning the occupant", not "the occupant is gone".

### Second diagnosis (correct)

There were **two independent owners of port 9077**:

1. The host framework's plugin **auto-spawned its own daemon on every gateway start**. This instance actually owned the port, was healthy, and was the one doing the work.
2. A **separate systemd unit** I had been managing. It could never bind — the plugin already owned the port — so it failed instantly and entered the endless `Restart=always` loop that surfaced as the incident.

The unit I was feverishly restarting was a competing *second* instance, not the daemon that needed fixing. The earlier kill of the "stale" process only worked briefly because a later gateway restart caused the plugin to respawn its instance, reoccupying the port.

## Fix

1. Retire the redundant systemd unit (`systemctl --user disable --now`). No second instance, no competing bind, no loop.
2. Confirm the plugin is the single owner on the port.
3. Also correct the plugin's model config, which had been pointed at the wrong provider during the flailing — repointed it to the intended one (verified via the running daemon's live environment, not the config file).

## Lessons

- **Find who actually owns the resource before debugging the copy you manage.** A systemd unit's existence does not mean it is the process in use. If another layer auto-spawns its own instance, your unit may be a stranger you never needed.
- **A restart loop that self-recovers is a signal, not a contradiction.** The loop wasn't flapping randomly — killing the wrong owner briefly freed the port, then the *real* owner respawned. That recovery-on-its-own pattern points at a second, self-healing owner.
- **Ownership, not mere occupation, is the question.** Two processes touching the same port is a bind conflict; the question is which one is the intended service.
- **Verify the running process's environment, not the config file.** The model config had drifted for the actual daemon; the config file and the live process disagreed. The live process was the source of truth.

## Files / artefacts

- systemd unit retired: `hindsight-daemon.service` (disabled + inactive)
- Plugin config repointed to intended provider; verified via running daemon env

Internal operational notes in the operator's daily log (not republished).
