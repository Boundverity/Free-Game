# Authentication & Authorization (items 1-5)

Contents: 1 Client-side auth · 2 Missing authorization (IDOR) · 3 Broken access control · 4 Weak sessions · 5 Auth bypass paths

The core idea: the browser is controlled by the user, so anything enforced only there is not enforced. Every decision about who someone is and what they may touch must be made on the server, on every request.

---

## 1. Client-side authentication — Critical

**Why it matters:** If the login check or role check runs in browser JavaScript, anyone can read the source, flip a variable in DevTools, or call the API directly and skip it.

**Look for:**
- Password or PIN compared in frontend code (`if (password === "...")`)
- Role or permission checks that only hide UI (`if (user.isAdmin) showPanel()`) with no matching server check
- Token "validation" done in the browser to decide access
- Auth state read from a client-writable place (localStorage flag, cookie without server verification)

**Vulnerable vs fine:** Hiding a button based on role is fine as a UX nicety *if* the endpoint behind it also enforces the role. It is vulnerable when the client check is the only check.

**Fix:** Move credential checks and role decisions to server endpoints. The client sends credentials and receives a session token or secure cookie after the server verifies them. Delete the client-side logic.

**Verify:** No password comparisons or role gating in frontend bundles; calling the protected API directly without credentials returns 401/403.

---

## 2. Missing authorization checks (IDOR) — Critical

**Why it matters:** Being logged in does not mean you may see everything. If `GET /api/orders/1234` returns the order to any logged-in user, changing the ID reads other people's data.

**Look for:** every endpoint that takes an identifier (`userId`, `orderId`, `documentId`, etc.) in the path, query, or body. For each, find the line that confirms the requester owns or may access that exact record. Also check list endpoints: do they filter by the current user or return everything?

**Vulnerable vs fine:** Vulnerable when the handler fetches by ID alone. Fine when the query is scoped to the authenticated user (`WHERE id = ? AND owner_id = ?`) or an explicit ownership/role check runs first. Row-level security policies (Supabase/Postgres) count if they are actually enabled and correct; check for tables with RLS off.

**Fix:** Scope every ID-based query to the authenticated user, or compare `resource.ownerId` to `user.id` (or check role/ACL) before returning. Return 403 or 404 on failure. Never trust a user ID sent by the client; take it from the verified session.

**Verify:** Logged in as user A, request user B's resource ID and confirm denial.

---

## 3. Broken access controls — Critical

**Why it matters:** Same problem as IDOR but for writes. Without ownership checks, any logged-in user can edit or delete anyone's data, and admin routes may be open to ordinary users.

**Look for:** all POST, PUT, PATCH, DELETE handlers. For each, identify the target record and the acting user, and confirm the code verifies the user may perform that action. Check admin-only routes for a server-side role check, and check for mass assignment (the handler saves the whole request body, letting a user set `role: "admin"` or `ownerId`).

**Fix:** Enforce ownership checks on every mutation, apply role-based checks for admin actions, allow-list the fields a user may update, and reject with 403.

**Verify:** As a normal user, attempt to modify another user's record and to call an admin route; both must fail.

---

## 4. Weak session handling — High

**Why it matters:** Predictable, never-expiring, or reusable sessions let attackers hijack accounts.

**Check each of these:**
1. **Token generation:** cryptographically secure random source with enough entropy. Flag `Math.random()`, timestamps, sequential IDs, or home-made tokens.
2. **Expiry:** both an idle timeout and an absolute lifetime. Flag tokens that never expire.
3. **Logout:** the session is destroyed on the server, not just deleted from the browser.
4. **Transport and cookie flags:** `Secure`, `HttpOnly`, and `SameSite` set; HTTPS only.
5. **Session fixation:** a new session ID is issued after login and privilege changes.
6. **JWT specifics (if used):** strong secret from the environment, algorithm pinned (reject `none`), short expiry, and a plan for revocation.

**Fix:** Use the framework's session library or a proven one rather than rolling your own, and configure it with the settings above.

---

## 5. Authentication bypass paths — Critical

**Why it matters:** One forgotten route with no auth middleware exposes protected functionality to the whole internet. This is very common when AI adds routes one at a time.

**How to audit:**
1. Enumerate every route and method (read the router files, framework file-based routes such as `pages/api` or `app/api`, serverless functions, and any admin or debug endpoints).
2. For each, record whether auth is enforced and whether it should be.
3. Flag public routes that touch user data or sensitive actions, and inconsistent middleware (applied to some routers but not others, or skipped by a conditional).
4. Check bypass tricks: path manipulation (`/admin/../x`, encoded slashes), case differences in route matching, trailing slashes, alternate HTTP methods hitting an unprotected handler, and leftover test or debug routes (`/api/test`, `/debug`, `/seed`).

**Fix:** Apply authentication middleware at the router or app level so protected routes are protected by default and public routes are the explicit exception. Normalize paths before authorization checks.

**Deliver:** include the route inventory (method, path, auth yes/no, should-be) in the report; it is the most useful artifact for this item.
