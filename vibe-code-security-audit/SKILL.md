---
name: vibe-code-security-audit
description: Audit a codebase for the security mistakes most common in AI-generated ("vibe-coded") apps, then fix them. Covers 19 vulnerabilities across authentication and authorization, hard-coded secrets and exposed API keys, injection (SQL, XSS, command), insecure data storage, missing security headers, CSRF and rate limiting, and risky dependencies (hallucinated packages, known CVEs, unsafe deserialization). Use this whenever the user wants a security review, security audit, pre-launch or pre-ship check, "is my app safe", or vulnerability scan of code they built with Cursor, Copilot, Claude, Lovable, Bolt, Replit or any AI tool, even if they never say "vibe coding". Also use it when they ask to check for leaked keys, IDOR, weak auth, SQL injection or XSS in their own project.
---

# Vibe Code Security Audit

AI coding tools produce apps that work but skip security by default. Login checks end up in browser JavaScript, API keys land in the frontend bundle, queries get built with string concatenation, and nobody adds rate limits or security headers. This skill turns a 19-item checklist into an actual audit: inspect the real code, report what is wrong with evidence, and fix it with the user's approval.

The goal is findings the user can trust and act on. A vague "looks fine" is worse than useless on a security review, so every conclusion should point at specific files and lines.

## Workflow

### 1. Orient

Before auditing, work out what you are looking at. Look at the repo root, manifests (`package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, `pom.xml`), framework config, and the folder layout. Establish:

- Language(s) and framework(s) (Express, Next.js, Django, Flask, FastAPI, Rails, Spring, serverless, Supabase/Firebase-style backend-as-a-service, etc.)
- Where server code and client code live. Many checks depend on which side of the trust boundary code runs on.
- How auth works (sessions, JWT, a third-party provider) and where the database is accessed.

If the user has not given you code access (no files, no repo), ask for it, or offer to walk through the checklist conceptually. Do not invent findings about code you have not read.

### 2. Scope

Default to a full audit of all 19 items. If the user names a concern ("just check my auth", "did I leak any keys?"), run only the relevant categories and say what you skipped. Mark items that cannot apply (for example, CSRF on a pure token-header API, or deserialization in a project with no deserializer) as Not Applicable with a one-line reason instead of padding the report.

### 3. Audit by category

Read only the reference files for the categories in scope. Each one has, per item: why it matters, what to search for, how to decide vulnerable vs fine, and what the fix looks like.

| Category | Items | Reference |
|---|---|---|
| Authentication & Authorization | 1 Client-side auth, 2 Missing authorization (IDOR), 3 Broken access control, 4 Weak sessions, 5 Auth bypass paths | `references/auth.md` |
| Secrets & Keys | 6 Hard-coded credentials, 7 API keys in client code | `references/secrets.md` |
| Injection | 8 SQL injection, 9 XSS, 10 Command injection | `references/injection.md` |
| Data Storage & Handling | 11 Insecure client storage, 12 Weak password storage, 13 Sensitive data in URLs/logs | `references/data-storage.md` |
| Headers & Protections | 14 Missing security headers, 15 No CSRF protection, 16 No rate limiting | `references/headers-protections.md` |
| Dependencies & Supply Chain | 17 Hallucinated packages, 18 Outdated libraries with CVEs, 19 Insecure deserialization | `references/dependencies.md` |

Start with the Critical items (1, 2, 3, 5, 6, 7, 8, 10, 12) since those cause the worst damage. Use grep and file reads to gather evidence; do not rely on skimming. Trace user input from where it enters (route params, body, query, headers, uploads) to where it is used (queries, shell calls, HTML output, file paths).

### 4. Report

Give the user the report before touching any code. Use this structure:

```
# Security Audit: <project name>

**Stack:** <languages/frameworks>   **Scope:** <full / categories>
**Result:** <N critical, N high, N medium found; N items clean; N not applicable>

## Findings (most severe first)

### [CRITICAL] <item number>. <title>
- **Where:** `path/to/file.ext:LINE` (one line per location)
- **Evidence:** <short snippet, with any secret value redacted>
- **Impact:** <what an attacker could actually do here>
- **Fix:** <one or two sentences; the concrete change>

## Passed / Not applicable
- <item>: <why it is fine or does not apply>

## Fix order
1. <the thing to do first, and why>
```

Severity comes from the checklist: Critical (1, 2, 3, 5, 6, 7, 8, 10, 12), High (4, 9, 11, 15, 16, 17, 18, 19), Medium (13, 14). Raise or lower a severity only with a stated reason, for example an IDOR on a public, non-sensitive resource.

Report what you found, not what you wish you had found. If a check turned up nothing, say what you searched so the user knows the clean result means something.

### 5. Fix (with permission)

After the report, ask which findings to fix, or offer to work through them in severity order. Then:

- Make the smallest change that closes the hole and fits the project's existing patterns and libraries.
- Prefer well-maintained libraries (helmet, bcrypt/argon2, a framework's own CSRF and session middleware, a rate limiter backed by a shared store) over hand-rolled security code.
- Show the user the diff or summarize each change, and tell them how to verify it (a test request, a header check, an audit command).
- Re-check the fixed item to confirm it is actually closed, then update its status.

Be upfront about anything the code change alone cannot fix. The clearest case is a leaked secret: removing it from the source does not help if it is already in git history or a deployed bundle, so tell the user to **rotate the key**, and say so prominently.

## Handling secrets safely

When you find real credentials, never print the full value in the report or in chat. Show the first few characters or a placeholder (`sk-live-AbC…`), name the file and line, and move on. Treat any secret that has ever been committed as compromised.

## Tracking status

Use these statuses so the user can follow progress across sessions:

- 🔴 Not started
- 🟡 In progress
- 🟢 Fixed
- ⚪ Not applicable

If the user wants to keep tracking outside the conversation, offer a short markdown checklist of the 19 items with current statuses.

## Limits to be honest about

A code review catches common, pattern-level mistakes. It does not replace penetration testing, a dependency scan run in the real environment, or review of infrastructure and cloud permissions. Say so briefly at the end of the report, and recommend a professional review before launching anything that handles payments, health data, or other sensitive personal data. Do not claim an app is "secure"; say which of these 19 checks it passed.

## Credit

Copyright (c) 2026 Boundverity