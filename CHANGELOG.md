# Changelog

All notable changes to the Obsidian Labs skills library.

Follows [Semantic Versioning](https://semver.org/): MAJOR.MINOR.PATCH.

## [0.3.0] — 2026-04-22

### Added
- New skill: `atlas-github-brain` — wires Atlas to read live KB from the client's GitHub repo, replacing any hardcoded system prompt. Ships alongside reference implementation in `labsobsidian/hardesty-ops` @ 7898647.
- Skills index (`skills/README.md`) restructured with active + pending sections.
- Pending-skills tracking: Chem-Dry patterns awaiting extraction; Hardesty patterns awaiting skillification; client-requested features (Musser A/R tab); planned Atlas KB connectors (GHL, Supabase, Gmail, QuickBooks, AccuLynx).

### Notes
- Phase C of the GitHub brain (`/api/commit` endpoint for Atlas write-back to the repo) is deferred. Tracked as pending skill `atlas-commit-endpoint`.

## [0.2.0] — 2026-04-20

### Added
- First two skills: `project-state-update`, `new-client-spinup`
- Skills index at `skills/README.md`

## [0.1.0] — 2026-04-20

Initial scaffolding. Meta docs: WORKING_LOOP, WRITING_SKILLS, PROJECT_STATE_PATTERN, CLAUDE_PROJECT_TEMPLATE.
