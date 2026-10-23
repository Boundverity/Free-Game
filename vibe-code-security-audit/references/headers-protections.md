# Security Headers & Protections (items 14-16)

Contents: 14 Missing security headers · 15 No CSRF protection · 16 Missing rate limiting

First identify the framework and where global middleware or response config lives, so fixes are applied once, centrally, not route by route.

---

## 14. Missing security headers — Medium

**Why it matters:** A handful of headers close off whole classes of attack and take minutes to add. Most vibe-coded apps ship with none.

**Check the headers the app actually sends** (read the middleware/config, and if the app is running, inspect a real response):

| Header | Purpose / target value |
|---|---|
| `Strict-Transport-Security` | Force HTTPS. `max-age` of a year or more, `includeSubDomains`; add `preload` only if the user understands it is hard to undo |
| `Content-Security-Policy` | Limit where scripts, styles, images, and connections may load from. Start from `default-src 'self'`, define `script-src`, `style-src`, `img-src`, `connect-src`, `font-src`, `frame-ancestors`, and list required third-party domains explicitly. Avoid `unsafe-inline` and `unsafe-eval` |
| `X-Frame-Options` | `DENY` (or CSP `frame-ancestors`) to stop clickjacking |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Disable browser features the app does not use (camera, microphone, geolocation) |

**Fix:** Use the framework's standard tool (`helmet` for Express, `next.config` headers for Next.js, `SecurityMiddleware` settings for Django, `flask-talisman` for Flask, Spring Security headers) and configure it globally. Roll out CSP in report-only mode first if the app is complex, then enforce, since a too-strict CSP can break pages. Also check the CORS config: wildcard origins combined with credentials is a serious misconfiguration.

**Verify:** inspect response headers in DevTools or with `curl -I`; run a header scanner if the user has one.

---

## 15. No CSRF protection — High

**Why it matters:** If the browser sends credentials automatically (cookies), a malicious site can trigger state-changing requests on behalf of a logged-in user.

**Decide applicability first:** CSRF matters when authentication rides on cookies. A pure API that authenticates with an `Authorization: Bearer` header set by JavaScript is largely not exposed (mark Not Applicable with that reason). Apps that moved tokens into cookies for item 11 become exposed, so check them.

**Look for:** POST, PUT, PATCH, DELETE endpoints with no CSRF token validation and cookies with no `SameSite` attribute, and any state change performed via GET.

**Fix:**
- Use the framework's CSRF middleware (Django's `CsrfViewMiddleware`, Flask-WTF, Spring Security, or a maintained Express alternative; the old `csurf` package is deprecated, so recommend a current option).
- Generate a per-session token, validate it on every state-changing request, return 403 on failure.
- Forms get a hidden token field; AJAX sends it in a header such as `X-CSRF-Token`.
- Set session cookies with `SameSite=Lax` or `Strict`, plus `Secure` and `HttpOnly`.
- Never change state on GET.

**Verify:** a state-changing request with a missing or wrong token is rejected.

---

## 16. Missing rate limiting — High

**Why it matters:** Without limits, attackers can brute-force logins, enumerate accounts, spam signups and password resets, or run up costs on expensive endpoints (especially anything that calls a paid AI or email API).

**Look for:** a rate-limiting layer on login, signup, password reset, OTP/verification, and public or expensive API endpoints. Note whether limits use a shared store (needed when running multiple instances; in-memory limits reset on restart and do not sync).

**Suggested starting policies** (tune to real traffic):
- Login and auth endpoints: about 5 attempts per IP per 15 minutes, with increasing lockouts on repeated violations
- Password reset: about 3-5 per hour per IP and per email
- Registration: about 5-10 per hour per IP
- General public API: a baseline per IP per 15 minutes appropriate to expected usage

**Fix:** Use the framework's established limiter (`express-rate-limit` with a Redis store, `django-ratelimit`, `Flask-Limiter`, Bucket4j, or the platform's built-in/edge rate limiting). Return `429` with a `Retry-After` header. Key by IP and, where relevant, account identifier. If behind a proxy or CDN, configure trusted proxies correctly so the real client IP is used, otherwise attackers can spoof `X-Forwarded-For` or everyone shares one IP.

**Verify:** exceeding the limit returns 429 and the limit holds across restarts and instances.
