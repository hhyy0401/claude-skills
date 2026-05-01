---
name: pdf_crawl
description: "Download PDFs for papers listed in a markdown roadmap file. Parses tables with columns 제목 / 저자 / 저널/학회 / 비고, then for each row uses Crossref to find a DOI, tries arXiv first, then resolves https://doi.org/{DOI} and parses the publisher landing page for the citation_pdf_url meta tag. Saves into a 'pdf/' folder next to the markdown file. Filenames follow {firstauthorlastname}_{firsttitleword}_{year}.pdf (e.g. kim_characterization_2026.pdf). Trigger when the user asks to download PDFs from a literature roadmap, fetch papers listed in a markdown file, or batch-download papers from a research-list .md. Also trigger on '/pdf_crawl <path>' or 'crawl PDFs from <path>'."
---

# pdf_crawl

Batch-download PDFs from a markdown literature roadmap.

## Input format

Any markdown file containing one or more tables with this exact header:

```
| #  | 제목 | 저자 | 저널/학회 | 비고 |
|----|------|------|-----------|------|
| 1  | Random convergence of olfactory inputs in the Drosophila mushroom body | Caron SJC, Ruta V, Abbott LF, Axel R | *Nature* 497:113 | 2013. [CITED]. AL→MB random convergence. |
| 2  | A theory of cerebellar cortex | Marr D | *J Physiol* 202:437 | 1969. [NEW]. Sparse-expansion theory. |
```

Rules:
- The script picks up only tables whose header includes all three of `제목`, `저자`, `저널`. Other tables (priority lists, legends, notes) are ignored.
- **연도** is extracted from the 비고 column (first 4-digit year, 1800-2199).
- **저자**: first comma-separated entry, first whitespace token = last name (e.g. `Caron SJC, Ruta V, ...` → `caron`).
- **제목**: first non-article word, lowercased, alphanumeric only. Articles skipped: `a`, `an`, `the`. Hyphens/punctuation stripped (e.g. `Cellular-resolution ...` → `cellularresolution`).
- Duplicate rows (same paper appearing in multiple tables) are deduplicated by output filename.

## Output

- Folder: `pdf/` next to the markdown file (created if missing).
- Filename: `{firstauthorlastname}_{firsttitleword}_{year}.pdf`
  - `Caron SJC, ... | Random convergence ... | ... | 2013. ...` → `caron_random_2013.pdf`
  - `Kim H, ... | Characterization of ... | ... | 2026. ...` → `kim_characterization_2026.pdf`
- Idempotent: re-running skips files already on disk.

## Invocation

```
python3 ~/.claude/skills/pdf_crawl/crawler.py "<absolute path to .md>" --email you@example.com
```

Or set once via environment variable:

```
export PDF_CRAWL_EMAIL="you@example.com"
python3 ~/.claude/skills/pdf_crawl/crawler.py "<absolute path to .md>"
```

Flags:
- `--email <addr>` (required) — contact email for Crossref's polite-pool User-Agent. Can also be set via `PDF_CRAWL_EMAIL`.
- `--dry-run` — print parsed paper count and planned filenames; no network.
- `--limit N` — process only the first N unique papers (testing).
- `--no-scihub` — disable Sci-Hub fallback (strictly OA sources only).

## Flow per paper

For each paper, the script collects candidate PDF URLs from multiple sources and tries them in order until one downloads as a real PDF:

1. **Crossref** multi-strategy DOI lookup (see "Crossref matching" below).
2. **arXiv** title+author search (token overlap ≥ 0.6).
3. **eLife API** (`api.elifesciences.org`) for any `10.7554/eLife.*` DOI — bypasses eLife's JS-rendered landing page.
4. **bioRxiv** direct URL pattern for `10.1101/*` DOIs.
5. **Publisher** landing: resolve `https://doi.org/{DOI}` with browser-like UA + cookie jar, parse `<meta name="citation_pdf_url">`.
6. **Semantic Scholar** Graph API (`openAccessPdf` field) for additional preprint sources.
7. **Sci-Hub** mirror (sci-hub.ru → .ee → .se → .st) as last-resort fallback for paywalled papers.

The download function recurses once if the response is HTML containing another `citation_pdf_url` tag (handles cookie walls).

Use `--no-scihub` to disable the Sci-Hub fallback if you only want strictly-OA sources.

## Crossref matching (strict + fuzzy)

If the markdown has a small typo, wrong year, or a `[VERIFY]` flag (uncertain title), the script tries multiple Crossref query strategies in order and falls back to a fuzzy match:

1. `query.bibliographic` (title + author) with `type:journal-article` + ±1-year filter.
2. `query.title` + `query.author` with same filters.
3. `query.title` + `query.container-title` (journal) — useful when the author name is hyphenated or has unusual capitalization.
4. Drop year filter, looser query.
5. Drop type filter (allows preprints, etc.).
6. Author + journal + year only (last resort for very wrong titles).

A candidate is scored on three checks (each 0/1): title token-overlap ≥ 0.55, year within ±1, first-author surname appears in candidate's author list.

- **strict** match (auto-accept): score ≥ 2 AND title overlap ≥ 0.45.
- **fuzzy** match (accept-with-warning): no strategy hit "strict", but the best candidate across all strategies has title overlap ≥ 0.35. Logged with `[FUZZY]` in the per-paper line and listed under "FUZZY MATCHES — please verify" in the final report.
- **not found**: nothing reaches the fuzzy threshold; listed under "NOT FOUND — manual download needed".

## Final report

The end-of-run summary prints:
- `done: X downloaded (Y fuzzy), Z already on disk, W not found`
- A **FUZZY MATCHES** section: per paper, shows both the roadmap entry and the matched Crossref title, so the user can spot wrong matches.
- A **NOT FOUND** section: per paper, the title and the failure reason (e.g., "no DOI from Crossref" or the publisher landing URL).

## When to use this skill

Trigger on requests like:
- "download PDFs from this roadmap"
- "fetch all the papers in `<file>.md`"
- "crawl PDFs from `<path>`"
- `/pdf_crawl <path>`

## How to drive it

1. Confirm the markdown path if ambiguous.
2. For lists > 30 papers, do a `--dry-run` first and confirm the count + a sample of filenames before launching the real run.
3. Run real downloads via Bash with `run_in_background: true` (full crawls can take 5+ minutes).
4. After completion, summarize: ok / skip / fail counts. Forward the manual-download candidate list (the script prints publisher URLs for each failure) so the user can grab paywalled ones via institutional access.

## Notes

- Stdlib only (`urllib`, `json`, `re`, `http.cookiejar`) — no external deps.
- Polite rate limits: ~0.3s between API calls, ~0.4s between papers.
- Failures usually mean: (a) Crossref couldn't find a confident title match, or (b) the publisher landing has no `citation_pdf_url` tag (rare; common publishers all emit it). Failure messages include the resolved landing URL for manual follow-up.
