# Free-Game

A growing collection of ready-to-use [Claude skills](https://www.anthropic.com/news/skills) that anyone can download, drop in, and use. Each skill teaches Claude how to do one job properly, so you don't have to write the prompt from scratch every time.

---

## What's a skill?

A skill is a folder with a `SKILL.md` file (plus optional reference files) that tells Claude how to handle a specific kind of task. Claude loads it only when it's relevant, so you can install a bunch without cluttering your chats.

## Skills

| Skill | What it does | Folder |
|---|---|---|
| 🔐 **vibe-code-security-audit** | Audits AI-generated ("vibe-coded") apps against 19 common security mistakes, like leaked API keys, missing auth checks, SQL injection and weak password storage, then helps fix them | [`skills/vibe-code-security-audit`](skills/vibe-code-security-audit) |

More coming. Watch or star the repo to catch new drops.

## Install

**Claude (web / desktop / mobile)**
1. Download the skill from the [Releases](../../releases) page (a `.skill` file), or zip the skill's folder.
2. Upload it in your Claude skills settings.
3. Start a new chat and ask for the thing the skill does. Claude picks it up automatically.

**Claude Code**
```bash
# one skill, available in every project
git clone https://github.com/<your-username>/free-game.git
cp -r free-game/skills/vibe-code-security-audit ~/.claude/skills/

# or just for one project
cp -r free-game/skills/vibe-code-security-audit your-project/.claude/skills/
```

## Using a skill

You don't need to name the skill. Just ask naturally. For example, with `vibe-code-security-audit` installed:

> "Run a security audit on this repo before I launch."

> "Did I leak any API keys in my frontend?"

> "Check my auth routes for IDOR."

## Repo layout

```
free-game/
├── README.md
├── LICENSE
└── skills/
    └── <skill-name>/
        ├── SKILL.md          # what it does and how Claude should run it
        └── references/       # extra detail Claude loads only when needed
```

## Contributing

Got a skill that should be free for everyone? Open a PR.

1. Fork the repo and add your skill under `skills/<skill-name>/` (lowercase, hyphens).
2. Make sure `SKILL.md` has a `name` and a clear `description` that says when the skill should trigger.
3. Keep `SKILL.md` focused and move long detail into `references/`.
4. Add a row to the table above.
5. Open a pull request and say what the skill is for and how you tested it.

Skills must be safe: no malware, no data exfiltration, nothing that surprises the user about what it does.

## A note on security skills

Security skills here do code review: they catch common, pattern-level mistakes. They are not a substitute for a professional penetration test, especially for apps that handle payments or sensitive personal data.

## Credits
Boundverity technologies Ltd

## License

[MIT](LICENSE) unless a skill's folder says otherwise.

---

Copyright (c) 2026 Boundverity
