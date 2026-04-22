# Atlas GitHub Brain

**Wire an Atlas deployment to read its knowledge base live from a client's GitHub repo, replacing any hardcoded system prompt with a real, updatable brain.**

## When to use this skill

- First real brain for any new Atlas client deployment (runs after `new-client-spinup`)
- Retrofitting an existing Atlas demo (hardcoded KB) into a live one
- Whenever "the brain is fake" — i.e. the chat's knowledge base is a static string in frontend code rather than sourced from the client's living docs

Do NOT use this skill for:
- Adding a second data source to an already-running GitHub-brain Atlas — that's a different skill (`atlas-ghl-connector`, `atlas-acculynx-sync`, etc.)
- Non-Atlas projects. This pattern assumes the UI/chat conventions established by the Atlas template.

## Prerequisites

- Client ops repo exists (e.g. `labsobsidian/hardesty-ops`) with at least `PROJECT_STATE.md`; ideally the full 5 living docs from `new-client-spinup`
- Vercel project linked to that repo, with auto-deploy from `main`
- GitHub org access (`labsobsidian`) with permission to create fine-grained PATs
- The repo already contains a minimal Atlas UI (an `index.html` that POSTs to `/api/claude` and reads SSE chunks with `content_block_delta` events) and a current `/api/claude.js` that proxies to Anthropic with streaming

## Inputs

- **Client name** (e.g. `Hardesty Roofing`) — displayed in preambles and UI
- **Client slug** (e.g. `hardesty`) — reserved for future connector namespacing
- **Client repo** (e.g. `labsobsidian/hardesty-ops`)
- **Anthropic API key** — usually already set on the Vercel project from the demo phase

## Method

### 1. Install the Octokit dependency locally

From the client repo root:

```bash
npm init -y    # only if there's no package.json yet
npm install @octokit/rest
```

**Important:** edit the generated `package.json` to include `"type": "module"` at the top level. The context + claude endpoints use ES module syntax (`import`/`export`) and Vercel's runtime handles this either way, but local `vercel dev` breaks without the explicit flag.

Minimum package.json shape:

```json
{
  "name": "<client-slug>-ops",
  "version": "0.2.0",
  "private": true,
  "type": "module",
  "dependencies": {
    "@octokit/rest": "^20.1.1"
  }
}
```

### 2. Create `/api/context.js`

This is the single endpoint that reads living docs from the repo, caches them, and returns a stitched KB. It's deliberately structured so future connectors slot in as siblings to `fetchGitHubDocs()`.

Key properties:

- Reads 6 docs in parallel via `Promise.all` + Octokit: `PROJECT_STATE.md`, `ARCHITECTURE.md`, `CONSTANTS.md`, `DECISIONS.md`, `GO_LIVE_CHECKLIST.md`, `DEMO_CONTEXT.md`
- 404s are skipped silently (so newer clients without `DEMO_CONTEXT.md` don't hard-fail)
- Each doc prepended with a `# === DOCUMENT: FILENAME.md ===` header so Claude can reason about which doc said what
- In-memory cache, 60s TTL, per-serverless-instance (fine — warm instances + 60s TTL stays well under GitHub's 5000 req/hr fine-grained PAT limit)
- Exports a `getContext()` function so `/api/claude.js` can call it in-process (no HTTP round-trip)
- Default export is an HTTP handler for debugging — returns `{ docsLoaded, kbLength, fromCache, fetchedAt }` metadata **without** the KB body to prevent accidental doc exposure via public URL

See the reference implementation section at the end of this skill for the canonical file set — clone `labsobsidian/hardesty-ops` at commit 7898647 to copy from. Templates directory will be populated in a future version once we've run this skill against a second client and extracted the client-agnostic shape.

### 3. Update `/api/claude.js`

Replace the existing proxy with one that:

- Accepts `{ role, messages }` as the request body (not `{ model, max_tokens, system, messages }` — the backend now owns model/tokens/system)
- Calls `getContext()` in-process to pull the live KB
- Builds the system prompt as: `[Atlas preamble with CLIENT_NAME] + [role preamble] + [fetched KB docs]`
- Has a `ROLE_PREAMBLES` object at the top of the file mapping role → short framing text
- **CRITICAL:** forces `stream: true` on every Anthropic call and pipes the SSE response through unchanged. The frontend depends on `content_block_delta` events; any refactor to buffered JSON will silently break chat.

Default role preambles for a typical SMB client:

- `owner` — focus on revenue, close rates, ROI, approval queue
- `ops` — focus on what needs doing now, scheduling, crew coordination
- `crew` — focus on TODAY'S JOBS only; no pipeline/revenue/approvals
- `admin` — Obsidian Labs full access; questions may be about the build itself

Add others as the client's org demands (e.g. `owner_marketing` for a dedicated marketing director).

### 4. Patch `index.html`

Two surgical changes:

- **Delete the `buildKB()` function** entirely. It's vestigial — the backend now owns this.
- **Change the fetch body** in the chat `run()` function:
  
  Before: `body: JSON.stringify({ model, max_tokens, system: buildKB(), messages: history })`
  
  After: `body: JSON.stringify({ role: currentRole, messages: history })`

Everything else (SSE parser, role detection from URL, UI tab gating) stays identical.

### 5. Create `DEMO_CONTEXT.md` in the client repo

Purpose: holds the rich operational narrative (specific pipeline, leads, schedule, financials) that makes Atlas sound knowledgeable during demos. Lives in the repo, read by `/api/context` alongside the 5 living docs.

Header must explicitly label it as demo content and mark it for deletion when live data sync is wired:

```markdown
# DEMO_CONTEXT.md — <Client Name>

**⚠️ DEMO CONTENT. Delete this file when live data sync is wired.**
```

Populate with realistic specifics the client will recognize: real names/places in their market, plausible pipeline values, today's schedule, marketing snapshot. Not aspirational data — believable operational state.

### 6. Create a fine-grained GitHub PAT

GitHub → avatar → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new.

- **Name:** `<client-slug>-atlas-brain`
- **Expiration:** 1 year. Calendar the rotation.
- **Resource owner:** `labsobsidian`
- **Repository access:** "Only select repositories" → **only** the client repo
- **Repository permissions:**
  - Contents: **Read-only** (for Phase B; upgrade to Read and Write when/if shipping Phase C's `/api/commit`)
  - Metadata: Read-only (auto-selected with Contents — do NOT deselect, the fetch silently fails without it)

Copy the token immediately — it's shown only once.

### 7. Set Vercel env vars

In the client's Vercel project → Settings → Environment Variables. Add these in **Production and Preview** (check both boxes per variable):

| Name | Value |
|---|---|
| `ANTHROPIC_API_KEY` | (usually already set) |
| `GITHUB_TOKEN` | the PAT from step 6 |
| `CLIENT_REPO` | `labsobsidian/<client-slug>-ops` |
| `CLIENT_NAME` | `<Client Name>` |
| `CLIENT_SLUG` | `<client-slug>` |

### 8. Commit, push, and redeploy

- Add a `.gitignore` to the client repo if there isn't one (must exclude `node_modules/`, `.env*`, `.vercel`, `.DS_Store`)
- Stage everything, commit with a `state:`-prefixed message describing the cutover
- Push to `main` — Vercel auto-builds
- Env var changes don't take effect on existing deploys. Go to Deployments → latest → ⋯ → **Redeploy** **without build cache** to pick up the new env vars

## Outputs

- `/api/context.js` endpoint returning a 20k+ character stitched KB from 6 living docs
- `/api/claude.js` serving a role-aware, doc-grounded chat experience with streaming intact
- Atlas UI that can be driven by URL params (`?role=`, `?email=`, `?name=`) with no hardcoded knowledge
- Client repo as the single source of truth — any `git push` to docs becomes new brain knowledge within 60s

## Verification

Run these four tests in order. Do not mark complete until all four pass.

**1. Context endpoint health**

Hit `https://<client-domain>/api/context` in a browser. Expect:

```json
{
  "client": "<Client Name>",
  "slug": "<slug>",
  "repo": "labsobsidian/<client>-ops",
  "docsLoaded": ["PROJECT_STATE.md", "ARCHITECTURE.md", "CONSTANTS.md", "DECISIONS.md", "GO_LIVE_CHECKLIST.md", "DEMO_CONTEXT.md"],
  "kbLength": 10000+,
  "fromCache": false,
  "fetchedAt": "<timestamp>"
}
```

Refresh within 60s — `fromCache` should flip to `true`.

**2. Chat pulls specific content from `DEMO_CONTEXT.md`**

On the main Atlas URL, ask a question whose answer exists only in `DEMO_CONTEXT.md` (e.g. a specific lead by name with specific quote amount). The response should name real specifics, not give generic industry advice. If it gives generic answers, the KB isn't making it into the system prompt — likely a role routing bug or a context fetch that returned empty.

**3. Cross-doc synthesis works**

Ask a question whose complete answer spans two or more living docs (e.g. "What's our follow-up sequence for a quote that hasn't gotten a response?" — draws from `PROJECT_STATE.md` workflow definitions plus `DEMO_CONTEXT.md` current leads). Response should weave both. This confirms the KB is being read as a whole, not just one doc at a time.

**4. Role switching changes behavior**

Append `?role=crew` to the URL and ask the same question from Test 2. Expect the crew role to decline or redirect — it's scoped to today's jobs only, not pipeline/quotes. If it answers happily, the role preamble isn't routing; the backend is likely using a default instead of the requested role.

## Known gotchas

- **Metadata permission on the PAT.** GitHub auto-selects Metadata: Read when you set Contents: Read, but if you ever manually deselect it, Octokit fetches silently return 404 for every file. The symptom is `docsLoaded: []` with no error. Always leave Metadata enabled.
- **`"type": "module"` in package.json.** Vercel's runtime handles ESM under `/api/` either way in production. But `vercel dev` locally throws a cryptic error without the explicit type declaration. Set it always; future-proofs against runtime default changes.
- **Env vars not taking effect after deploy.** Env var changes only apply to **new** deploys. If you add vars and the app still 500s, redeploy from the Deployments tab — and uncheck "Use existing Build Cache" to be safe.
- **`node_modules/` leaking into the commit.** First `npm install` in a repo without a `.gitignore` makes `git add -A` sweep in hundreds of files. Always create `.gitignore` with `node_modules/` before the first stage.
- **CRLF warnings on Windows.** Running the first commit on Windows will flag dozens of CRLF warnings. They are benign; Git normalizes line endings on commit. Ignore.
- **Per-request latency on cold starts.** First request after serverless instance cold-start + 6 parallel GitHub fetches = ~400–800ms before streaming begins. 60s cache covers subsequent requests. If a client complains about slowness, check whether they're hitting the app after long idle periods.
- **KB body exposure via debug endpoint.** The default `/api/context` handler returns metadata only, not the KB content. It's tempting to uncomment the `kb` field for debugging — do so only locally, never in deployed code. Living docs aren't secret but they shouldn't be trivially scrapable either.
- **Cache is per-serverless-instance.** If Vercel spawns 5 warm instances during load, each has its own 60s cache. Edits to living docs may take up to 60s to appear everywhere. Normally invisible; worth knowing for debugging.

## Planned extensions (future connectors)

The `buildKB()` assembler in `context.js` uses a `Promise.all` array. Additional data sources slot in as new fetch functions. Each becomes its own skill once implemented and proven against a real client:

| Connector | Skill (future) | What it adds |
|---|---|---|
| GHL pipeline | `atlas-ghl-connector` | Live contacts, pipeline stages, workflow status |
| Supabase live data | `atlas-supabase-queries` | Real-time queries against synced client data |
| AccuLynx sync | `atlas-acculynx-supabase-sync` | Nightly sync from AccuLynx to Supabase standard schema |
| Gmail threads | `atlas-gmail-connector` | Recent email context per contact |
| QuickBooks | `atlas-quickbooks-connector` | Invoicing, P&L, accounting truth |

Each connector should return `{ text: string, loaded: string[] }` from its fetch function so the assembler stays uniform.

## Related skills

- `new-client-spinup` — creates the client repo with living docs (runs before this)
- `project-state-update` — keeps living docs current so Atlas stays smart (runs throughout)
- `atlas-commit-endpoint` (Phase C, not yet drafted) — adds `/api/commit` so Atlas can write back to the repo

## Reference implementation

The canonical implementation lives in `labsobsidian/hardesty-ops` at commit `7898647`. Diff that commit against its parent to see the exact file set:

- `api/context.js` (new)
- `api/claude.js` (modified — simplified signature, added role preambles)
- `index.html` (modified — `buildKB` removed, fetch body shrunk)
- `package.json` (new)
- `package-lock.json` (new, generated)
- `.gitignore` (new)
- `DEMO_CONTEXT.md` (new)
- `README.md` (expanded)

Copy from there when running this skill for a new client — change the client-specific values and re-deploy.
