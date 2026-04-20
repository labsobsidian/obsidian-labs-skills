# New Client Spin-Up

**Create a fresh client ops repo with the full living-docs scaffolding, ready for a first work session.**

## When to use this skill

When a new client signs and Obsidian Labs is starting work on their AI ops stack. Also when adding a new client repo to an existing agency relationship (e.g., second location, additional brand).

## Prerequisites

- Client discovery is at least partially complete (we know stack, primary contacts, goals)
- GitHub access under the `labsobsidian` org
- Access to the `obsidian-labs-skills` repo to know current skills version

## Inputs

- **Client name** (e.g., "Chem-Dry of Richmond")
- **Client slug** (e.g., `chem-dry`) — used for repo name as `<slug>-ops`
- **Primary contacts** (name, email, role)
- **Stack** — which services will be involved (GHL, HCP, Supabase, Vercel, n8n, Inngest, etc.)
- **Mission** — one paragraph on what this project is delivering
- **Current skills version** — pin to the current latest tag from `obsidian-labs-skills`

## Method

### 1. Create the GitHub repo
gh repo create labsobsidian/<slug>-ops --private --description "<Client Name> AI ops command center" --add-readme

Or use GitHub web UI.

### 2. Clone locally
cd ~/obsidian-labs    # or your parent folder
git clone https://github.com/labsobsidian/<slug>-ops.git
cd <slug>-ops

### 3. Create `.skills-version`

A single-line file pinning the current skills library version:
v0.1.0
### 4. Create the five living docs from templates

Use the skeleton template in `obsidian-labs-skills/templates/client-docs-skeleton/` (or the fallback inline templates below if the templates folder isn't populated yet).

Required files:
- `PROJECT_STATE.md` — Mission, Current Phase, Completed (empty), Active Work (empty), Remaining, Last Updated
- `ARCHITECTURE.md` — Stack, Data Flow, Service Responsibilities (stub sections OK to start)
- `CONSTANTS.md` — IDs and URLs (fill in what's known, mark unknowns as TODO)
- `DECISIONS.md` — empty with header
- `GO_LIVE_CHECKLIST.md` — empty with header

### 5. Fill in what's knowable at kickoff

- Mission paragraph in PROJECT_STATE
- Stack list in ARCHITECTURE
- Known IDs in CONSTANTS (GHL location ID if created, etc.)
- Initial "Remaining" work in PROJECT_STATE based on scope

Anything unknown gets `TODO` marker, not guessed values.

### 6. Commit and push
git add .
git commit -m "Initial repo scaffolding for <Client Name>"
git push

### 7. Create the Claude Project

- In Claude.ai, create a new Project named `<Client Name> — Obsidian Ops`
- Paste the Claude Project Instructions from `obsidian-labs-skills/meta/CLAUDE_PROJECT_TEMPLATE.md`, replacing placeholders
- Attach the GitHub MCP connector, scoped to the client's ops repo and the skills repo

### 8. Run the first-session smoke test

In the new project, ask: *"What's the mission and what should we work on first?"*

If Claude answers using the repo docs, the spin-up is complete.

## Outputs

- New GitHub repo under `labsobsidian/<slug>-ops` with full scaffolding
- `.skills-version` pinning the current skills library version
- Claude Project created and wired to GitHub
- Verified first-session smoke test

## Verification

- Repo visible on GitHub with all 5 living docs + `.skills-version`
- Claude Project can read the repo via MCP
- Claude correctly summarizes mission and next work from docs alone

## Known gotchas

- **Forgetting `.skills-version`.** Without it, skill references are ambiguous and can drift silently as the library evolves.
- **Filling CONSTANTS with guesses.** If you don't know the GHL location ID yet, write `TODO` — not a placeholder that might get used as if real.
- **Skipping the smoke test.** A repo and a project that aren't talking to each other look fine until the first real session fails.

## Related skills

- `project-state-update` — the maintenance skill after spin-up
- (future) `ghl-subaccount-provision` — typically runs alongside spin-up for GHL-based clients