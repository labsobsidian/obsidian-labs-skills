# Project State Update

**Update a client project's living docs at the end of a work session so the next session can pick up with zero re-priming.**

## When to use this skill

At the end of any meaningful work session on a client project. Specifically:
- After a feature is shipped
- After an architectural decision is made
- After a blocker is discovered or resolved
- Before Garrett signs off for the day

If the session was trivial (just answered a question, no changes), updates may not be needed — use judgment.

## Prerequisites

- Client repo is present and accessible via GitHub MCP connector
- The five living docs exist: `PROJECT_STATE.md`, `ARCHITECTURE.md`, `CONSTANTS.md`, `DECISIONS.md`, `GO_LIVE_CHECKLIST.md`

## Inputs

- What was done this session (Claude already knows from the conversation)
- What was decided
- What blockers emerged or resolved
- Any new constants introduced

## Method

### 1. Summarize the session's changes in plain terms

Before editing anything, state: *"Here's what I'm about to update in the project state..."* and list the changes. This gives Garrett a chance to correct or add anything before the commit.

### 2. Update `PROJECT_STATE.md`

- Move completed items from "Active Work" or "Remaining" to "Completed"
- Add new items to "Active Work" if work is ongoing
- Update the "Last Updated" date
- If "Completed" has grown past ~40 items, propose archiving older ones to `HISTORY.md`

### 3. Update `CONSTANTS.md` if any new IDs/URLs/endpoints were introduced

- New webhook URLs
- New Vercel deployment URLs
- New Supabase table names if they're referenced by external systems
- Never put secrets here

### 4. Append to `DECISIONS.md` if an architectural choice was made

Use the format:
YYYY-MM-DD: Brief title
Context: what prompted this
Decision: what we chose
Reasoning: why
Alternatives considered: what else we looked at
Revisit if: conditions under which we'd reconsider

Never delete prior entries. If reversing a decision, add a new entry referencing the old one.

### 5. Update `GO_LIVE_CHECKLIST.md` if pre-launch items were completed or added

### 6. Update `ARCHITECTURE.md` only if architecture actually changed

Most sessions don't touch this. If a new service was added, a data flow changed, or a component's responsibility shifted, update it. Otherwise leave it alone.

### 7. Commit

Use a clear commit message following this format:
state: <one-line summary of what changed>

bullet of specific change
bullet of specific change

Example:
state: Review-to-Referral SMS workflow shipped

Added GHL workflow CD-Review-to-Referral-SMS
New webhook constant in CONSTANTS.md
Moved from Active Work to Completed in PROJECT_STATE

### 8. Propose skillification (if applicable)

Ask: *"Is this a one-off or should we skillify this for future clients?"* If reusable, draft a new `SKILL.md` in a branch on `obsidian-labs-skills` and open a PR.

## Outputs

- Updated living docs committed to the client repo
- Clean git history showing what changed and why
- A ready-to-go next session with no context debt

## Verification

Open a fresh Claude conversation in the client project. Without providing any context, ask: *"What's the current state of the project and what should we work on next?"*

If Claude answers correctly using only the repo docs, the state update succeeded.

## Known gotchas

- **Forgetting to actually commit.** Updating the files locally but not pushing means the next session sees stale state. Always push.
- **Treating DECISIONS.md as a diary.** It's for architectural choices, not a log of everything done. Routine work goes in PROJECT_STATE.md only.
- **Skipping this skill because "the session was small."** Small sessions still move the ball. A two-line update beats no update.
- **Putting secrets in CONSTANTS.md.** Never. Secrets live in environment variables.

## Related skills

- `new-client-spinup` — creates the initial versions of these docs
- (future) `session-kickoff` — the inverse: how Claude reads state at the start of a session