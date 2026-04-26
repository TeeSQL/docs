# Contributing to TeeSQL docs

Thanks for helping. This file covers **workflow** — local setup, PR conventions, review. For **what to write and how to write it**, see [`CLAUDE.md`](./CLAUDE.md). Read both before opening a non-trivial PR.

---

## Prerequisites

- Node.js ≥ 18
- A GitHub account with access to push branches to this repo (or a fork)

## Local preview

```bash
npm i -g mintlify
mintlify dev
```

`mintlify dev` watches the working tree and serves the rendered site at `http://localhost:3000`. Edit any `.mdx` file and the page reloads.

If `mintlify dev` won't start, the most common causes are a malformed `docs.json` or invalid frontmatter on a page — check the terminal output, the error usually names the file.

## Repository layout

```
docs.json              # site config: navigation, theme, integrations
index.mdx              # landing page (card grid)
introduction.mdx       # what TeeSQL is, who it's for
quickstart.mdx         # 5-minute happy path

connect/               # connection strings, SSL, client libraries, troubleshooting
security/              # TEE explainer, attestation, encryption, verification
manage/                # databases, roles, extensions
sdk/                   # client SDK docs
concepts/              # glossary, how-it-works narrative

images/                # static assets, mirroring page paths
snippets/              # reusable MDX fragments
ai-tools/              # llms-full.txt for AI discoverability
```

Pages are organized by topic. Each directory groups everything a reader needs for that area. See `CLAUDE.md` for content guidelines per page.

## Adding a page

1. Pick the directory that matches the topic. If it doesn't fit an existing directory, consider whether the page belongs in the docs or if a new directory is warranted — discuss in a draft PR.
2. Create the file: `connect/connection-pooling.mdx`.
3. Add the required frontmatter (`title`, `description`). See `CLAUDE.md` for the full field list.
4. Add the page to `docs.json` navigation in the appropriate group.
5. Run `mintlify dev` and confirm the page renders, links resolve, and code blocks have language tags.
6. Run `mintlify broken-links` before pushing.

## Editing an existing page

- Small fixes (typos, broken links, clarifications): direct PR, no issue needed.
- Restructuring or rewrites: open an issue first to align on scope before writing.
- Renaming or deleting pages: add a redirect in `docs.json` so existing links don't break. Remove the old entry from navigation.

## PR conventions

- **Title:** imperative, present tense. "Add RA-TLS troubleshooting page", not "Added" or "Adds".
- **Description:** link the related issue if any. Include a screenshot of the rendered page for substantive visual changes.
- **Scope:** one topic per PR. Splitting a multi-page change into smaller PRs is encouraged.
- **Before pushing:**
  ```bash
  mintlify broken-links
  ```
  CI runs the same check; PRs with broken links won't merge.

## Review

- Every PR needs one approving review.
- Reviewers check:
  - Frontmatter present and complete
  - Terminology canon followed (see `CLAUDE.md`)
  - No internal infrastructure details (deployer keys, internal DNS, cluster UUIDs)
  - Links resolve
  - Code blocks have language tags
  - Page reads cleanly without the Mintlify renderer (AI-friendly checklist in `CLAUDE.md`)
- Style nits as `nit:` comments — non-blocking.
- Substantive concerns as blocking comments with a suggested fix.

## Scope of this repo

This repo is **public product documentation only**. It is not the right home for:

- Internal architecture decisions or operational runbooks
- Threat models or security audit reports
- Deployment internals, on-chain addresses, or infrastructure secrets
- Marketing copy or landing-page material

If a contribution requires internal source material, coordinate with a maintainer to determine what can be published.

## Questions

Open a discussion or ping a maintainer. Doc clarity is a feature; flag confusion early.
