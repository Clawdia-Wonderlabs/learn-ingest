# Changelog

All notable changes to the learn-ingest skill.

---

## [1.1.0] — 2026-04-27

### Added

**[A] Insights — confidence-scored action item routing**
Action items extracted during the Insights pass are now scored 0–100 and automatically routed:

| Score | Action |
|---|---|
| 80–100 | Auto-create in Google Tasks (`account: personal`) |
| 60–79 | Listed under **Flagged for Review** in vault note |
| < 60 | Silently omitted |

Scoring factors: specificity of the action, fit to current projects, effort level. Each auto-routed task includes confidence score, course name, community, and vault path in the notes field. Uses `mcp__google__tasks_add`.

**[H] Scan Vault — new action**
Search existing vault content without ingesting anything new. Useful for "what do I already know about X?" before starting a project. Returns lessons and mining extracts ranked by keyword hit count with matching snippets. Can be invoked standalone or as the pre-flight check at intake start.

**Step 0 — Pre-flight vault scan**
Before asking material type, the skill now automatically scans the vault for existing content on the same topic. Reports matches and asks whether to review them first or proceed with new content. Skips silently if no matches.

### Fixed

- Typo in Step C: "Point to the point to the source" → "Ask for the source"
- Typo in [F] Teach Me: "Let you's response" → "Let your response"
- Action menu description for [A] now notes confidence-based task routing

---

## [1.0.0] — initial release

- Guided intake flow: material type → community → source → action selection
- Actions A–G: Insights, Implement, Content Mining, Conversation Mining, Skill Extraction, Teach Me, Just Archive
- Google Drive MCP integration for folder/file ingestion
- VTT/SRT timestamp stripping
- Python-based vault writes (no heredocs/echo)
