# RwLock Self-Deadlock Under Token-Refresh — Case Study

_Date: 2026-08-09_
_Context: a Tauri desktop app that refreshes OAuth tokens on a polling timer_

## What happened

A pre-existing concurrency bug in a token-refresh helper caused a self-deadlock under refresh load. The helper refreshed OAuth tokens, then persisted the refreshed tokens to disk. Persistence took a **read lock** on the same `RwLock` that the caller was holding a **write guard** on. `parking_lot` does not allow a thread to acquire a read lock while it already holds a write lock on the same `RwLock` — the thread parks forever. The app hung silently on the first proactive refresh after launch.

The bug was not in the CAS logic itself. It was in the *lifetime* of the write guard relative to the persistence call.

## Root cause

Callers passed `&mut *state.tokens.X_mut()` — a reborrow of a `parking_lot::RwLockWriteGuard`. The temporary lives until the end of the call statement, which means the write guard is still held when the persistence function tries to take its own read lock inside the helper's success path.

```
caller: write guard held
  → helper: persist() takes read lock
    → SAME THREAD already holds write guard
      → deadlock
```

This only manifested under proactive refresh (the polling timer path), not during the initial OAuth exchange, which is why it survived earlier testing.

## Fix

Move persistence **out of the helper** and into the callers, in a statement that runs *after* the CAS call returns — when the write guard is provably dropped.

- The helper went from 7 args to 5; its body no longer calls the persist function.
- All three call sites persist on `Committed` in a separate statement.
- Added two regression tests:
  - a runtime same-lock persist test with a 10s `recv_timeout` (catches deadlocks)
  - two compile-time guards asserting the helper body contains no persist call and that persistence only occurs at the expected call sites

## Why it matters

This class of bug — write-to-read on the same lock, same thread — is invisible in code review unless you're tracing guard lifetimes explicitly. The fix is small, but the diagnostic step required understanding the app's polling architecture, the token-storage module's locking strategy, and the exact drop semantics of `RwLockWriteGuard` reborrows.

The regression tests are the real value. The runtime test catches future refactors that accidentally reintroduce the pattern; the compile-time guards make it a build error to add a persist call inside the helper.

## Lessons

- **Guard lifetimes are API contracts.** When a helper is documented as "lock-focused," callers reasonably assume the lock is held for the duration. Mutating that assumption by adding side effects inside the helper creates a hidden coupling.
- **Proactive paths need separate test coverage.** The initial OAuth flow was tested; the timer-driven refresh path was not. Bugs that live in "later" paths survive longer.
- **parking_lot's write→read deadlock is silent.** There's no panic, no timeout, no log — the thread just parks. A hung app with no frontend error is a hard diagnostic.