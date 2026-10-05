# Agent Configuration

The system uses specialized agents rather than asking one model to perform every role.

This improves:

- clarity of responsibility
- review quality
- failure isolation
- quota efficiency
- browser safety

---

## 1. Claude - Job Operator

### Recommended Configuration

```text
Provider: Claude Code
Model: Claude Sonnet 4.6
Context: Standard
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Role

The main operator is responsible for:

- discovering jobs
- screening vacancies
- controlling Chrome
- selecting the correct CV
- filling application forms
- preparing required text
- performing factual self-checks
- delegating complex reviews
- applying reviewer corrections
- submitting applications
- verifying success
- updating campaign state
- maintaining Manual Review tabs
- continuing to the next opportunity

### Important Rules

The operator must:

- be the only active browser operator
- use actual local PDF files
- avoid `Private/`
- avoid duplicate submissions
- check the tracker before applying
- preserve blocked tabs instead of stopping the whole campaign
- use Codex only when the additional review is worth the quota

---

## 2. Claude - Application Reviewer

### Recommended Configuration

```text
Provider: Claude Code
Model: Claude Sonnet 4.6
Context: Standard
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Role

Optional independent Claude review.

Useful when:

- the operator wants a second perspective
- Codex quota should be conserved
- the application is not important enough to justify cross-provider review

### Output Format

```text
APPROVED
```

or:

```text
CHANGES REQUIRED:
- exact material issue
- exact required correction
```

This reviewer does not normally control Chrome or submit applications.

---

## 3. Codex - Application Reviewer

### Recommended Configuration

```text
Provider: Codex
Model: GPT-5.6 Sol
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Role

Codex is treated as a scarce, high-value independent reviewer.

Use it for:

- important cybersecurity/security roles
- technically complex applications
- substantial free-text answers
- recruiter outreach
- legal/work-authorization questions
- salary/compensation questions
- ambiguous eligibility
- unusual mandatory requirements
- material uncertainty flagged by Claude

### Input Format

To reduce context overhead, send only what is needed:

```text
Company:
Role:
Location:
Key mandatory requirements:
Selected CV:
Important answers:
Relevant free text:
Specific uncertainty:
```

Do not send full PDF contents unless absolutely necessary.

### Output Format

```text
APPROVED
```

or:

```text
CHANGES REQUIRED:
- exact material issue
- exact required correction
```

---

## 4. Codex - Job Operator

### Recommended Configuration

```text
Provider: Codex
Model: GPT-5.6 Sol
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Role

Emergency fallback operator.

Use it when Claude is:

- unavailable
- rate-limited
- quota-limited
- unable to continue

The Codex operator should inherit the same operating rules as Claude.

### Critical Rule

Before Codex takes browser ownership, Claude must stop starting new browser actions.

Exactly one operator may control Chrome.

---

## 5. Persistent Agent Reuse

Where possible, reuse:

- one persistent Codex reviewer
- one persistent fallback Codex operator

Do not spawn a new reviewer for every application.

Benefits:

- less repeated context
- lower token usage
- simpler monitoring
- easier failover
- cleaner Paseo session list

---

## 6. When to Use Codex Review

Routine application:

```text
Claude Operator
→ self-check
→ submit
```

Complex application:

```text
Claude Operator
→ Codex Reviewer
→ fix if required
→ submit
```

Codex should not be used just because it is available.

---

## 7. Language Filtering

Candidate language constraints should be implemented as explicit rules.

Example:

If a vacancy explicitly requires:

```text
B2+
C1
fluent German
fließend Deutsch
sehr gute Deutschkenntnisse
verhandlungssicher Deutsch
```

and the candidate does not meet that level:

```text
SKIP
```

If German is only:

```text
preferred
wünschenswert
von Vorteil
```

the vacancy may still be considered.

This prevents the agent from treating hard language requirements as soft preferences.

---

## 8. Manual Review Policy

These blockers should not stop the whole campaign:

- CAPTCHA
- reCAPTCHA
- Cloudflare
- MFA
- login verification
- security confirmation
- broken ATS controls
- manual-only verification

For a blocked application:

1. preserve the tab
2. preserve entered state where possible
3. label it Manual Review
4. record the remaining action
5. continue another job

Do not attempt to bypass security controls.

---

## 9. Full Prompts

See:

- [Claude Job Operator](prompts/claude_job_operator.md)
- [Claude Application Reviewer](prompts/claude_application_reviewer.md)
- [Codex Application Reviewer](prompts/codex_application_reviewer.md)
- [Codex Job Operator](prompts/codex_job_operator.md)
