+++
authors = ["Yadullah Duman"]
title = "My Debugging Workflow with Agents"
date = "2026-03-16"
description = "Three levels of solving bugs with the help of coding agents like Claude Code"
categories = ["Agentic Coding"]
tags = ["Agentic Coding", "Claude Code", "Debugging", "AI"]
+++

{{< notice note >}}
This post is fully written by me and was checked for typos and grammar errors with Claude.
{{< /notice >}}

Debugging is where agents shine a lot. I [briefly touched on this](/posts/six-months-of-agentic-coding/#fixing-bugs-faster) in my previous post and I think this topic deserves its own deep dive, because the workflow is more nuanced than just "paste the error and let the agent fix it". Over months of real-world usage, I developed three levels of debugging that I apply based on how nasty the bug is.

Most bugs are simple. You paste the error, add some context, and the agent nails it. But some bugs fight back. For those, you need a system. Here are my three levels.

# Level 1: Context-Rich Debugging

This one is easy. A bug shows up, you have a clear error message or can reproduce it, and all you need to do is give the agent enough context to work with. No fancy workflow needed. Just good input.

### Useful context to provide

- Bug description: what is happening versus what should happen
- Error messages: the full output including stack traces. Do not summarize or truncate. The details matter.
- Logs: relevant log output leading up to the error. For backend issues, server logs are often the fastest path to resolution.
- Screenshots: for frontend or UI bugs, a screenshot showing the problem provides immediate visual context
- Reproduction steps: how to trigger the bug
- Relevant code paths: if you know which files or functions are involved, point the agent there

The key here is: _more context is almost always better_.

### Example

Say your Node.js service crashes on startup after a dependency update. You would give the agent something like:

> After upgrading `pg` from 8.11 to 8.13, the service fails to start. Here is the full error:
>
> ```
> Error: connect ECONNREFUSED 127.0.0.1:5432
>     at TCPConnectWrap.afterConnect [as oncomplete] (node:net:1595:16)
>     at Pool._connect (/app/node_modules/pg-pool/index.js:45:11)
> ```
>
> The database is running and I can connect with `psql`. Other services on the same host work fine. The only change was the `pg` version bump. Here is our connection config in `src/db/pool.ts`.

With that level of context, the agent can trace the issue. Maybe the new `pg` version changed its default SSL behavior, maybe there is a breaking change in connection pooling. The agent reads the config, checks the changelog, and proposes the fix. Done in minutes.

### When to use level 1

- Bugs with clear error messages
- Issues with obvious reproduction steps
- Single-cause bugs without complex interactions
- Anything where you have a rough idea of where the problem lives

This covers 80 to 90 percent of all bugs I encounter. Most of the time, the combination of a good error message, logs, and pointing the agent to the right area of the codebase is enough.

# Level 2: Iterative Fix Tracking

Sometimes level 1 is not enough. The agent proposes a fix, you test it, and it does not work. Maybe the error changes, maybe the symptoms shift slightly, but the bug is still there. This is where most people start going in circles. The agent retries the same approach, or worse, undoes its previous fix and tries something contradictory.

The solution is to create a tracking file that you constantly feed to the agent.

### The fixing loop

1. Create a file like `DEBUG.md` and ask the agent to document the bug
2. Let the agent propose a fix and implement it
3. Test it manually and report the results back
4. The agent updates the tracking file with what was tried and what happened
5. Repeat until the bug is found

The tracking file acts as shared memory between you and the agent. It prevents the agent from retrying failed approaches and, more importantly, each failed attempt provides new information that narrows down the resolution space.

### Example tracking file

```markdown
# Debug: Payment webhook returns 400

## Symptoms
- Stripe webhook endpoint returns 400 for `invoice.paid` events
- Other event types (`checkout.session.completed`) work fine
- Started after deploying v2.4.1
- Logs show "Invalid signature" but the webhook secret is correct

## Attempt 1
**Hypothesis**: Webhook secret was rotated and env var is stale
**Change**: Verified secret matches Stripe dashboard, redeployed with fresh env
**Result**: Still failing. Secret is correct.

## Attempt 2
**Hypothesis**: Request body parsing middleware strips raw body needed for signature verification
**Change**: Added raw body preservation middleware before JSON parser
**Result**: Still failing. But now the signature error is gone — new error is "Missing required field: subscription_id"

## Attempt 3
**Hypothesis**: Stripe changed the payload shape for `invoice.paid` in their latest API version
**Change**: Updated handler to use `subscription` instead of `subscription_id`, matching Stripe API 2024-12-18
**Result**: Fixed. Root cause was Stripe API version mismatch after their November update.
```

Notice how each attempt reveals new information. Attempt 2 did not fix the bug, but it eliminated the signature issue and exposed the real problem underneath. Without tracking, the agent might have kept circling around the signature issue.

I described in my [previous post](/posts/six-months-of-agentic-coding/#fixing-bugs-faster) a complicated frontend bug where I had to resort to this approach. The whole process took at most an hour, including implementing the tests for the fix. What surprised me was how rapidly I could cycle through that many fix attempts. The agent proposes, I test, I report, the agent adjusts. Each cycle takes minutes.

### When to use level 2

- The first fix attempt failed
- The bug involves multiple interacting components
- The root cause is not obvious from the error message alone
- Race conditions, state management issues, or non-deterministic bugs

The overhead of maintaining a tracking file is minimal. It pays off immediately for any bug that survives the first fix attempt.

# Level 3: Hypothesis-Driven Instrumentation

This is the option when you are completely lost. The error messages are misleading or nonexistent, the bug is intermittent, and guessing is getting you nowhere. Instead of trying to fix the bug, you step back and gather evidence first.

The core idea: do not ask the agent to fix anything yet. Ask it to form hypotheses and then instrument the code to test those hypotheses.

### Steps

1. Present the bug and ask the agent to generate multiple hypotheses about potential causes. **Explicitly say: do not fix anything yet.**
2. Design instrumentation: for each hypothesis, the agent adds targeted logging or debugging statements to capture runtime state.
3. Reproduce the bug with the instrumentation in place.
4. Analyze the evidence together with the agent. The logs reveal which hypothesis is correct, or point to something nobody considered.
5. Targeted fix: now, with concrete evidence, the agent proposes a precise fix instead of a speculative one.

### Example

Bug: users report that their dashboard data resets to empty after exactly 5 minutes, even though the session timeout is configured to 30 minutes.

**Step 1: Hypotheses**:
1. Session timeout configuration is not being read correctly
2. A background job or cron is clearing session data
3. The token refresh mechanism is failing silently
4. There is a cache layer with its own TTL overriding the session config

**Step 2: Instrumentation**:

```javascript
// Added to session middleware
console.log(`[SESSION] Created: ${session.createdAt}`);
console.log(`[SESSION] Configured TTL: ${config.session.timeout}ms`);
console.log(`[SESSION] Expires at: ${session.expiresAt}`);
console.log(`[SESSION] Server time: ${new Date().toISOString()}`);

// Added to token refresh endpoint
console.log(`[REFRESH] Attempting refresh for session ${sessionId}`);
console.log(`[REFRESH] Token age: ${Date.now() - token.issuedAt}ms`);
console.log(`[REFRESH] Result: ${refreshResult.status}`);

// Added to cache layer
console.log(`[CACHE] Key: user:${userId}:dashboard`);
console.log(`[CACHE] TTL set: ${cacheTTL}s`);
console.log(`[CACHE] Remaining: ${await redis.ttl(cacheKey)}s`);
```

**Step 3: Evidence from logs**:
```txt
[SESSION] Configured TTL: 1800000ms       ← 30 min, correct
[REFRESH] Attempting refresh for session abc123
[REFRESH] Result: 401                     ← refresh is failing!
[CACHE] TTL set: 300s                     ← 5 minutes!
```

**Step 4: Analysis**: Two findings. The token refresh is returning 401, and the cache TTL is hardcoded to 300 seconds. The session itself is fine, but the cached dashboard data expires after 5 minutes and the refresh call that would repopulate it is failing because it is validating against the wrong secret.

**Step 5: Fix**: Two-line change. Update the cache TTL to match the session config, and fix the secret used in refresh token validation. No guessing involved.

### When I used this

I used this approach so far once for a tricky frontend state bug in our project. The state was getting corrupted in a way that made no sense from reading the code alone. Standard debugging was not getting anywhere. So I had the agent generate hypotheses and instrument the state management layer with detailed logging. Reproducing the bug with that instrumentation revealed the exact sequence of state updates that caused the corruption. The fix was surgical.

Interestingly, this is essentially how [Cursor's debug agent](https://cursor.com/en-US/blog/debug-mode) works. Their debug mode follows the same pattern: generate hypotheses, instrument the code, collect evidence, then produce a targeted fix. It is a sound approach for hard bugs.

### When to use level 3

- The root cause is genuinely unknown
- The bug cannot be reproduced in a debugger easily
- Production issues where you need data without disrupting service
- Complex system interactions where guessing is unlikely to succeed

This approach takes more upfront time, but for the truly hard bugs, it leads to faster resolution because the fix is based on evidence instead of trial and error.

# Choosing the right level

These levels are not mutually exclusive. You can and should escalate. Start at level 1. If the first fix does not work, create a tracking file and move to level 2. If you realize you are guessing without enough data, switch to level 3 and instrument before fixing.

In practice, I rarely reach level 3. Most bugs fall at level 1, and the ones that survive get caught by level 2. But when you need level 3, you really need it.

# General tips

- Trust the agent with full logs. Do not pre-filter or summarize log output. Agents are effective at parsing and correlating logs. Give them the raw output and let them find the relevant entries.

- Be specific with feedback. _"Still broken"_ is not useful. _"Still failing, but now the error is X instead of Y"_ is. The more precise your feedback after each attempt, the faster the agent converges on the solution.

- Clear context between bugs. Once a bug is resolved, start a fresh session for the next one. Debugging history from one issue can confuse diagnosis of an unrelated problem.

# Closing thoughts

Debugging with agents is not magic. It is a structured workflow. Give them good input, track what you have tried, and when things get hard, gather evidence before guessing. The agents are doing the heavy lifting, but you are still driving.

If you want more on this topic, Anthropic published a great guide on [fixing bugs faster with Claude](https://claude.com/blog/fix-software-bugs-faster-with-claude) that covers similar ground.
