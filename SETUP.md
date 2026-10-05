# Setup Guide

This guide explains how to reproduce the multi-agent job application pipeline.

> The exact model names and provider limits may change over time. Treat the versions below as the configuration used for this project, not a permanent requirement.

---

## 1. Requirements

Recommended environment:

- Linux / Kubuntu
- Node.js 22+
- Google Chrome
- Git
- Paseo
- Claude Code
- OpenAI Codex
- Playwright

Check Node and npm:

```bash
node --version
npm --version
```

---

## 2. Install and Test Claude Code

Install Claude Code using Anthropic's current official installation method.

Verify:

```bash
claude --version
```

Run:

```bash
claude
```

Authenticate and make sure a normal prompt succeeds before connecting Claude to Paseo.

---

## 3. Install and Test Codex

Install the OpenAI Codex CLI using OpenAI's current official instructions.

Verify:

```bash
codex --version
```

Authenticate and test it independently before using it through Paseo.

---

## 4. Install Paseo

Install Paseo on:

- the machine that runs the agents
- optionally, a second machine used to manage/monitor the host

Verify:

```bash
paseo --version
```

In Paseo, enable the host's **Paseo tools / agent orchestration tools** so the primary operator can launch and message other agents.

After changing orchestration settings, start a fresh agent session so the tools are available in that session.

---

## 5. Create a Dedicated Chrome Profile

Do not automate the browser profile you use for normal browsing.

Launch a dedicated Chrome profile:

```bash
google-chrome-stable \
  --user-data-dir="$HOME/chrome-job-automation" \
  --remote-debugging-port=9222
```

Leave this browser running.

Verify CDP:

```bash
curl -s http://localhost:9222/json/version
```

A healthy response should contain fields similar to:

```text
Browser
webSocketDebuggerUrl
```

The agents connect to:

```text
http://localhost:9222
```

### Why use a dedicated profile?

It isolates:

- job-board logins
- Gmail
- LinkedIn
- application tabs
- browser automation state

from the user's normal browser profile.

---

## 6. Log Into Required Sites Manually

Before autonomous operation, use the dedicated Chrome profile to log into the services you want the workflow to use.

Examples:

- LinkedIn
- Gmail
- Indeed
- StepStone
- XING
- employer career portals

Do not store credentials directly in prompts.

---

## 7. Recommended Workspace

Example:

```text
Resumes/
├── Job_Application_Tracker.xlsx
├── Technical/
│   ├── technical_de.pdf
│   └── technical_en.pdf
├── NonTechnical/
│   ├── nontechnical_de.pdf
│   └── nontechnical_en.pdf
└── Private/
```

Normal application assets should be clearly separated from private documents.

### Active assets

The agents may normally use:

- the tracker
- technical CVs
- non-technical CVs

### Private assets

Anything under:

```text
Private/
```

should be explicitly off-limits unless the user authorizes a specific file.

---

## 8. File-Handling Rules

The operator should be instructed to:

- attach actual local PDFs directly
- never base64-encode CVs
- never serialize PDF binaries into prompts
- avoid sending full PDF contents to reviewers
- avoid creating duplicate CVs
- avoid unnecessary temporary files
- avoid creating duplicate trackers
- never inspect `Private/` without authorization

This reduces token usage and limits accidental exposure.

---

## 9. Create Paseo Agent Profiles

Create these profiles:

1. `Claude - Job Operator`
2. `Claude - Application Reviewer`
3. `Codex - Application Reviewer`
4. `Codex - Job Operator`

See [AGENTS.md](AGENTS.md) for detailed settings and responsibilities.

---

## 10. Recommended Profile Settings

### Claude - Job Operator

Example configuration used by this project:

```text
Provider: Claude Code
Model: Claude Sonnet 4.6
Context: Standard
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Claude - Application Reviewer

```text
Provider: Claude Code
Model: Claude Sonnet 4.6
Context: Standard
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Codex - Application Reviewer

```text
Provider: Codex
Model: GPT-5.6 Sol
Reasoning / Effort: Medium
Auto Accept: Enabled
```

### Codex - Job Operator

```text
Provider: Codex
Model: GPT-5.6 Sol
Reasoning / Effort: Medium
Auto Accept: Enabled
```

The exact available model names may differ in future versions.

---

## 11. Browser Ownership Rule

Only one operator may control Chrome at once.

Normal:

```text
Claude - Job Operator
```

Fallback:

```text
Codex - Job Operator
```

Never let both manipulate the same browser session simultaneously.

---

## 12. Run a Production-Readiness Dry Run

Before allowing real submissions, verify:

- Chrome CDP connectivity
- visible browser control
- job search
- opening a real vacancy
- file isolation
- reviewer launch
- reviewer response
- fallback operator launch
- Manual Review policy

Do not submit real applications during the first test.

---

## 13. Test Codex Orchestration

A saved Paseo profile is not the same thing as a running agent.

The Claude operator should be able to:

- launch `Codex - Application Reviewer`
- wait for its response
- reuse that reviewer
- launch `Codex - Job Operator` as an idle fallback

A successful test should prove that the profiles can be instantiated on demand.

---

## 14. Move to Production

Once the dry-run tests pass:

- enable real submissions
- keep Codex review selective
- preserve blocked applications in Manual Review
- keep the tracker as persistent source of truth
- continue searching after each completed application

See [AGENTS.md](AGENTS.md) and [FAILOVER.md](FAILOVER.md).
