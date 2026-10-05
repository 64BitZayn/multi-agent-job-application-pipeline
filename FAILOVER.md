# Failover Strategy

AI providers have practical limits:

- rate limits
- weekly quotas
- temporary outages
- context limits
- failed sessions
- tool failures

The pipeline is designed so that one provider failing does not necessarily stop the campaign.

---

## Normal State

```text
Claude - Job Operator
        |
        +----> Codex - Application Reviewer (when needed)
```

Claude owns Chrome.

Codex remains a reviewer or idle fallback.

---

## Claude Operator Becomes Unavailable

Before switching operator:

1. stop new Claude browser actions
2. preserve existing Chrome tabs
3. preserve partially completed forms
4. preserve Manual Review tabs
5. preserve tracker state
6. preserve pending URLs
7. confirm only the fallback operator will continue browser work

Then:

```text
Claude Operator
      X
      |
      v
Codex - Job Operator
      |
      v
same Chrome CDP session
```

The browser and tracker become the handoff state.

---

## Reviewer Becomes Unavailable

Codex review is valuable but not required for every application.

If Codex is unavailable:

- routine applications can continue with Claude self-check
- high-risk applications can be deferred
- Claude Application Reviewer can be used where appropriate

The pipeline should not consume time repeatedly retrying an unavailable reviewer.

---

## Quota Conservation

Independent review is intentionally selective.

### Routine

```text
Claude
→ self-check
→ submit
```

### Complex

```text
Claude
→ Codex review
→ submit
```

This stretches provider quota much further than dual-model review on every form.

---

## Persistent Reviewer Reuse

Prefer:

```text
one Codex Reviewer
→ many review tasks
```

over:

```text
new Codex Reviewer
new Codex Reviewer
new Codex Reviewer
...
```

This reduces repeated initialization and context overhead.

---

## Manual Blockers

CAPTCHA, MFA, Cloudflare, and similar blockers do not trigger operator failover.

They are application-level blockers, not campaign-level failures.

```text
Blocked application
      |
      v
Manual Review
      |
      +----> continue another job
```

---

## Browser Failure

If the CDP endpoint itself is unavailable:

```text
http://localhost:9222
```

the operator should stop browser work.

Do not launch uncontrolled duplicate browser instances unless explicitly intended.

The browser should be restored first, then the pipeline resumed.

---

## Handoff Checklist

Before operator takeover:

- [ ] old operator stopped new Chrome actions
- [ ] current browser still running
- [ ] pending tabs preserved
- [ ] tracker preserved
- [ ] Manual Review queue preserved
- [ ] fallback operator identified
- [ ] fallback connects to same CDP endpoint
- [ ] no second operator remains active

---

## Future Supervisor

A future improvement is a dedicated supervisor that:

- checks operator health
- checks reviewer health
- resumes idle workers
- performs provider failover
- ensures exactly one browser owner
- avoids duplicate workers

This should be added only after the basic operator/reviewer architecture is stable.
