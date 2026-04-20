# The PROJECT_STATE Pattern

Every client repo maintains 5 living docs that together form the project's memory. These replace the "summary prompt at the start of every session" ritual. Claude reads them, works on them, and updates them.

## The 5 docs

### 1. `PROJECT_STATE.md` — what IS

The current state of everything. Updated at the end of every work session.

**Sections:**
- **Mission** — one paragraph on what this project is
- **Current Phase** — one line on where we are (e.g., "Phase 2: integration")
- **Completed** — checklist of finished work
- **Active Work** — what's in progress, with status and blockers
- **Remaining** — what's next, ordered by priority
- **Last Updated** — date

### 2. `ARCHITECTURE.md` — why it's built this way

The decisions and diagrams. Why Inngest for some workflows and n8n for others. Data flow from booking funnel to GHL to Supabase to HCP. Webhook topology. Which service owns what.

Updated when architecture changes. Rare but important.

### 3. `CONSTANTS.md` — the IDs and URLs

GHL location ID, Supabase project ref, Vercel project URL, webhook endpoints, phone numbers, notification emails. The stuff Claude needs to plug into code and workflows.

**Never put secrets here.** API keys and passwords go in `.env` files or Vercel env vars, not in the command center.

### 4. `DECISIONS.md` — append-only log

Every non-trivial architectural decision, with date and reasoning.

**Format:**
YYYY-MM-DD: Brief title of the decision
Context: what situation prompted this
Decision: what we chose
Reasoning: why
Alternatives considered: what else we looked at
Revisit if: conditions under which we'd reconsider

Never delete from DECISIONS.md. If a decision is reversed, add a new entry explaining the reversal.

### 5. `GO_LIVE_CHECKLIST.md` — the pre-launch punch list

The items that must be true before this system is live in production. Flip test-mode flags, swap notification emails, add production Stripe keys, publish GHL workflows, etc.

When the project goes live, archive this (rename to `GO_LIVE_CHECKLIST_ARCHIVED.md`) so the history is preserved.

## How Claude uses these

At the start of any conversation in the client project, Claude is instructed to:
1. Read `PROJECT_STATE.md` first
2. Read `CONSTANTS.md` if the task involves any infrastructure
3. Reference `ARCHITECTURE.md` when making changes that touch multiple services
4. Read `DECISIONS.md` when proposing a change that might conflict with a prior decision

## The discipline that makes this work

**Update at the end of every session.** Don't "batch" updates — they drift.

**Prefer additions over deletions.** Move completed items to the Completed section rather than deleting them. History is valuable.

**Keep sections small.** If `PROJECT_STATE.md` grows past 500 lines, it's time to archive old completed work into a `HISTORY.md`.