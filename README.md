# Obsidian Labs — Skills Library

The shared, versioned library of reusable methods for building and operating Obsidian Labs client AI ops stacks. This repo is the agency's IP.

## What lives here

**`/skills`** — Reusable methods. Each skill is a self-contained folder with a `SKILL.md` that tells Claude (or a human) exactly how to execute that method. Skills are the building blocks of every client deployment.

**`/templates`** — Boilerplate config and code that skills reference. e.g., the canonical `config/client.json` shape, a skeleton Next.js booking funnel, GHL workflow export examples.

**`/meta`** — How the Obsidian Labs system itself works. Conventions, the working loop, the PROJECT_STATE pattern, how to write a good skill.

## How clients use it

Each client has their own repo (e.g., `chem-dry-ops`, `hardesty-ops`). Those repos reference this one via:

1. A `.skills-version` file in the client repo that pins which version of skills they're built against (e.g., `v1.0.0`)
2. Claude's project instructions for that client include a reference to `labsobsidian/obsidian-labs-skills` so skills are readable via the GitHub MCP connector

Skills are never copy-pasted into client repos. Always referenced.

## How to add or improve a skill

See `/meta/WRITING_SKILLS.md`. Short version: follow the SKILL.md format, test it on a real build, commit, tag a new version.

## Versioning

Semver. Breaking changes to a skill's contract = major bump. Non-breaking improvements = minor. Typo fixes = patch.

Client repos pin a version in `.skills-version`. To upgrade a client's skills baseline, update their `.skills-version` file and test.

## Current version

See `CHANGELOG.md`.