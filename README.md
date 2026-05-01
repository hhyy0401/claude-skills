# claude-skills

Three [Claude Code](https://docs.claude.com/en/docs/claude-code/skills) skills for academic research workflows. They are designed to compose: `paper-survey` finds and verifies papers, writes a roadmap markdown file, and `pdf_crawl` then turns that roadmap into a folder of downloaded PDFs. `gt-research-slides` is independent and handles presentation building.

## Skills

### `paper-survey/`

A literature-survey workflow that finds, verifies, and synthesizes real papers on any research topic without fabricating citations.

- Searches HuggingFace `paper_search` and web search across multiple query angles.
- Applies a citation filter: a paper must have **≥ 50 citations** OR be **published within the last 12 months** (foundational papers are exempt).
- Verifies every paper twice — existence (HF or arxiv search) and full metadata (authors, exact title, venue, year) — before citing.
- Classifies findings into 4–8 technique families and writes an in-chat summary with key insights and open questions.
- Saves a `roadmap.md` whose paper table follows the column schema consumed by `pdf_crawl`, so the next step is a single command away.
- Optional report mode: full LaTeX + compiled PDF survey with per-family sections, equation/algorithm boxes, and a clickable bibliography.
- Final output goes through a hallucination-detection pass before being shown to the user.

Trigger phrases: *"survey X"*, *"find papers on X"*, *"what's the related work for X"*, *"find more papers for my thesis"*.

### `pdf_crawl/`

A small crawler that downloads PDFs for every paper in a markdown literature roadmap. Designed as the natural follow-up to `paper-survey`, but works on any markdown file that uses the same table schema.

- Parses markdown tables with four columns (title, authors, venue, notes); tables without those headers are ignored, so you can mix curated tables with notes/legends in the same file.
- For each paper, walks a resolution chain until a real PDF lands: **Crossref** (multi-strategy DOI lookup) → **arXiv** (title + author search with token-overlap scoring) → **eLife API** for `10.7554/eLife.*` DOIs → **bioRxiv** for `10.1101/*` DOIs → **publisher landing page** (parses the `citation_pdf_url` meta tag with a browser-like UA + cookie jar) → **Semantic Scholar** Graph API → **Sci-Hub** mirror chain as a last-resort fallback.
- Crossref matching combines **strict** (auto-accept) and **fuzzy** (accept-with-warning) modes scored on title overlap, year proximity, and first-author surname.
- Output filenames follow `{firstauthorlastname}_{firsttitleword}_{year}.pdf` and are saved into a `pdf/` folder next to the markdown. Re-running is idempotent — files already on disk are skipped.
- Stdlib only (`urllib`, `json`, `re`, `http.cookiejar`) — no external dependencies. Polite rate limits between API calls.
- Final report prints `done / fuzzy / skipped / not-found` counts and a manual-download list (with publisher URLs) for any paper the crawler could not resolve.

Run as:

```bash
python3 pdf_crawl/crawler.py "<path to .md>" --email you@example.com
```

Or set `PDF_CRAWL_EMAIL` once in your shell environment. The email is required because Crossref's polite pool needs a contact in the User-Agent header.

### `gt-research-slides/`

A general guide for structuring LaTeX Beamer research talks. It is a style sheet and narrative scaffold, not a slide generator — Claude reads the rules here and applies them when building or extending a `.tex` deck.

- **Visual template:** a six-color palette with a defined semantic role for each color, beamer outer/inner theme skeleton, reusable `tcolorbox` macros (`goldbox` for content cards, navy boxes for takeaways, `\gtemph` and `\gthot` for inline emphasis), and a section page auto-inserted via `\AtBeginSection`.
- **Narrative structure:** a canonical slide order for a 10–15 min research talk, plus variants for group-meeting decks (30–45 min), conference talks (12 min), and thesis defenses (60 min).
- **Content-density rules:** the two-object-per-slide rule, a primary-vs-elaboration typographic split, an "ongoing-project hypothesis as a yes/no question" rule, and a strict cut-list for the kinds of phrases that don't earn slide space.
- **Layout patterns:** ready-to-paste LaTeX scaffolds for two-column hypothesis-and-method splits, figure-with-takeaway, three-direction sidebars, research-trajectory TikZ diagrams, and top-of-slide thesis boxes — all using placeholders so the content stays generic.
- **Vertical-positioning discipline:** a table of default `\vspace` values plus a "box-up recipe" for the recurring problem of content sitting too low under the frame title.
- **Script pairing:** every slide is paired with a markdown script block written to be read aloud, with hard rules (no colons, no em-dashes, no noun-phrase fragments, first-person active voice, no slide-layout references, no meta markers like `Method:`).

Trigger phrases: *"make slides for my talk / interview / defense"*, *"turn this paper into a presentation"*, *"continue from my presentation.tex"*.

## Typical pipeline

```
"Survey papers on <topic>"     →  paper-survey   →  roadmap.md
                                                    │
                                                    ▼
                                              pdf_crawl   →  pdf/*.pdf

"Make slides for <talk>"       →  gt-research-slides   →  presentation.tex + presentation_script.md
```

## Install

Symlink each skill into your local Claude Code skills directory:

```bash
ln -s "$(pwd)/paper-survey"       ~/.claude/skills/paper-survey
ln -s "$(pwd)/pdf_crawl"          ~/.claude/skills/pdf_crawl
ln -s "$(pwd)/gt-research-slides" ~/.claude/skills/gt-research-slides
```

Claude Code picks them up at the start of a new session.

## License

MIT.
