# Secrets & Keys (items 6-7)

Contents: 6 Hard-coded credentials · 7 API keys in client-side code

Never print a full secret value in your output. Redact to a few leading characters and name the file and line.

---

## 6. Hard-coded credentials — Critical

**Why it matters:** Secrets in source get committed to git and stay in history even after deletion. Anyone with repo access (or a public repo) can use them.

**Look for:**
- Database passwords and connection strings with credentials (`postgres://user:pass@host`)
- API keys and tokens (Stripe `sk_live_`/`sk_test_`, OpenAI `sk-`, AWS `AKIA…`, SendGrid `SG.`, GitHub `ghp_`, Slack `xox`)
- Private keys (`-----BEGIN ... PRIVATE KEY-----`), JWT signing secrets, encryption keys
- High-entropy strings assigned to variables named `secret`, `token`, `key`, `password`
- Committed `.env` files, config files, Docker/compose files, CI configs, notebooks, and test fixtures
- Git history: check whether secrets were committed earlier even if they are gone from the working tree (for example `git log -p -S` on suspicious strings)

**Fix:**
1. Remove the secret from code and move it to an environment variable loaded at runtime.
2. Add `.env` (and other secret files) to `.gitignore`.
3. Create a `.env.example` listing every required variable with placeholder values.
4. **Rotate every secret that was ever committed.** Deleting it from the file does not make it safe again.
5. If history contains secrets, recommend purging it (`git filter-repo` or BFG) after rotation, and note this rewrites history for collaborators.

**Verify:** no secrets in tracked files; `.env` ignored; app reads config from the environment.

---

## 7. Exposed API keys in client-side code — Critical

**Why it matters:** Everything in a frontend bundle is public. Anyone can open DevTools or view the built JavaScript and copy the key, then run up your bill or access your data.

**Look for:**
- Third-party secret keys in frontend source, built output, or framework env vars that get inlined into the bundle (`NEXT_PUBLIC_*`, `VITE_*`, `REACT_APP_*`, `EXPO_PUBLIC_*` are public by design)
- Direct calls from the browser to paid or privileged APIs (OpenAI, Stripe secret operations, SendGrid, AWS) using a key the browser holds

**Important distinction:** Some keys are designed to be public and are fine in the client, such as Stripe publishable keys (`pk_`), Firebase web config, and Google Maps keys that are restricted by referrer. Flag keys that grant privileged access, and for the "public" kind confirm they are restricted (domain/referrer limits, security rules) and not over-permissioned. Supabase: the anon key is public by design, but the service-role key must never appear in the client.

**Fix:**
1. Remove the key from frontend code.
2. Store it in a server-side environment variable.
3. Add a backend proxy endpoint: the frontend calls your endpoint, which validates input, enforces authentication and rate limits, calls the external API with the server-held key, and returns only the data the client needs.
4. Point the frontend at the proxy.
5. **Rotate the exposed key**, since it has already been public.

**Verify:** search the built bundle for the key; no privileged keys remain in client output.
