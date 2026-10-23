# Data Storage & Handling (items 11-13)

Contents: 11 Insecure client-side storage · 12 Weak password storage · 13 Sensitive data in URLs and logs

---

## 11. Insecure client-side storage — High

**Why it matters:** `localStorage` and `sessionStorage` can be read by any JavaScript on the page, including a script injected through XSS. A stolen token is a stolen account.

**Look for:** every use of `localStorage`, `sessionStorage`, and non-HttpOnly cookies. For each, note what is stored and whether it is sensitive (JWTs, session IDs, refresh tokens, role flags, personal data).

**Fix:** Keep auth tokens out of browser storage. Have the server set them in cookies with `HttpOnly`, `Secure`, and an appropriate `SameSite`. Update the frontend to rely on the cookie rather than reading the token. Cookie-based auth makes CSRF protection necessary (item 15), so fix both together.

**Not a problem:** harmless UI preferences (theme, last-opened tab) in localStorage.

**Verify:** no tokens or personal data in browser storage; auth cookies carry the three flags.

---

## 12. Weak password storage — Critical

**Why it matters:** Plain text, MD5, SHA-1, or unsalted SHA-256 are cracked in seconds once the database leaks, and people reuse passwords across sites.

**Look for:** registration, login, and password-reset code. Identify how passwords are stored and compared: plain text, `md5`/`sha1`/`sha256` calls, home-made salting, or a proper password hash. Also check that passwords are never logged or returned in API responses.

**Fix:**
- Use `bcrypt` (cost factor 12 or higher) or `Argon2id`; let the library handle salting.
- Compare with the library's verify function, which is constant-time.
- Do not truncate or alter the password before hashing.
- Migrate existing weak hashes without forcing resets: at login, if the stored hash is the legacy type, verify it, then re-hash with the new algorithm and save.

If the app uses a managed auth provider (Auth0, Clerk, Supabase Auth, Firebase Auth), this item is usually handled for the user; confirm there is no parallel custom password table, then mark it Not Applicable.

**Verify:** no plain text or fast-hash passwords in the database or code.

---

## 13. Sensitive data in URLs and logs — Medium

**Why it matters:** URLs end up in browser history, server logs, proxies, analytics, and referrer headers. Log files often live for months and are widely readable.

**Look for:**
- Tokens, passwords, API keys, reset codes, or session IDs in query strings (for example `?token=...`)
- Personal data (emails, names) in URL paths
- Log statements that dump whole request or response objects, headers, cookies, or bodies
- Debug logging left on in production
- Error responses that expose stack traces, SQL, file paths, or internal details to users

**Fix:**
- Move sensitive data to POST bodies or the `Authorization` header.
- Use structured logging that selects specific fields instead of dumping objects.
- Redact passwords, tokens, cookies, and authorization headers before logging.
- Use different log levels in development and production.
- Return generic error messages to users and log the detail server-side.
- Note one-time links (password reset, email verification) legitimately put a token in the URL; make them single-use, short-lived, and not logged.

**Verify:** no credentials in URLs (other than short-lived one-time links); logs are scrubbed; production errors are generic.
