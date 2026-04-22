# CLAUDE.md — Obsidian Labs Skills Library (obsidian-labs-skills)

You are working on the Obsidian Labs shared skills library, maintained by Garrett.

This is the agency's IP — a versioned library of reusable methods for building and operating Atlas deployments. Every client repo references this library. Skills compound: each solved problem deposits a reusable method that speeds up the next client build.

---

## Always read first

1. `CHANGELOG.md` — current version and what changed
2. `skills/README.md` — index of all available skills
3. `/meta/WRITING_SKILLS.md` — how to write a good SKILL.md

---

## The Working Loop

```
Problem → Method → Approval → Build → Update State → (maybe) Skillify
```

In this repo, the primary action IS skillification. When a pattern from real client work is reusable, it gets promoted here.

---

## What This Repo Is

The shared, versioned method library. Not client code — that lives in client repos. This is:
- Reusable skills (SKILL.md files) that Claude Code and Garrett can execute
- Meta docs explaining how the Obsidian Labs system works
- Templates referenced by skills (grows from real client work)
- The source of truth for which version of skills each client is built against

---

## Repo Structure

```
obsidian-labs-skills/
├── README.md                        ← Library purpose + how clients reference it
├── CHANGELOG.md                     ← Semver log (currently v0.2.0)
├── skills/
│   ├── README.md                    ← Skills index
│   ├── project-state-update/        ← How to update living docs at end of session
│   │   └── SKILL.md
│   └── new-client-spinup/           ← How to scaffold a new client ops repo
│       └── SKILL.md
├── templates/                       ← Empty — grows from real client work
└── meta/
    ├── WORKING_LOOP.md              ← The Problem→Method→Approval→Build→Update→Skillify discipline
    ├── WRITING_SKILLS.md            ← How to write a good SKILL.md
    ├── PROJECT_STATE_PATTERN.md     ← The five living docs spec
    └── CLAUDE_PROJECT_TEMPLATE.md  ← Template for Claude.ai project instructions
```

---

## Current Version: v0.2.0

### Active Skills
- `project-state-update` — updates living docs at end of session
- `new-client-spinup` — scaffolds a new client ops repo end-to-end

### Skills Pending Promotion (from Chem-Dry work)
These patterns exist in chem-dry-ops but haven't been extracted into reusable skills yet:
- `booking-funnel-deploy` — deploy a 4-step Next.js booking funnel + GHL webhook
- `ghl-subaccount-provision` — stand up a GHL sub-account for a new client
- `hcp-api-integration` — connect HCP (Housecall Pro) API to Supabase via n8n
- `a2p-registration-concierge` — walk a client through A2P SMS registration
- `weekly-ops-report-generator` — n8n workflow that generates Monday reports
- `chem-dry-redeploy` — meta-skill for forking the Chem-Dry stack for new home-service clients

### Skills Pending Promotion (from Hardesty work)
- `ghl-lead-nurture-workflow` — Day 0/1/3/7 lead follow-up sequence
- `ghl-approval-workflow` — owner approval via SMS one-click link
- `elevenlabs-voice-agent-setup` — ElevenLabs + Twilio + GHL routing
- `booking-form-with-lead-scoring` — HTML booking form + Google Maps + lead score + GHL webhook
- `atlas-brain-deploy` — deploy the 7-tab Atlas Brain app to Vercel with streaming

---

## Versioning Rules

- **Major bump:** Breaking change to a skill's contract (inputs/outputs change)
- **Minor bump:** New skill added, non-breaking improvements
- **Patch:** Typo fixes, clarifications

Client repos pin a version in `.skills-version`. To upgrade a client: update their `.skills-version` file and test.

---

## How to Write a Good SKILL.md

Every skill follows this format:
1. One-sentence description
2. When to use this skill
3. Prerequisites
4. Inputs
5. Method (numbered steps)
6. Outputs
7. Verification
8. Known gotchas
9. Related skills

See `/meta/WRITING_SKILLS.md` for full guidance.

**Key rule:** Write for Claude-tomorrow. The person reading it in 6 months has none of your context. Over-explain the why. Include failure modes — lessons learned are the most valuable content.

---

## Conventions

- Skills are referenced, never copy-pasted into client repos
- Templates grow from real work — don't invent them abstractly
- When in doubt, draft the skill — skills can be improved later
- Semver tag every meaningful change
- Never delete skills — deprecate with a note if superseded

## Tone

Direct, concrete, no filler. Skills are executable methods, not documentation. If you can't execute your own skill, nobody else can either.
