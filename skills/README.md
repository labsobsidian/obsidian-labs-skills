# Skills

Reusable methods for building and operating Obsidian Labs client AI ops stacks. Each skill is a self-contained folder with a `SKILL.md` that tells Claude (or a human) exactly how to execute that method.

Skills are referenced by client repos via `.skills-version`, never copy-pasted. Improvements to a skill flow to all clients on the next version bump.

## Active skills

| Skill | What it does | Runs |
|---|---|---|
| `new-client-spinup` | Scaffolds a fresh client ops repo with the 5 living docs, pushes to GitHub, creates the Claude Project | Once, at client kickoff |
| `project-state-update` | Updates living docs at the end of a work session so context doesn't drift | At the end of every meaningful session |
| `atlas-github-brain` | Wires Atlas to read its KB live from the client's GitHub repo, replacing hardcoded system prompts | Once per client, after initial Atlas UI is deployed |

## Pending skills

Patterns we're tracking but haven't turned into full skills yet. When a pattern shows up in two or more client conversations, or we actually implement it for a client, it gets promoted to an active skill.

**From Chem-Dry work** (patterns already built but not yet extracted):

- `booking-funnel-deploy` — 4-step Next.js booking funnel + GHL webhook
- `ghl-subaccount-provision` — stand up a GHL sub-account for a new client
- `hcp-api-integration` — connect Housecall Pro API to Supabase via n8n
- `a2p-registration-concierge` — walk a client through A2P SMS registration
- `weekly-ops-report-generator` — n8n workflow generating Monday reports

**From Hardesty work** (patterns built but not yet skillified):

- `booking-form-with-lead-scoring` — HTML booking form + Google Maps Places + lead score + GHL webhook
- `ghl-lead-nurture-workflow` — Day 0/1/3/7 follow-up sequence
- `ghl-approval-workflow` — owner approval via SMS one-click link
- `elevenlabs-voice-agent-setup` — ElevenLabs + Twilio + GHL routing
- `atlas-commit-endpoint` — Phase C of the GitHub brain: write-back endpoint for Atlas to edit its own repo

**From client conversations** (patterns requested, not yet implemented):

- `atlas-ar-tab` — Accounts Receivable tab in Atlas. Invoice aging, outstanding balances. Source: QuickBooks (requires `atlas-quickbooks-connector` first). Requested by Musser Biomass.

**Future connectors for the Atlas KB** (to be built as additional data sources alongside `atlas-github-brain`):

- `atlas-ghl-connector` — live GHL pipeline in the KB
- `atlas-acculynx-supabase-sync` — nightly AccuLynx → Supabase sync, KB reads from Supabase
- `atlas-supabase-queries` — real-time queries against synced client data
- `atlas-gmail-connector` — recent email thread context
- `atlas-quickbooks-connector` — invoicing + P&L + A/R source

## Versioning

See `CHANGELOG.md` at the repo root.

## How to add a skill

See `/meta/WRITING_SKILLS.md`. Short version: follow the SKILL.md format, test it on a real build, commit, tag a new version.
