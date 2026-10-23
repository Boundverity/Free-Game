# Dependencies & Supply Chain (items 17-19)

Contents: 17 Hallucinated packages · 18 Outdated libraries with known CVEs · 19 Insecure deserialization

---

## 17. Hallucinated packages — High

**Why it matters:** AI models sometimes recommend packages that do not exist. Attackers register those names with malicious code ("slopsquatting"), so a blind `npm install` or `pip install` runs their code. Typosquats of popular packages are the same risk.

**How to audit:** Read every dependency manifest (`package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, `composer.json`) and the lock files. For each dependency, check:
- It exists on the official registry under exactly that name (if network access is available, query the registry; otherwise flag it for the user to verify)
- The name is not a near-miss of a well-known package (`reqeusts`, `colourama`, `lodash-utils`)
- It has a real repository, maintainers, release history, and meaningful usage
- No sign of abandonment or a suspicious recent ownership change

Pay extra attention to packages you do not recognize, very low-download packages, and anything added in a single AI-assisted commit.

Also check install scripts: `postinstall` hooks in `package.json` and unusual `setup.py` behavior.

**Report:** a list of dependencies with verification status, plus a separate "flagged for manual review" section with the reason for each. Be honest when you could not verify something rather than marking it safe.

**Fix:** Remove packages that cannot be verified, replace with well-known alternatives, and make it a habit to verify any package an AI suggests before installing it.

---

## 18. Outdated libraries with known CVEs — High

**Why it matters:** Known-vulnerable dependencies are one of the most common real-world entry points, and the exploits are public.

**How to audit:** Run the audit tool that matches the stack, if the environment allows it:

| Stack | Command |
|---|---|
| Node | `npm audit` (or `pnpm audit`, `yarn npm audit`) |
| Python | `pip-audit` |
| Ruby | `bundle audit` |
| Rust | `cargo audit` |
| Go | `govulncheck ./...` |
| PHP | `composer audit` |

If you cannot run tools (no shell or no network), read the manifests and lock files, call out versions that are obviously old, and give the user the exact command to run themselves. Do not claim a clean result you did not obtain.

**Report grouped by severity** (Critical, High, Medium, Low) with: package and current version, CVE ID(s), short description, minimum fixed version, and the exact upgrade command. Prefer non-breaking upgrades first and flag any that need a major version bump.

**Automate it:** add Dependabot (`.github/dependabot.yml` with the project's ecosystem and a weekly or daily schedule, security updates on) or Renovate (`renovate.json`) so this does not depend on memory.

**Also check:** lock files are committed and current, and dependency versions are pinned or constrained sensibly.

---

## 19. Insecure deserialization — High

**Why it matters:** Deserializing untrusted data with a format that can create arbitrary objects can lead to remote code execution.

**Look for:**
- Python: `pickle.load(s)`, `marshal.load(s)`, `shelve`, `yaml.load()` without a safe loader, `jsonpickle`
- Java: `ObjectInputStream.readObject` on untrusted streams, polymorphic JSON typing in Jackson/Gson
- PHP: `unserialize()` on user data
- Ruby: `Marshal.load`, YAML `load` instead of `safe_load`
- Node: `node-serialize`, `eval` or `new Function` on parsed content, unsafe YAML loaders
- Any JSON parser configured to instantiate classes from type hints in the data

**Decide:** the risk exists only when the data can be influenced by a user (request bodies, cookies, uploads, cached values an attacker can write, message queues). Deserializing your own trusted files is lower risk; say so.

**Fix:** Use plain data formats (strict JSON), `yaml.safe_load()` or equivalent, avoid object instantiation during parsing, and validate against a schema (JSON Schema, Pydantic, Zod, Joi) before using the data. Remove deserialization that is not needed.

**Verify:** no untrusted data reaches an object-creating deserializer; inputs are schema-validated.
