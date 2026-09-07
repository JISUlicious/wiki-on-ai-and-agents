# Wiki Lint Operation

You are performing a health check on the wiki. Scan all pages and report issues.

## Pre-flight

0. Read `.wiki-config.json` if it exists at the project root. Resolve `{WIKI}` per the **Path Resolution** rules in SKILL.md. If no config, use `wiki/` relative to project root. Always double-quote paths containing spaces in Bash commands.
1. Read `CLAUDE.md` (at project root) for domain-specific conventions
2. Read `{WIKI}/index.md`
3. Glob all markdown files in `{WIKI}/**/*.md`

## Checks to Perform

### 1. Broken Wikilinks (Error)

For every `[[wikilink]]` found in wiki pages:
- Resolve the link to a file: search for `{{link-name}}.md` anywhere under `wiki/`
- If no matching file exists, report as a broken link

**How to scan**: Use Grep to find all `\[\[.*?\]\]` patterns across `wiki/**/*.md`. For each unique link target, use Glob to check if the file exists.

### 2. Orphan Pages (Warning)

Pages with no inbound `[[wikilinks]]` from other pages. Exclude these from orphan detection:
- `wiki/index.md`
- `wiki/log.md`
- `wiki/overview.md`

**How to scan**: For each wiki page, Grep all other wiki pages for `[[page-name]]`. If no other page links to it, it's an orphan.

### 3. Missing Pages (Suggestion)

Concepts or entities mentioned prominently in wiki pages that don't have their own page yet. Look for:
- Repeated mentions of the same term across multiple pages without a corresponding `[[wikilink]]`
- `[[wikilinks]]` that point to non-existent pages (overlaps with broken links, but the suggestion here is to CREATE the page rather than fix the link)

### 4. Index Consistency (Error)

- Pages that exist in `wiki/` subdirectories but are missing from `wiki/index.md`
- Entries in `wiki/index.md` that point to pages that don't exist
- Pages filed in the wrong category (e.g., an entity page listed under Concepts)

### 5. Frontmatter Validation (Warning)

For each wiki page (excluding index.md and log.md), check:
- `title` field exists and is non-empty
- `type` field exists and is one of: entity, concept, source, comparison, query, overview
- `created` field exists and is a valid date
- `updated` field exists and is a valid date
- `sources` field exists (can be empty list for overview)
- `status` field exists and is one of: draft, complete, needs-update
- `importance` field exists and is one of: high, medium, low

### 6. Stale Content (Suggestion)

- Pages whose `updated` date is significantly older than the most recent ingest in `wiki/log.md`, AND whose topic has been touched by newer sources
- Source summary pages that reference source files no longer in `sources/`

### 6b. Unresolved Contradictions (Warning)

Scan all wiki pages for `> [!warning] Contradiction` callouts. These are added during ingest when new information conflicts with existing content. Report each one with:
- The page containing the contradiction
- A brief summary of the two conflicting claims
- Suggest resolution: which source is more authoritative, more recent, or more widely cited?

**How to scan**: Grep all `{WIKI}/**/*.md` for the pattern `\[!warning\] Contradiction`.

### 7. Cross-Reference Gaps (Suggestion)

- Entity/concept pages that mention other entities/concepts by name without using `[[wikilinks]]`
- Pages that discuss related topics but don't link to each other

**How to scan**: For each entity and concept page name, Grep all other wiki pages for that name (case-insensitive). Flag mentions that aren't wrapped in `[[...]]`.

### 8. Knowledge Gaps (Suggestion)

Identify areas where the wiki's coverage is thin and suggest external sources to look for:

- **Draft pages**: Pages with `status: draft` that have minimal content. Suggest a specific search query to enrich them (e.g., `search "PaLM 2 technical report Google 2023"` to fill out [[palm]]).
- **Single-source concepts**: Concept pages whose `sources` list has only one entry. Suggest finding additional sources for triangulation and broader perspective.
- **Stale topics**: Pages where the most recent source is significantly older than the current date. Suggest searching for recent developments.
- **Mentioned but unexplored**: Topics that appear across multiple pages but have no dedicated page AND no source directly covering them. Suggest a search query.

**How to scan**: Read frontmatter of all wiki pages. For draft pages and single-source pages, generate a concrete search query string the user could paste into Google Scholar, Semantic Scholar, or a search engine.

### 9. Tag Hygiene (Error/Warning/Suggestion)

Tags should be a thin organizational layer per SKILL.md Tag Policy. Scan all wiki pages and flag:

**Error — entity classification tag missing/invalid.** Every `type: entity` page MUST carry **exactly one** tag from the entity classification vocabulary declared in `CLAUDE.md` (per SKILL.md → Tag Policy → "Entity classification tags"). Report per-page violations:
- Zero classification tags on an entity page → `entity X missing required classification tag (allowed: <vocabulary>)`
- Multiple classification tags on an entity page → `entity X has multiple classification tags {a, b}; keep exactly one`
- A tag that looks like a classification (e.g., `model`, `person`) but isn't in the declared vocabulary → `entity X uses classification-like tag Y not in CLAUDE.md vocabulary; either add Y to the vocabulary or use an allowed tag`

**How to find the vocabulary**: Read `CLAUDE.md` and look for the "Entity classification vocabulary" table. The first column gives the allowed values.

**Warning — tag duplicates a page slug.** A tag value that matches an existing page filename. Report each occurrence as `page X has tag Y which duplicates [[Y]] — drop the tag and link the page in the body if not already linked`. *Classification tags are exempt — they are taxonomy markers and don't have matching pages.*

**Warning — over-tagged pages.** Pages with more than 3 tags. Suggest which tags to drop based on the Tag Policy anti-patterns. *The entity classification tag must not be dropped to stay under the limit; remove organizational tags first.*

**Suggestion — promotion candidates.** Non-classification tags appearing on ≥3 pages with no corresponding page. Report top 10 by frequency:
```
- Promote tag `<tag-name>` (on N pages: [[page-a]], [[page-b]], ...) — create `wiki/concepts/<tag-name>.md` (or `wiki/entities/` if it's an entity).
```

**How to scan**:
1. Glob `{WIKI}/**/*.md`, parse frontmatter, collect (page → tags, type) tuples.
2. Read `CLAUDE.md` to obtain the entity classification vocabulary (parse the table).
3. Compute tag frequencies and the set of existing page slugs.
4. For each `type: entity` page: check that exactly one tag is in the vocabulary.
5. For each non-classification tag: check duplicates (vs slugs), check page count (vs promotion threshold), check per-page count (vs max=3).

### 10. Typed Relation Hygiene (Error/Warning)

Relation-bearing frontmatter fields (e.g., `authors`, `founders`, `affiliations`, `trained-on`, `builds-on`, `components`, `introduces`, `predecessor`, `successors`, `family`, plus any others declared in `CLAUDE.md`) must follow the conventions in SKILL.md → "Typed Relations".

**Identifying relation fields.** A field is a relation field if either (a) its name appears in CLAUDE.md's relation vocabulary or in SKILL.md's default fallback list, **or** (b) the field's value contains one or more `[[...]]` wikilink strings. Scalar metadata fields (`title`, `type`, `created`, `updated`, `tags`, `status`, `importance`, `arxiv_id`, `venue`, `parameters`, `release_date`, `year`, `family` when string-valued like `gpt`) are NOT relation fields.

**Error — relation value not wikilink-shaped.** A relation field has a value that should be a wikilink but isn't. Examples:
- `authors: - Tom B. Brown` (bare string) → should be `[[tom-brown]]` if an entity page exists, or stay string only if no page has been created yet. Flag as Error when an entity page with the matching slug *does* exist (the page exists but the field doesn't link to it).
- `predecessor: gpt-2` (bare string, no brackets) → must be `[[gpt-2]]`.

**Error — relation wikilink target doesn't resolve.** Same diagnostic as §1 Broken Wikilinks, but applied to frontmatter values. Report once per (page, field, target) — do not double-count with §1.

**Warning — dict-form relation missing `target:` (or equivalent).** When a relation uses the dict form, each item must have a key naming the related page:
- Generic case: `target:` key.
- Field-specific aliases recognized: `org:` (for `affiliations:`), `model:` (where unambiguous in context). The project's `CLAUDE.md` may declare additional aliases.

Items missing both `target:` and a recognized alias are flagged.

**Suggestion — relation field missing on a page where it would apply.** Heuristic: if an entity is classified as `person` (per §9) and has a body section titled "Affiliations" or "Notable publications", but the frontmatter lacks the corresponding `affiliations:` or `authored:` field, suggest promoting the prose to typed frontmatter. Similarly for `founders:` on org pages with body text like "founded by X and Y", and `authors:` on source pages without a frontmatter authors list. Report as a suggestion, not an error — the prose is still valid; the typed form just enables Dataview queries.

**How to scan**:
1. Glob `{WIKI}/**/*.md`, parse frontmatter.
2. Read `CLAUDE.md` for the project's relation vocabulary (extends or overrides SKILL.md defaults).
3. For each parsed frontmatter, identify relation fields per the rules above.
4. Validate each relation field's values: wikilink shape, target resolution, dict-form keys.
5. For each entity/source page, check the suggested fields against body section headings.

### 11. Markdown Rendering Hygiene (Error)

Constructs that are valid as *text* but break as *markdown*. These are invisible to §1 and §10 because the wikilink itself is well-formed — only the rendering is wrong — so they need their own pass.

**Error — unescaped pipe inside a table cell.** A `[[slug|alias]]` wikilink in a table row renders its pipe as a **column separator**. The row gains a phantom column, the table goes ragged, and the link shows as literal text. Obsidian, GitHub and most renderers all break the same way.

```markdown
| [[mem0|Mem0]] | 94.4 |          <!-- WRONG: renders as three columns -->
| [[mem0\|Mem0]] | 94.4 |         <!-- RIGHT -->
```

Writing `[[slug|Alias]]` is correct everywhere *except* inside a table, which is why it keeps getting through.

**Error — double-escaped pipe.** `[[slug\\|alias]]` renders a literal backslash. Any run of two or more backslashes before the pipe is wrong. **This is the failure mode of a careless fix, not of authoring** — see the fix procedure below.

**Error — ragged table.** Any table whose rows disagree on separator count, counting only pipes *not* preceded by a backslash. Usually a symptom of the above, but also catches a hand-edited row with a missing or extra `|`.

**Warning — unbalanced code fence.** An odd number of ` ``` ` in a file. Everything after the unclosed fence renders as code, silently hiding content and defeating link resolution for the rest of the page.

**How to scan.** Glob `{WIKI}/**/*.md`. Track fenced-code state line by line and **skip fenced regions entirely** — a table inside a code block is illustrative, not real. Then, for lines whose first non-space character is `|`:

| Defect | Pattern |
|---|---|
| double-escaped | `\[\[[^\]]*?\\{2,}\|` |
| unescaped | `\[\[[^\]]*?(?<!\\)\|` |
| column count | number of matches of `(?<!\\)\|` |

Test double-escaped **before** unescaped, since the double case also matches the unescaped pattern. Flag a table block when its rows produce more than one distinct column count. Separately, flag any file with an odd count of ` ``` `.

**How to fix.** Only the two pipe errors are auto-fixable. Apply per table-cell line, outside fences:

1. **Collapse first**: replace `(\[\[[^\]]*?)\\{2,}\|` with `\1\|`, repeating until the line stops changing.
2. **Then escape**: replace `(\[\[[^\]\|]*?)(?<!\\)\|` with `\1\|`.

> [!warning] Do not skip step 1, and do not reorder these
> A naive "escape every pipe in a wikilink" pass **will corrupt links that were already correct**, turning `\|` into `\\|`. This is not hypothetical: a 2026-09 repair pass did exactly that to 95 already-correct links before the verification step caught it. The `[^\]\|]` in step 2's first group and the `(?<!\\)` lookbehind are both load-bearing — they make the operation **idempotent**, so running it twice is safe.

After fixing, re-scan and confirm three things: zero unescaped, **zero double-escaped**, and zero ragged tables. Verifying only the first will miss damage you just caused.

**Never auto-fix** ragged tables or unbalanced fences — the correct repair depends on which column or fence the author intended.

> [!important] Why this check exists
> This is the one defect class the wiki has repeatedly reintroduced. Ingest agents write `[[slug|Alias]]` correctly in prose and carry the habit into comparison tables. A 2026-09 sweep found **53 instances across 13 files**, some months old — none visible to any other check, because every link resolved fine.

## Report Format

Group findings by severity:

```markdown
## Lint Report — {{today}}

### Errors (must fix)
- [ ] Broken link: `[[nonexistent-page]]` in wiki/concepts/foo.md:15
- [ ] Index missing: wiki/entities/bar.md not in index.md

### Warnings (should fix)
- [ ] Orphan page: wiki/entities/baz.md (no inbound links)
- [ ] Missing frontmatter: wiki/sources/qux.md missing `type` field

### Suggestions (nice to have)
- [ ] Missing page: "reinforcement learning" mentioned in 3 pages but has no wiki page
- [ ] Cross-ref gap: wiki/entities/alice.md mentions "Bob" without [[wikilink]]
- [ ] Stale content: wiki/concepts/old-topic.md not updated since 2 new sources ingested
- [ ] Knowledge gap: [[palm]] is status:draft with 1 source — search "PaLM 2 technical report Google 2023"
- [ ] Single source: [[some-concept]] could use more perspectives — search "concept name survey 2024"
```

## Auto-Fix

After presenting the report, ask the user:

> "Would you like me to auto-fix any of these? I can:
> 1. Fix all errors (broken links, index sync)
> 2. Fix errors + warnings (add missing frontmatter, link orphans)
> 3. Fix everything including suggestions (create missing pages, add cross-references)
> 4. Let me pick specific items"

For auto-fix:
- **Broken links**: If the target page clearly should exist, create a stub page. If it's a typo, fix the link.
- **Index sync**: Add missing pages to the index with a generated one-line description.
- **Orphan pages**: Add links from the most relevant related pages.
- **Missing frontmatter**: Generate reasonable defaults based on file location and content.
- **Missing pages**: Create stub pages with basic content derived from how the concept is discussed in existing pages.
- **Cross-ref gaps**: Add `[[wikilinks]]` around unlinked mentions.
- **Tag promotion**: For each promotion candidate, create a stub concept/entity page derived from contexts where the tag appears (Grep the tagged pages, extract a short description). Remove the tag from those pages and replace mentions with `[[wikilink]]` where appropriate.
- **Tag-as-page duplicates**: Remove the duplicating tag; ensure the body links the page.
- **Over-tagged pages**: Drop the most page-worthy tags by promoting them (see above); keep at most 3 purely organizational tags.
- **Rendering hygiene**: Apply §11's two-step fix (collapse double-escapes, then escape unescaped), in that order, then re-scan for *both* error types. Ragged tables and unbalanced fences are reported but **never auto-fixed**.

## Log

After lint completes, append to `wiki/log.md`:

```markdown
## [{{today}}] lint | Wiki health check

Found: {{N}} errors, {{N}} warnings, {{N}} suggestions. {{Auto-fixed X items | No auto-fix applied.}}
```
