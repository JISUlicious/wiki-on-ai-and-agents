# Newsletter-Ingest Runbook (weekly)

Deterministic procedure for the recurring "read newsletters → ingest new papers" task.
State lives in `newsletter-ingest-state.json` (Gmail MCP is **read-only**, so processed
threads are tracked here, not via Gmail labels).

**Senders:** `contact@alphaxiv.org`, `nlpnews@substack.com`

## Steps

1. **Gate on cadence.** Read `newsletter-ingest-state.json`. If `today − last_processed_date < 7 days`,
   stop (nothing to do yet). Otherwise continue.

2. **Read new mail.** For each sender, `mcp__claude_ai_Gmail__search_threads` with
   `from:<sender> in:inbox after:<last_processed_date>`. Collect thread IDs not already in
   `processed_thread_ids`. (Gmail auth only exists in an interactive claude.ai session — see Caveat.)

3. **Extract candidate arXiv IDs.** `get_thread` (FULL_CONTENT) per new thread; pull arXiv IDs.
   - alphaXiv digests expose IDs directly (reliable).
   - nlpnews "Top AI Papers" mask IDs behind Substack redirects → resolve + verify.
   - nlpnews "AI Agents Weekly" are paywalled previews → usually no IDs in body; low yield.

4. **Verify every ID** resolves at `arxiv.org/abs/{id}` before any download. Never fabricate IDs.
   Network note (this sandbox): only **foreground, low-concurrency** fetches work for the main agent;
   **subagents** have reliable sequential network. Background bash jobs lose network. Pace curls ~1s,
   no parallelism inside a single agent.

5. **Dedupe vs wiki.** Skip IDs whose paper already has a `wiki/sources/*.md` page (check arxiv_id).

6. **Domain-triage.** Keep on-domain papers (LLMs, agents, ML/AI per `CLAUDE.md`); drop off-domain.

7. **Ingest** each kept paper as a concise (~40–60 line) source page via parallel subagents:
   - download PDF → `pdftotext` → `sources/<slug>.md` (prepend `<!-- Source URL … | Fetched: <date> -->`)
   - slug = `{kebab-title}-{first-author-or-org}-{year}` (26xx → 2026, 25xx → 2025)
   - write `wiki/sources/<slug>.md` with house-style frontmatter + Summary / Key points /
     Concepts & entities (`[[wikilinks]]` to existing pages) / References
   - create concept/entity pages only when a term recurs across pages or passes the page threshold

8. **Reconcile.** Add new source pages to `wiki/index.md` (theme-grouped); bump the count line;
   prepend a `wiki/log.md` entry. Promote any link referenced by ≥3 pages with no page.

9. **Advance state.** Append new thread IDs to `processed_thread_ids`, set `last_processed_date = today`,
   append a `runs[]` record (date, threads_read, ingested count, new pages).

10. **Commit + sync.** `git add wiki/ newsletter-ingest-state.json` → commit
    (`ingest: Newsletter sweep <date> — N sources`) → `./scripts/sync-to-vault.sh`. Do **not** push
    unless asked. Do **not** stage `scripts/evolution-graph.html` or `.claude/`.

## Caveat — why this isn't fully unattended

The Gmail MCP only authenticates inside an interactive claude.ai session. A headless cron run would
likely fail at step 2 (no Gmail auth). So the recurrence is driven by `/loop` while a session is alive:
each loop tick gates on step 1 and only does real work once 7 days have elapsed. To run unattended
between sessions, a human must keep a session open or trigger it manually (this runbook makes that a
one-shot: "run the newsletter-ingest runbook").
