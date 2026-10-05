# Lessons Learned

Building this system exposed problems that were more interesting than the form automation itself.

---

## 1. Browser Automation Was Not the Hard Part

Opening pages and filling fields is relatively straightforward.

The harder problems were:

- state
- ownership
- retries
- quotas
- handoff
- review boundaries
- duplicate prevention
- failure recovery

---

## 2. Multiple Agents Need Explicit Ownership

The system became much safer after introducing a simple rule:

> Exactly one operator controls Chrome.

Without that rule, multiple agents could:

- click different elements
- navigate away from forms
- close each other's tabs
- upload conflicting files
- submit the same application twice

---

## 3. Model Quotas Affect Architecture

Provider limits are not just billing details.

They directly influence system design.

Using a strong independent reviewer on every routine application wastes quota.

The final design separates:

- routine execution
- high-value review
- emergency fallback

---

## 4. Persistent State Should Live Outside the Model

Agent sessions can:

- compact
- restart
- disappear
- hit context limits
- hit provider limits

Important campaign state should therefore live externally.

The tracker acts as the persistent source of truth.

---

## 5. CAPTCHA Should Not Stop the Pipeline

A single CAPTCHA should not kill an otherwise productive autonomous run.

The Manual Review queue changed the behavior from:

```text
CAPTCHA → entire campaign stops
```

to:

```text
CAPTCHA → preserve tab → queue → continue another job
```

---

## 6. Reusing Agents Matters

Creating a fresh reviewer for every application adds:

- repeated initialization
- repeated context
- extra usage
- noisy orchestration

Persistent reviewers are more efficient.

---

## 7. Bounded Tasks Are Better Than Open-Ended Tasks

Search agents can spend too long "still searching."

A better pattern is:

```text
Find a maximum of N strong leads.
Return immediately when enough are found.
```

This improves both latency and quota usage.

---

## 8. Structured Review Packages Beat Full Context Dumps

A reviewer usually does not need:

- the entire webpage
- the entire PDF
- large browser dumps

A concise package is usually enough:

```text
Company
Role
Requirements
CV selected
Important answers
Free text
Specific uncertainty
```

This dramatically reduces token overhead.

---

## 9. Debug Visibility Should Be Temporary

During setup, stage-level debug messages are useful:

```text
[SEARCH START]
[LEAD FOUND]
[REVIEW START]
```

During production, constant narration wastes context and attention.

The ideal system is observable when needed, quiet when healthy.

---

## 10. Hard Filters Must Actually Be Hard

If the candidate has German A2 and the vacancy explicitly requires:

```text
B2
C1
fluent German
sehr gute Deutschkenntnisse
```

the system should skip it.

An agent should not reinterpret hard requirements as "probably flexible" just to maximize application count.

---

## 11. Real-World Automation Needs Queues

The workflow naturally became queue-based:

- active applications
- Manual Review
- pending recruiter outreach
- follow-up
- fallback work

This is more reliable than treating the entire campaign as one continuous linear task.

---

## 12. The Project Became a Distributed-System Problem

The final design involved:

- specialized workers
- persistent state
- ownership
- resource constraints
- queues
- failover
- health checks
- bounded retries

That was the most interesting engineering lesson from the project.
