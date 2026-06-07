# Changelog

All notable changes to `baoyu-design` are tracked in this file.

## Unreleased

### Added

- Added this changelog to keep project updates in one place.

### Changed

- Clarified that final design and prototype delivery should include the running preview, not only the generated file.
- Updated Claude Code, Codex Agent, and Cursor harness references to make final preview handoff visible to the user after verification.
- Updated the English and Simplified Chinese README files to document Codex Agent support, the new `references/codex.md` harness map, install examples, usage notes, and changelog links.

## 2026-06-06

### Added

- Added Codex Agent support with `skills/baoyu-design/references/codex.md`, covering question asking, browser preview, screenshots, debugging, deliverable surfacing, and subagent verification defaults.
- Added English and Simplified Chinese README files.
- Added the portable `baoyu-design` Agent Skill, core design methodology, built-in skill prompts, starter components, and harness reference docs for Claude Code and Cursor.
- Added the initial repository files and MIT license.

### Changed

- Updated the skill entry flow to detect Codex Agent and fall back cleanly for Claude Desktop-like or other file-capable harnesses.
- Reworded preview and verification instructions in built-in export skills so they route through the selected harness reference instead of hard-coded preview tool names.
