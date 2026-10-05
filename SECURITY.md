# Security and Privacy

This project interacts with personal information, authenticated browser sessions, and external websites.

Security boundaries are therefore part of the architecture.

---

## Never Commit

Do not publish or commit:

- API keys
- passwords
- OAuth tokens
- browser cookies
- browser profile directories
- session tokens
- private CVs
- residence permits
- identity documents
- home addresses
- phone numbers
- real application tracker data
- private recruiter contact details

---

## Private Workspace

Sensitive material should be isolated.

Example:

```text
Private/
```

Agents should be explicitly instructed:

```text
Do not inspect, open, attach, upload, summarize, or use anything inside Private/
unless the user explicitly authorizes that specific file.
```

---

## Browser Profile

The dedicated Chrome profile can contain authenticated sessions.

Never commit:

```text
~/chrome-job-automation/
```

Treat the profile as sensitive credential material.

---

## API Keys

Do not:

- paste keys into public prompts
- commit keys to Git
- include them in screenshots
- store them in README examples

Use provider-supported authentication or environment variables as appropriate.

---

## CAPTCHA / MFA

The system does not attempt to bypass:

- CAPTCHA
- reCAPTCHA
- MFA
- Cloudflare verification
- security challenges
- employer access controls

These become Manual Review events.

---

## Screenshots

Before publishing screenshots:

- blur emails
- blur phone numbers
- remove addresses
- hide application IDs
- remove access tokens
- hide recruiter private contact data
- crop unrelated tabs
- verify browser extensions are not exposing secrets

---

## Repository Hygiene

Recommended `.gitignore` entries:

```gitignore
.env
.env.*
*.key
*.pem

chrome-job-automation/
Private/

*.xlsx
*.pdf

cookies*
session*
tokens*
```

If you intentionally publish a sanitized spreadsheet or PDF, explicitly unignore that file.

---

## Least-Privilege Principle

The active agent workspace should contain only what is needed for routine operation.

Avoid giving autonomous agents access to sensitive files simply because those files happen to be in the same directory.

---

## Human Authorization

Sensitive documents should require explicit authorization for each intended use.

The existence of a file is not permission to upload it.
