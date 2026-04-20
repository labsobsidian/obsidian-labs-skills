# The Obsidian Labs Working Loop

This is the discipline that makes the system self-reinforcing. Every feature, every bugfix, every change follows this loop. Claude is instructed to follow it in every client project.

## The Loop
Problem → Method → Approval → Build → Update State → (maybe) Skillify
### 1. Problem

Garrett (or a team member) describes what needs to happen. Often in natural language, sometimes forwarded from a client.

*Example: "Katie wants an SMS to go out 30 minutes after a customer clicks the review link, asking if they want to refer a friend."*

Claude's job at this step:
- Read `PROJECT_STATE.md` to check for related context or existing work
- Read `ARCHITECTURE.md` to understand where this change fits
- Read `CONSTANTS.md` for any IDs, URLs, or credentials needed
- Ask clarifying questions if the problem is ambiguous
- **Do not start building yet**

### 2. Method

Claude proposes the solution before building. A method includes:
- Which services are touched (GHL, n8n, Supabase, Vercel, etc.)
- Which files change
- Which new files are created
- Any new env vars or credentials needed
- Rough estimate of complexity (simple / medium / significant)

This step is fast — usually 4–10 bullet points. Its purpose is to surface disagreements early.

### 3. Approval

Garrett approves, rejects, or modifies the method. Sometimes a single message like "go." Sometimes "change step 3, then go." If significant disagreement, return to step 1.

**Claude does not skip this step.** Even for small changes. The 30-second approval prevents 30 minutes of wrong direction.

### 4. Build

Execute the approved method. Claude writes code, runs commands, explains what it's doing. If the method turns out to be wrong mid-build, **stop and return to step 2** — don't silently improvise.

### 5. Update State

When the build works, update `PROJECT_STATE.md`:
- Move the item from "Active Work" or "Remaining" to "Completed"
- If any architectural decisions were made, log them in `DECISIONS.md`
- If any new constants were introduced (webhook URLs, IDs), add to `CONSTANTS.md`
- If any new blockers surfaced, add to "Active Work"

This step is non-negotiable. It's what makes the project self-documenting and eliminates summary prompts forever.

### 6. (Maybe) Skillify

At the end of a build, Claude asks: **"Is this a one-off for this client, or will we do this again for other clients?"**

If reusable, Claude drafts a `SKILL.md` for `obsidian-labs-skills` and proposes adding it to the library. Examples of things that deserve skillification:
- Any setup done more than once (A2P registration, HCP API integration, GHL sub-account provisioning)
- Any pattern that generalizes across verticals (booking funnel deploy, referral flow, nightly sync)
- Any hard-won lesson (the Adminify booking function silent-fail discovery is skill-worthy)

One-off client things (a specific copy change, a client-unique workflow) don't need skillification.

## Why this loop matters

Without the loop, every session starts from zero. You re-prime context, you re-explain architecture, you rediscover decisions you already made.

With the loop, every session builds on the last. The project state is the source of truth. Claude reads it, works on it, updates it. Skills compound. Client #5 takes 10% of the time of client #1.

## Anti-patterns to watch for

- **Silent improvisation.** Claude changes the method mid-build without telling you. If the approach needs to change, pause and re-propose.
- **State drift.** Forgetting to update PROJECT_STATE.md at the end of a session. One skipped update becomes two, becomes a dead doc.
- **Premature skillification.** Turning every small change into a skill. Skills are for reusable methods, not logs of what happened.
- **Reluctant skillification.** The opposite — refusing to promote patterns to skills because it "doesn't feel like a real skill yet." When in doubt, draft it. Skills can be improved later.