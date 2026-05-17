# Settings

This memory root stores durable context for Agents Remember.

Use this Markdown file for human and agent instructions, scaffold notes, and operational context. Machine-readable storage, path-rule, and cross-repo settings live in `system/settings.json`.

Do not duplicate active `pathRules` here as the authoritative machine source when `system/settings.json` exists.

## Scaffold

| Layer         | Location               | Purpose                                                     |
| ------------- | ---------------------- | ----------------------------------------------------------- |
| instructions  | `system/settings.md`   | Human and agent guidance, path contract, and scaffold notes |
| path settings | `system/settings.json` | Machine-readable storage, pathRules, and cross-repo data    |
| sources       | `system/sources.md`    | External and domain documentation registry                  |
| tools         | `system/tools.md`      | Repo-specific commands, checks, and local tool notes        |
| onboarding    | `onboarding/`          | Durable repo and file-level code commentary                 |
| docs          | `docs/`                | Local domain docs, mirrors, and reference material          |
