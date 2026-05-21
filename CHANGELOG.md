# Changelog

Усі зміни плагіна `plusone-creative`. Формат — [Keep a Changelog](https://keepachangelog.com/), версіонування — [SemVer](https://semver.org/).

## [0.1.0] — 2026-05-21

Перший реліз. Pilot для дизайнерів Plusone.

### Added
- Маніфест плагіна (`.claude-plugin/plugin.json`).
- Одноплагінний marketplace (`.claude-plugin/marketplace.json`) для установки через `claude plugin marketplace add HoshovskyiV/plusone-creative`.
- Конфіг remote MCP серверів (`.mcp.json`) — Figma + Higgsfield, обидва через HTTP transport з OAuth через Claude Desktop UI.
- Головний skill `plusone-creative` з 10-кроковим workflow (parse brief → style extraction → content matrix → parallel asset generation → frame composition → validation → retry → report).
- 4 reference docs у `skills/plusone-creative/references/`:
  - `style-extraction.md` — як інвентаризувати Figma-проект.
  - `doodle-deconstruction.md` — 6-осьова система опису стилю дудлів + 7 готових формулювань.
  - `prompt-patterns.md` — шаблони Higgsfield промптів, включно з hard rule проти checkerboard для gpt-image-1.5 transparent PNG.
  - `figma-conventions.md` — очікувані структури шаблонів Plusone, два варіанти graceful degradation.
- 4 sub-agents у `agents/`:
  - `style-analyzer` — інвентар Figma → JSON style profile.
  - `asset-generator` — паралельні Higgsfield calls для дудлів + фото.
  - `frame-composer` — duplicate template + fill slots у Figma з reuse компонентів і змінних.
  - `validator` — screenshot-based перевірка composed frames.
- Hard dependencies на офіційні skills `figma-use` і `figma-generate-design` (copied from `figma/mcp-server-guide`).
- README українською з 5 готовими промпт-шаблонами для Cowork.
- HANDOFF.md — покрокова інструкція встановлення для дизайнера.

### Known limitations
- Tool restrictions для sub-agents винесені у Hard rules секції body замість frontmatter `tools:` field — точні MCP tool name prefixes валідуються першим pilot-run.
- Доступ до Google Drive / Sheets для content briefs покладається на Drive connector у Claude Desktop — не тестовано.
- Validator не перевіряє типографічну ієрархію (правильно H1 використано чи ні) — тільки overflow та detached styles.
- Доодл retry — максимум 1 раз, після цього падає у warnings без зупинки workflow.

### Notes
- v0.1.0 — pilot. Після фідбеку буде v0.2.0 з конкретними фіксами.
