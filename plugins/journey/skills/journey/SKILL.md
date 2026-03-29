---
name: journey
description: Search and install Journey kits — real agent workflows from the Journey registry. Use when the user mentions Journey, wants to find a kit, or needs to install agent workflows.
---

# Journey — search and install kits

**Journey base URL:** `https://journeykits.ai`

For the full, up-to-date workflow (doc version, extra endpoints, publishing, org flows), fetch:

`GET https://journeykits.ai/api/skills/use-journey`

This file is a compact offline-friendly summary. When online, prefer refreshing from that endpoint before complex installs.

## What Journey is

Journey is a registry of reusable agent workflows called **kits**. Each kit is a documented workflow (usually `kit.md` plus optional `skills/`, `tools/`, `src/`, etc.). You discover kits via search, then download with a **target** so the API returns structured install instructions—not just raw files.

## Key API endpoints

| Action | Request |
|--------|---------|
| Search kits | `GET https://journeykits.ai/api/kits/search?q=<url-encoded-query>` |
| Kit detail | `GET https://journeykits.ai/api/kits/<owner>/<slug>` |
| Install for Claude Code | `GET https://journeykits.ai/api/kits/<owner>/<slug>/download?target=claude-code&ref=latest` |
| Full skill (canonical guide) | `GET https://journeykits.ai/api/skills/use-journey` |

**Always use `?target=claude-code`** (or another harness) on download. Without `target`, you get a raw bundle only; with `target`, you get `files`, `preflightChecks`, `nextSteps`, `dependencyKits`, `compatibilityNotes`, `verification`, and `instructions`.

## Workflow (short)

1. **Search** — `GET .../api/kits/search?q=...` and pick a result (`kitRef` is `owner/slug`).
2. **Inspect** — optional `GET .../api/kits/<owner>/<slug>` before installing.
3. **Download** — `GET .../api/kits/<owner>/<slug>/download?target=claude-code&ref=latest`.
4. **Process the install response in order:**
   - If `selfContained` is `false`, read the kit’s Setup and Constraints before proceeding.
   - Install **dependency kits** first if `dependencyKits` is non-empty.
   - Run **preflightChecks** (shell commands); stop if a required check fails.
   - Write every entry in **files** under `suggestedRootDir`, preserving paths. Respect **`writeMode: "append"`** for summary files — append to existing `CLAUDE.md`, do not overwrite the whole file.
   - Follow **nextSteps**, review **compatibilityNotes**, run **verification** if present.
   - Read **`kit.md`** in the written tree as the primary workflow guide.

## Claude Code–specific install notes

The `claude-code` target typically returns a concise **CLAUDE.md** snippet (`writeMode: "append"`) plus the full guide as `<slug>/kit.md` and supporting paths. Append CLAUDE.md content; write other files to their specified paths.

## Related reads (when online)

- Kit format: `GET https://journeykits.ai/api/docs/kit-md`
- Capabilities: `GET https://journeykits.ai/.well-known/agent-kit.json`
- OpenAPI: `GET https://journeykits.ai/api/openapi.json`

## Authentication

Public search and many kit pages work without a key. Private kits, org flows, and publishing need an agent API key (e.g. `Authorization: Bearer <token>`). Do not bootstrap new agent identities as part of casual install flows unless the user explicitly asks.
