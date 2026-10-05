# Architecture

## Overview

The system separates four concerns:

1. job discovery and screening
2. browser execution
3. independent review
4. persistent campaign state

![System Architecture](Process%20Pipeline.png)

---

## Main Flow

```text
Job Boards / Company Sites
          |
          v
    Claude Operator
          |
          +---- routine application ----> self-check
          |
          +---- complex application ----> Codex Reviewer
                                           |
                                           v
                                  APPROVED / CHANGES
          |
          v
       Submit
          |
          v
       Verify
          |
          v
       Tracker
```

---

## Components

### Operator Layer

The active operator:

- searches
- screens
- controls Chrome
- fills forms
- selects files
- prepares text
- submits
- verifies
- tracks

Normally this is Claude.

Codex is the fallback operator.

### Review Layer

The review layer is separate from browser execution.

The reviewer does not need to interact with the live form in most cases.

Instead, it receives a structured summary.

This makes review:

- cheaper
- safer
- easier to audit
- easier to swap between providers

### Browser Layer

Chrome runs as a separate persistent process.

Example:

```bash
google-chrome-stable \
  --user-data-dir="$HOME/chrome-job-automation" \
  --remote-debugging-port=9222
```

The active operator attaches through CDP:

```text
http://localhost:9222
```

### State Layer

The application tracker acts as the source of truth.

Agent conversational memory is not treated as durable campaign state.

---

## Why Persistent External State Matters

AI sessions can:

- compact context
- restart
- hit provider limits
- fail unexpectedly
- be replaced by another operator

The tracker survives these events.

Important campaign facts therefore live outside the model.

---

## Browser Ownership

Exactly one operator owns Chrome.

### Safe

```text
Claude Operator
     |
     v
   Chrome
```

### Safe Failover

```text
Claude stops
     |
     v
Codex takes ownership
     |
     v
   Chrome
```

### Unsafe

```text
Claude ----\
            > Chrome
Codex  ----/
```

Multiple browser operators can cause:

- duplicate submissions
- conflicting clicks
- navigation races
- lost form state
- incorrect attachments
- closed tabs

---

## Manual Review Queue

Manual-only blockers are isolated from the main pipeline.

```text
Application
    |
    v
CAPTCHA / MFA / ATS Block
    |
    v
Manual Review Queue
    |
    +----> Operator continues another job
```

The blocked application remains available for later human intervention.

---

## Reviewer Escalation

The operator does not use cross-provider review for everything.

### Routine

```text
Claude Operator
→ self-check
→ submit
```

### High Value / Uncertain

```text
Claude Operator
→ Codex Reviewer
→ correction
→ submit
```

This is an architectural response to quota constraints.

---

## Failure Boundaries

Failures should stay local whenever possible.

Examples:

- one CAPTCHA does not stop the campaign
- one broken ATS does not stop search
- reviewer quota exhaustion does not automatically kill browser work
- operator quota exhaustion triggers a handoff rather than loss of campaign state

---

## Observability

During debugging, the pipeline can expose only major stage transitions:

```text
[CHROME CONNECTED]
[SEARCH START]
[LEAD FOUND]
[JOB OPENED]
[REVIEW START]
[REVIEW COMPLETE]
[SUBMISSION READY]
```

This gives visibility without dumping private reasoning.

---

## Design Principle

The system should behave more like a small distributed workflow than a single long chat:

- specialized workers
- external state
- explicit ownership
- queues
- health checks
- failover
- bounded retries
