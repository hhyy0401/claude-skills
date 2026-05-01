---
name: paper-survey
description: "Rigorous academic literature survey skill that finds, verifies, and synthesizes real papers on any research topic. Never fabricates citations. Default output is (a) a short in-chat summary with taxonomy, key insights, and open questions, and (b) a saved `<topic>-roadmap.md` whose paper table follows the column schema (`제목 / 저자 / 저널·학회 / 비고`) consumed by the `pdf_crawl` skill — so the user can run `pdf_crawl` immediately on the output to batch-download every paper. Applies a field-aware citation filter and runs a hallucination check before presenting results. Trigger phrases: \"survey X\", \"find papers on X\", \"related work for X\", \"find papers for my thesis\"."
---

# Academic Paper Survey Skill

Find, verify, and synthesize real papers on any research topic — without hallucinated citations. Output is designed to chain directly into `pdf_crawl` for batch PDF download.

## Three absolute rules

1. **Never fabricate.** Cite only papers confirmed via HuggingFace `paper_search` or web search. Never invent titles, authors, arXiv IDs, DOIs, or results.
2. **Never guess metadata.** If anything is uncertain — author order, venue, year — ask the user or mark `[Authors anonymized]` / `[VERIFY]` rather than guessing.
3. **Maximize coverage.** A survey with 30 verified papers beats one with 10 vague ones.

## Quick start

1. **Phase 1** — clarify scope (topic, sub-angle, field, depth)
2. **Phase 2** — search broadly, verify each paper twice (existence + full metadata)
3. **Phase 3** — classify into 4–8 technique families
4. **Phase 3b** — short in-chat summary (no full paper table)
5. **Phase 3c** — save `<topic>-roadmap.md` in `pdf_crawl`-compatible format
6. **Phase 4–6** *(only if explicitly requested)* — LaTeX/PDF report

> Token-saving rule: do **not** generate LaTeX or compile a PDF unless the user says "produce the report", "write the survey", "generate the PDF", or similar. The in-chat summary plus `roadmap.md` is the default and is sufficient for almost every request.

## Chaining with `pdf_crawl`

After this skill writes `<topic>-roadmap.md`, the user runs:

```bash
python3 ~/.claude/skills/pdf_crawl/crawler.py "<path>" --email you@example.com
```

to batch-download every paper. The roadmap's table schema and per-column rules are exactly what `pdf_crawl` expects (see Phase 3c).

---

## Phase 1 — Scope

Confirm (or infer from context) before searching:

| Question | Why it matters |
|----------|----------------|
| Core topic? | Drives search queries |
| Sub-angle / constraint? | e.g., "post-2022 only", "graph-based methods only", "human fMRI" |
| **Field?** | Sets the citation-filter defaults (see Phase 2b): *neuroscience (broad)*, *neuroscience (niche)*, *data mining / KDD*, *NeuroAI / workshop-heavy*, *general ML*, or *other* |
| Depth? | In-chat summary vs. full LaTeX report |
| Output path? | Where to save `roadmap.md` and (if requested) the report |

If the conversation already provides this, skip asking.

---

## Phase 2 — Search and verify

### Step 2a — Search broadly

Run multiple queries; collect all unique papers.

**Primary — HuggingFace `paper_search` MCP** *(set `results_limit: 12`)*:
- Query 1: core topic (broad)
- Query 2: core topic + specific angle
- Query 3: alternative terminology / closely related concept
- Query 4: `"survey"` or `"review"` + topic *(existing surveys are bibliography goldmines)*

**Supplemental — web search:**
- `site:arxiv.org [topic] survey 2023 2024 2025`
- `site:biorxiv.org [topic]` *(neuroscience preprints)*
- `[topic] review` OR `[topic] annual review` *(neuro and data-mining sub-fields publish many structured reviews)*
- `best papers on [topic] KDD / ICDM / WWW` *(data mining)*
- `best papers on [topic] Neuron / Nature Neuroscience / eLife` *(neuroscience)*

**Supplemental — specialized indexes:**
- **PubMed** *(neuroscience / biology)*: `WebSearch: site:pubmed.ncbi.nlm.nih.gov "[topic]"` — more reliable than HuggingFace for wet-lab and clinical papers.
- **Google Scholar** *(citation counts and broader coverage in neuro / social science)*: `WebSearch: site:scholar.google.com "[topic]"` — use the snippet to estimate citation counts when HF returns none.

### Step 2b — Citation filter (field-aware)

Default: include a paper if **established** (≥ N citations) **or recent** (published within the last M months). Pick `(N, M)` from the field set in Phase 1:

| Field | Established threshold (N) | Recent window (M) | Why |
|-------|---------------------------|-------------------|-----|
| **Default / general** | 50 citations | 12 months | Balanced impact vs. recency |
| **Neuroscience (top journals: Nature/Neuron/eLife/etc.)** | 100 citations | 12 months | High base rate; the bar should match |
| **Neuroscience (niche / sub-area)** | 20 citations | 18 months | Smaller field; lower bar avoids missing real work |
| **Data mining / KDD / databases** | 50 citations | 12 months | Default fits |
| **NeuroAI / workshop-heavy / new sub-area** | 20 citations | 18 months | Workshop papers and brand-new sub-fields are under-cited but important |

Always include a paper, regardless of threshold, if it is uniquely **foundational** to the topic (mark `[FOUNDATIONAL]` in `비고`).

When citation count is uncertain, write "approx." and use HF, Google Scholar snippet, or Semantic Scholar to estimate.

### Step 2c — Record metadata for every candidate

| Field | What to record |
|-------|----------------|
| Title | Full title, exactly as published |
| Authors | All authors if ≤ 4; first author + "et al." if more |
| Venue | Conference or journal + year — verified in Step 2e |
| Year | Publication year (venue year, not arXiv upload date) |
| arXiv ID / DOI | At least one, verified |
| Citations | Approximate count, or "< 1 yr" if recent |
| Key idea | Core technique in 1–2 sentences |
| Key insight | Main result or takeaway in 1 sentence |
| Category | Taxonomy family (Phase 3) |

### Step 2d — Verify existence

For each paper:

```
paper_search(query="<exact title>")
```

Fallback: `WebSearch: '"<exact title>" <first-author surname> <year> arxiv'`. Do not cite a paper if neither confirms it.

### Step 2d-bis — Verify full metadata

HuggingFace confirms existence but is unreliable for full metadata. Run a verification pass before any output:

1. Search by arXiv ID: `WebSearch: "arxiv.org/abs/XXXX.XXXXX"` — returns the canonical author list.
2. Cross-check authors (all, in order), title (exact wording), venue, year.
3. **Common AI metadata errors to watch for:**
   - Wrong last author — AI tends to substitute a famous landmark name in the field. In **neuroscience**, watch for false attribution to Hopfield, Marr, Sejnowski, Friston, Gerstner, Hassabis. In **data mining / databases**, watch for Han, Faloutsos, Aggarwal, Leskovec, Tsujii. In **NeuroAI**, watch for DiCarlo, Yamins, Olshausen, Sussillo.
   - Missing middle authors or swapped order
   - Accented characters dropped (Hébert → Hebert)
   - arXiv upload date confused with conference year
4. If a source is unreachable, mark `[VERIFY]` (instead of `[CITED]` / `[NEW]`) in `비고` and tell the user which fields need manual checking.

### Step 2e — Verify venue

> **Do NOT default to "arXiv".** Many papers appear on arXiv first but are later published. Always check.

For each paper:

1. Search DBLP / Semantic Scholar: `WebSearch: "[paper title]" site:dblp.org OR site:semanticscholar.org`
2. Check the arXiv abstract page — the "Comments" field often says e.g. "Accepted at NeurIPS 2024".

**Common venues by area:**

- **Data mining / KDD:** KDD, ICDM, WWW, WSDM, CIKM, SDM
- **Databases:** VLDB, SIGMOD, ICDE, *TKDE*, *DMKD*
- **Network science / graphs:** *Network Science*, *Network Neuroscience*, *J Complex Networks*, *Phys Rev X*, *Phys Rev Research*
- **Neuroscience (general):** *Nature*, *Science*, *Cell*, *Neuron*, *Nature Neuroscience*, *eLife*, *PNAS*, *J Neurosci*, *NeuroImage*, *Cerebral Cortex*, *Current Biology*
- **Computational neuroscience:** *PLOS Comp Bio*, *Network Neuroscience*, COSYNE (abstracts only), CCN, CNS
- **NeuroAI / cross-cutting ML:** NeurIPS (incl. NeurReps workshop), ICLR, ICML, TMLR
- **General ML/AI** *(use only if the paper is plainly in this lane)*: ICML, NeurIPS, ICLR, AAAI, JMLR

Venue must be consistent across the per-paper entry, the table, and the bibliography.

---

## Phase 3 — Classify into a taxonomy

Group verified papers into **4–8 technique families**:

- Each family shares a **core mechanism**, not just a topic area
- Families are mutually exclusive at the primary level (a paper may carry a secondary label)
- Assign each family a letter (A, B, C…) and a short name
- Build the taxonomy from the papers you actually found — don't impose a predetermined structure

---

## Phase 3b — In-chat summary

Default output. Keep it short — the full paper table goes in `roadmap.md`, not in chat.

1. **Overview** — topic, total verified papers, families identified
2. **Per-family summary** — 2–4 sentences per family, naming key papers
3. **Key insights** — 3–5 cross-cutting observations
4. **Open questions** — 2–3 gaps not addressed in the literature

---

## Phase 3c — Save `<topic>-roadmap.md` (pdf_crawl-compatible)

Always save a markdown file with the paper table in the schema `pdf_crawl` consumes. This lets the user batch-download every paper with one command.

**Filename:** `<topic-slug>-roadmap.md` (e.g. `hypergraphlets-brain-roadmap.md`). Use the user's path if they gave one; otherwise default to the current working directory and tell them where it was written.

**Required header (must be exactly this):**

```markdown
| #  | 제목 | 저자 | 저널/학회 | 비고 |
|----|------|------|-----------|------|
| 1  | <full paper title> | <First Last, Co-Author, ...> | *<Venue>* <vol>:<page> | <YYYY>. arXiv:<id>. [CITED]. <one-line key insight>. |
```

**Per-column rules:**

- **제목** — full title, exactly as published.
- **저자** — comma-separated; the **first whitespace token of the first entry must be the first author's surname** (the crawler uses this for the output filename, e.g. `caron_random_2013.pdf`).
- **저널/학회** — short venue + volume/page if available. Examples: `*Nature* 497:113`, `NeurIPS 2024`, `*J Physiol* 202:437`, `KDD 2023`.
- **비고** — must start with the **4-digit publication year** (1800–2199; the crawler extracts it). Optional but encouraged: include `arXiv:XXXX.XXXXX` or `doi:10.XXXX/...` right after the year — `pdf_crawl` skips a Crossref roundtrip when present, making the crawl noticeably faster. Then a tag (`[CITED]`, `[NEW]`, `[FOUNDATIONAL]`, or `[VERIFY]`) and a one-line key insight.

**Multiple tables — encouraged:** the crawler picks up *every* table whose header includes `제목`, `저자`, `저널`, so you can split papers across multiple tables, one per taxonomy family. This pre-organizes the roadmap by family in the markdown without affecting the crawl. Tables without those headers (priority lists, legends, notes) are ignored.

**Around the tables (optional, ignored by crawler):** topic + survey date + paper count above; "Key insights" and "Open questions" sections below.

After writing, end the chat reply with:

> *"Saved `<topic>-roadmap.md` with N papers (M families). Run `python3 ~/.claude/skills/pdf_crawl/crawler.py <path> --email you@example.com` to download all PDFs. Want me to also generate a full LaTeX/PDF report?"*

---

## Phase 4–6 — LaTeX / PDF report *(only on explicit request)*

> Triggered by phrases like "produce the report", "write the survey", "generate the PDF", "make the LaTeX file". Otherwise stop after Phase 3c.

### Phase 4 — Document structure

```
1. Title page (title, date, stats: N papers, M families)
2. Table of contents
3. Scope (topic, filter criteria, survey date)
4. Classification taxonomy (overview table: family letters + names + paper count)
5–N. One section per family:
       - header (letter + name)
       - motivation paragraph
       - key equations (eqbox), algorithms (algobox), if applicable
       - per-paper entries: authors, venue/year, key idea, key insight
N+1. Key insights — 4–6 cross-cutting observations
N+2. Complete paper table — longtable
N+3. References — thebibliography with clickable \href links
```

**Per-paper entry / equation / algorithm boxes:**

```latex
\papersection{Title}{Authors}{Venue, Year}
\textbf{Key idea:} ...\\
\textbf{Key insight:} ...
```

```latex
\begin{eqbox}
\textbf{Equation name:}
\begin{equation}\label{eq:name} ... \end{equation}
\textbf{Intuition:} ...
\end{eqbox}
```

```latex
\begin{algobox}{Algorithm name --- Paper(s)}
\textbf{Input:} ...\\\textbf{Output:} ...
\begin{enumerate}[leftmargin=2em,label=\arabic*.]
  \item ...
\end{enumerate}
\end{algobox}
```

**Bibliography rules:** use `\begin{thebibliography}{99}`; every `\bibitem` has a clickable `\href` (arXiv preferred, DOI otherwise); never invent a DOI or arXiv ID; every `\cite{}` has a matching `\bibitem{}`.

**Deep-analysis sub-section** *(when requested for a specific sub-topic)*: add `\section{Deep Analysis: <Topic>}` with opening, sourced background, step-by-step technique, results tied to verified papers, comparison table, and limitations.

### Phase 5 — Compile

```bash
pdflatex -interaction=nonstopmode survey.tex
pdflatex -interaction=nonstopmode survey.tex   # second pass: TOC + cross-refs
grep "Output written" survey.log
```

Run twice. Fix fatal errors before saving.

### Phase 6 — Hallucination check (before any output, default mode included)

Run the `hallucination-detection` skill on the draft (markdown summary in default mode, full `.tex` in report mode). Focus on:

1. Citation verification — every paper real, with correct authors, title, venue, year
2. Factual claims — key insights match what the papers actually report
3. Numerical accuracy — speedups, parameter counts, etc.
4. Venue accuracy — no paper falsely listed as a major venue

Act on findings: fix CRITICAL/HIGH immediately; fix MEDIUM if straightforward, else flag; LOW only if trivial.

---

## Expanding an existing survey

When asked to add more papers or a new sub-topic:

1. Read the existing `roadmap.md` (and `survey.tex` if report mode).
2. Run HF + web search for the new sub-topic; apply the field-aware citation filter.
3. Verify every new paper (Steps 2d, 2d-bis, 2e).
4. Insert into the existing table(s) — do not rewrite sections that are already good.
5. Recompile if in report mode.

---

## Quality checklist (before any output)

- [ ] Every paper confirmed via HF `paper_search` or web search
- [ ] Field-aware citation filter applied (Phase 2b)
- [ ] Metadata cross-checked (Step 2d-bis); no famous-name substitutions
- [ ] Venue verified (Step 2e); no paper listed as "arXiv" without checking
- [ ] `<topic>-roadmap.md` saved with the exact `| # | 제목 | 저자 | 저널/학회 | 비고 |` header
- [ ] Each `비고` starts with the 4-digit year; arXiv ID / DOI included when known
- [ ] Each `저자` entry's first whitespace token is the first author's surname
- [ ] Hallucination-detection pass completed
- [ ] *(Report mode)* every `\cite` has a matching `\bibitem` with a clickable link; pdflatex ran twice
