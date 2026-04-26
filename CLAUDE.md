# Writing docs for TeeSQL

This file is the canonical content guide for the TeeSQL docs repo. Read by both human contributors and AI assistants. If a rule here conflicts with something elsewhere, this file wins for content; `CONTRIBUTING.md` wins for workflow.

TeeSQL is a TEE-hosted PostgreSQL DBaaS — Postgres 17 running inside Intel TDX confidential VMs orchestrated by [dstack](https://github.com/Dstack-TEE/dstack), with hardware attestation, encrypted storage, and managed replication. Docs are built with [Mintlify](https://mintlify.com).

---

## Audience

Two readers, equally important:

1. **TEE app developers** — Solidity/Rust/TypeScript engineers shipping confidential apps who need a real database. They know Postgres. They are new to RA-TLS, attestation, and CVMs.
2. **AI assistants** — Claude, Cursor, ChatGPT, etc., ingesting `/llms.txt`, `/llms-full.txt`, and per-page `.md`. They have no visual context and no prior session memory.

Every page must serve both. If a page only makes sense with the rendered Mintlify components, it will fail for AI ingestion.

---

## Site structure

Pages are organized by **topic**, not by content type. Each directory groups everything a reader needs for that area — connecting, securing, managing, etc.

```
index.mdx                  # landing page (card grid, no prose)
introduction.mdx           # what TeeSQL is, who it's for
quickstart.mdx             # 5-minute happy path

connect/                   # connection strings, SSL, clients, troubleshooting
security/                  # TEE explainer, attestation, encryption, verification
manage/                    # databases, roles, extensions
sdk/                       # client SDK docs
concepts/                  # glossary, how-it-works narrative

snippets/                  # reusable MDX fragments
images/                    # static assets
```

Within each directory, keep pages focused. A single page should answer one question or walk through one task. If a page drifts into both "how to do X" and "why X works this way," split it into two pages with a cross-link.

---

## Page structure

Every `.mdx` page follows this skeleton:

```mdx
---
title: "Connect from a CVM"
description: "Open a mutually-attested RA-TLS connection from your dstack app to a TeeSQL cluster."
icon: "plug"
keywords: ["ra-tls", "client", "cvm", "connect"]
---

One-sentence summary that restates the title in active voice. An LLM seeing only this paragraph should know what the page covers.

## Prerequisites

- Bullet list of concrete preconditions
- Use links for anything not assumed common knowledge

## Main content

(Steps, explanation, reference tables — whatever this page is)

## Verify

How the reader confirms it worked. Include this on any page with a procedure.

## Related

- [Encryption](/security/encryption)
- [TypeScript SDK](/sdk/typescript)
```

---

## Frontmatter

Required on every page:

| Field | Purpose |
|---|---|
| `title` | H1 + browser tab + sidebar default. ≤60 chars. Sentence case. |
| `description` | One sentence, ≤160 chars. Used for SEO, social cards, AND llms.txt. Treat as the page's TL;DR for an LLM. |

Use when applicable:

| Field | Use it when |
|---|---|
| `sidebarTitle` | Title is long; pick a short label for nav |
| `icon` | Page is a top-level entry in a section. Use [Font Awesome](https://fontawesome.com/icons) names. |
| `keywords` | Domain terms users might search but don't appear in the title (e.g. `["mtls", "attestation"]` on an RA-TLS page) |
| `tag` | "Beta", "Deprecated", "Experimental" — short status |
| `mode: "wide"` | Page has wide tables or diagrams |

Do not set `hidden: true` to "soft delete" — delete the file and remove it from `docs.json`.

---

## Voice

- **Second person, present tense.** "You connect" not "Users connect" or "We will connect".
- **Active voice.** "TeeSQL verifies the quote" not "The quote is verified".
- **Concrete > abstract.** "RA-TLS rejects the handshake" not "the connection fails".
- **Show, then tell.** Code or diagram first, prose second. AI assistants and skim-readers both benefit.
- **No marketing.** No "powerful", "seamless", "robust", "blazing-fast". State what it does.
- **No hedging.** "May", "should", "typically" only when the behavior is genuinely conditional. Otherwise commit.
- **No future tense for shipped behavior.** "TeeSQL will encrypt" → "TeeSQL encrypts". Future tense is reserved for roadmap items.
- **Define on first use.** First mention of an acronym gets expanded: "Confidential Virtual Machine (CVM)". After that, the acronym alone.

---

## Terminology canon

These are non-negotiable. Search/replace before merging.

| Use | Not |
|---|---|
| TeeSQL | teesql, TeeSql, TEE-SQL, tee-sql |
| Postgres (in prose) | postgres, postgresql, PostgresQL |
| PostgreSQL (in formal/version contexts) | e.g. "PostgreSQL 17 release notes" |
| dstack | DStack, Dstack, DSTACK |
| CVM | confidential VM, cVM (after first expansion) |
| RA-TLS | RaTLS, ra-tls (in prose), Remote Attestation TLS |
| Intel TDX | TDX, Intel-TDX, Intel Trust Domain Extensions (after first expansion) |
| attestation quote | quote, TDX quote (acceptable on first reference) |
| KMS | kms, Key Management Service (after first expansion) |
| sidecar | side-car, side car |

---

## Code blocks

- **Always specify the language.** ` ```python ` not ` ``` `. LLMs use the language tag to decide what to do with the snippet; bare blocks confuse them.
- **Minimal complete examples.** A snippet should run as-is given the prerequisites. No `...` placeholders unless you immediately follow with the elided context.
- **Use `<CodeGroup>` for multi-language equivalents** of the same operation, not for unrelated examples. Every block inside `<CodeGroup>` must have a filename.
- **Show output where it clarifies.** Use a second block tagged `text` or `console`.
- **Use realistic but obviously synthetic values.** `cluster-abc123`, `your-cluster.teesql.com`, `your-password`. Avoid `<angle-brackets>` — Mintlify renders them badly and they break copy-paste.
- **Comments in code, not after the block.** A reader scanning code shouldn't need the surrounding prose.

```python
# Good: realistic, runnable, language-tagged
import psycopg
import teesql_client  # provides ra_tls_ssl_context()

ctx = teesql_client.ra_tls_ssl_context(cluster="your-cluster")
conn = psycopg.connect(
    "host=your-cluster.teesql.com port=5433 dbname=app",
    sslmode="require",
    ssl_context=ctx,
)
```

---

## Links

- **Internal links are root-relative paths without `.mdx`.** `/concepts/glossary` not `./glossary.mdx` or `https://docs.teesql.com/concepts/glossary`.
- **External links use full URLs and meaningful link text.** `[dstack repo](https://github.com/Dstack-TEE/dstack)` not "click [here](...)" and not bare URLs.
- **Cross-link liberally.** A how-to should link to the concept it relies on; a concept should link to the how-to that applies it. This is what makes the doc set navigable for both humans and LLMs.
- **Never duplicate content across pages.** If two pages need the same fact, one owns it and the other links. For repeated code patterns, use a snippet from `snippets/`.

---

## MDX components — when to reach for each

| Component | Use it for |
|---|---|
| `<Steps>` / `<Step>` | Ordered procedures. Never for a list of options. |
| `<Tabs>` / `<Tab>` | Same task, different language/platform (Python vs TS, Linux vs macOS). |
| `<CodeGroup>` | Same code in multiple languages. Prefer over `<Tabs>` for code-only. |
| `<Card>` / `<CardGroup>` | Section landing pages — link to children. Not for inline emphasis. |
| `<Note>` | Side info that's safe to skip. |
| `<Tip>` | Recommended-but-not-required practice. |
| `<Warning>` | Footguns. Data loss, security, irreversible operations. Use sparingly so it stays loud. |
| `<Info>` | Version requirements, prerequisites callouts. |
| `<Accordion>` / `<AccordionGroup>` | FAQ-style or rarely-needed details. Hides content from skim readers — confirm that's the intent. |
| `<Frame>` | Wrap an image or diagram with a caption. |
| `<Snippet>` | Reusable content from `snippets/`. `<Snippet file="connection-string.mdx" />` |
| `<ParamField>` | SDK/API parameter documentation. |

No imports needed — all components are globally available in Mintlify.

Components flatten to plain markdown in `/llms-full.txt`, so an LLM reads `<Warning>` content as a normal paragraph. Don't put critical info *only* inside a callout — the surrounding prose must still make sense without it.

---

## Diagrams and images

- Prefer Mermaid diagrams or ASCII art over images for architecture. They render in `.md`, are diff-able, and LLMs read them as text. Use ` ```mermaid ` code blocks.
- For raster images, use SVG when possible. PNG fallback is fine.
- Every image has alt text. The alt text is the LLM's only access to it — describe the content, not the file.
- Store under `images/` mirroring the page path: `images/security/trust-boundary.svg`.

---

## What not to write

- **Speculation.** Document shipped behavior. Roadmap belongs on a separate, clearly-labeled page.
- **Why a competitor is worse.** Compare to alternatives only when the user has to choose (e.g. "vs. self-hosted Postgres") and stick to facts.
- **Internal infrastructure details.** No deployer keys, internal DNS labels, cluster UUIDs, or operational details. Public docs describe the product from the user's perspective.
- **Time-sensitive claims.** "Currently the fastest" — drop or cite a benchmark.
- **Apologies, opinions, signatures.** "We think this is great" — delete.

---

## AI-friendly checklist

Before merging, verify the page reads cleanly without the Mintlify renderer:

- [ ] `description` is a true one-sentence summary; reading only it, you know the page's purpose.
- [ ] First paragraph restates the page's purpose in plain prose (not only a callout).
- [ ] Every code block has a language tag.
- [ ] Every image has descriptive alt text.
- [ ] Headings nested correctly (one H1 from frontmatter, then H2 → H3, no skipped levels).
- [ ] Acronyms expanded on first use.
- [ ] No content lives *only* inside `<Accordion>` or `<Tabs>` that an LLM might miss in a flat read.
- [ ] Internal links use root-relative paths without `.mdx`.

---

## Workflow

For setup, local preview, PR conventions, and review process see [`CONTRIBUTING.md`](./CONTRIBUTING.md).
