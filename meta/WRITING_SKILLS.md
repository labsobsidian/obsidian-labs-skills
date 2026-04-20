# How to Write a Good SKILL.md

Skills are reusable methods. A well-written skill means the next time the situation comes up — for the same client or a new one — the method is already defined and Claude can execute it without improvising.

## Skill folder structure

Each skill lives in its own folder under `/skills`:
skills/
booking-funnel-deploy/
SKILL.md           ← the method (required)
assets/            ← any code templates, config examples (optional)

The folder name is the skill's ID. Use lowercase-kebab-case. Be specific: `ghl-a2p-registration` not `a2p`.

## SKILL.md format

Every SKILL.md follows this structure:

```markdown
# <Skill Name>

**One-sentence description of what this skill does.**

## When to use this skill

The trigger conditions. Be specific. This is what lets Claude (or a human) recognize when the skill applies.

## Prerequisites

What must already exist before running this skill. e.g., "Client repo exists with config/client.json," "GHL sub-account created," "HCP API key in Vercel env vars."

## Inputs

What Claude needs to know before executing:
- Client location ID
- Service-specific credentials
- Any client-specific customizations

## Method

The actual steps. Numbered, ordered, concrete. Each step should be a thing a person or Claude can actually do.

## Outputs

What exists after the skill runs. e.g., "Webhook endpoint live at <url>," "New GHL workflow published," "Supabase table `X` created and populated."

## Verification

How to confirm the skill worked. Specific tests, not vibes.

## Known gotchas

Lessons learned. Places where this skill has failed before. Things that trip people up.

## Related skills

Other skills commonly run before or after this one.
```

## Rules of thumb

**Be specific, not aspirational.** "Deploy the booking funnel" is too vague. "Deploy a 4-step Next.js booking funnel to Vercel, wire GHL webhook inbound, create a contact + opportunity in the target pipeline" is a skill.

**Write for Claude-tomorrow.** The person reading this skill in 6 months (you, or a Claude instance, or a VA) has none of the context you have now. Over-explain the "why" for non-obvious decisions.

**Include the failure modes.** The Adminify silent-fail discovery from Chem-Dry belongs in a skill. Lessons are the most valuable content.

**Version when a skill's contract changes.** If a skill previously assumed Inngest but now supports n8n too, that's a minor version bump and a note in the skill itself + CHANGELOG.

**When in doubt, write it.** A rough skill today beats a perfect skill that never gets written.

## Testing a new skill

Before committing, run the skill end-to-end on a real build (or a sandbox). If you can't execute your own skill, nobody else can either.