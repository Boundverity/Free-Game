# Injection Vulnerabilities (items 8-10)

Contents: 8 SQL injection · 9 Cross-site scripting (XSS) · 10 Command injection

The common thread: untrusted input gets interpreted as code. For each finding, trace the data from its source (request param, body, header, upload, database field written by another user) to the dangerous sink, and state that path in the report.

---

## 8. SQL injection — Critical

**Why it matters:** String-built queries let an attacker read, change, or delete the whole database.

**Look for:**
- String concatenation or template literals inside queries: `` `SELECT * FROM users WHERE id = ${id}` ``, `"... WHERE name = '" + name + "'"`, f-strings and `%` formatting in SQL
- ORM "raw" escape hatches fed with user input (`sequelize.query`, `prisma.$queryRawUnsafe`, `knex.raw`, Django `.raw()` / `.extra()`, SQLAlchemy `text()` with interpolation)
- Dynamic table or column names and `ORDER BY` built from user input (parameters cannot bind identifiers; these need an allow-list)
- NoSQL equivalents: query objects built straight from request bodies (for example MongoDB operators like `$ne` or `$where`)

**Fix:** Use parameterized queries or prepared statements, or the ORM's safe query builder. Allow-list anything that cannot be a bound parameter. Validate types on input. Give the database user the minimum privileges it needs (no DROP, no superuser).

**Verify:** no query is built by joining strings with user data; the DB account is least-privilege.

---

## 9. Cross-site scripting (XSS) — High

**Why it matters:** Injected scripts run as the victim, stealing sessions or acting on their behalf.

**Look for:**
- React `dangerouslySetInnerHTML`, Vue `v-html`, Angular `[innerHTML]`/`bypassSecurityTrust*` with unsanitized data
- `element.innerHTML = ...`, `outerHTML`, `insertAdjacentHTML`, `document.write`, jQuery `.html()` with user-influenced values
- Server-rendered templates with escaping turned off (`|safe`, `{{{ }}}`, `<%- %>`, `Markup()`)
- User content in `href`/`src` (`javascript:` URLs), inline event handlers, and `eval`/`new Function`
- Markdown or rich text rendered to HTML without a sanitizer

**Fix:** Use framework bindings that escape by default or `textContent`. If HTML must be rendered, sanitize it with a trusted library (for example DOMPurify) first. Remove `document.write`. Add a Content-Security-Policy as defense in depth (see `headers-protections.md`), avoiding `unsafe-inline` and `unsafe-eval` where possible.

**Verify:** no unsanitized sink receives user data; CSP is present.

---

## 10. Command injection — Critical

**Why it matters:** User input reaching a shell lets an attacker run arbitrary commands on your server.

**Look for:** `exec`, `execSync`, `spawn` with `shell: true`, `child_process`, Python `os.system`, `subprocess` with `shell=True`, `popen`, Ruby backticks/`system`, PHP `shell_exec`/`passthru`, Go `exec.Command("sh", "-c", ...)`. Also file-handling code that passes filenames to tools (ImageMagick, ffmpeg, git, pdf converters).

**Decide first:** is the shell call needed at all? Often a native library does the same job. If it is removed, no injection surface remains.

**If it must stay:** pass arguments as an array, not one interpolated string; disable shell interpretation (`shell=False`); validate input against a strict allow-list; guard against arguments that begin with `-` being read as options.

**Related:** path traversal in file reads and writes (`../` in user-supplied filenames) is a close cousin. Flag it if you see it, and resolve paths against a fixed base directory.

**Verify:** no user-influenced data reaches a shell string.
