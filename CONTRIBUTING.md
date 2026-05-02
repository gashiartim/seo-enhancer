# Contributing

## Maintainer checklist — editing policy or content

Run through this before opening a PR that changes audit rules, wording, or examples.

### Source of truth

`SKILL.md` is the canonical policy file. All other files derive from it.

When you change a rule in `SKILL.md`, update the same rule in every file it appears in:

| Changed in | Also update |
|------------|-------------|
| `SKILL.md` | `INSTRUCTIONS.md`, all `adapters/`, affected `references/` |
| `adapters/*.md` or `adapters/*.mdc` | Verify it matches `SKILL.md` — do not introduce new rules |
| `references/*.md` | `SKILL.md` patterns section if the implementation changes |

### Policy accuracy checklist

Before merging any change to audit rules or wording:

- [ ] **Severity is correct** — Critical = genuine indexing risk (missing title/description with no parent coverage, missing alt on informational images, hreflang absent on i18n site). Enhancement = show + ask.
- [ ] **robots.txt absence is Enhancement** — crawl is allowed by default; absence is not a blocker.
- [ ] **`next/image` is Enhancement** — performance improvement, not an SEO hard requirement.
- [ ] **`next/head` in app router is Enhancement** — native metadata API is preferred, not required.
- [ ] **JSON-LD is Enhancement** — and always goes in the page component `<script>` tag, not in `generateMetadata`.
- [ ] **SPA rendering risk uses "deferred"** — Google does execute JavaScript but with deferred timing and limited resources. Do not say "Google may not execute JS."
- [ ] **No `rel=next/prev`** — deprecated by Google in 2019. Use self-referencing canonicals on paginated pages instead.
- [ ] **Remix — inherited coverage** — a missing local `meta` export is only Critical when the route has no effective metadata coverage from root defaults.
- [ ] **i18n wording uses "locale disambiguation"** — not "duplicate content penalty." The real risk is Google misclustering locale variants and serving the wrong regional page.
- [ ] **Alt guidance is precise** — informational images need descriptive alt. Decorative images use `alt=""` (empty string, not missing). Do not flag `alt=""` as a problem.
- [ ] **Parent layout inheritance** — a Next.js page missing a local `metadata` export is only Critical if no ancestor layout provides coverage.

### Adapter consistency

After editing, verify each adapter file still matches `SKILL.md`:

```
adapters/cursor.mdc
adapters/copilot.md
adapters/windsurf.md
adapters/cline.md
adapters/agents.md
```

Key things to check in each:
- Trigger globs/conditions do not include generic component directories (`src/components/`)
- robots.txt listed under Enhancement, not Critical
- `next/image` listed under Enhancement with "performance improvement, not SEO requirement" note
- JSON-LD note: page component only, not `generateMetadata`
- i18n wording: "locale disambiguation" not "duplicate content penalty"
