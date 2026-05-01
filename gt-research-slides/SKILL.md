---
name: gt-research-slides
description: Build LaTeX Beamer research slides in the Georgia Tech palette, with a methods-first narrative, keyword-only slides paired to a spoken script, and accessibility for non-specialists. Trigger when the user asks for Beamer slides for a research talk, interview, group meeting, thesis defense, or wants to continue a presentation.tex / interview.tex lineage already styled this way.
type: presentation-skill
author: hhyy0401
version: 1.2
---

# GT Research Slides

A reusable style and structure guide for academic research presentations, distilled from iterative builds of research and interview talks. The skill covers **visual template + typography**, **narrative structure**, **content density rules**, **layout discipline**, and **pairing slides with a spoken script**.

---

## 1. Visual Template

### Color palette (define once, use everywhere)

```latex
\definecolor{gtgold}{RGB}{179,163,105}   % primary accent
\definecolor{gtnavy}{RGB}{0,48,87}       % strong contrast
\definecolor{gtlight}{RGB}{245,240,225}  % soft background
\definecolor{gtdark}{RGB}{40,40,40}      % primary text
\definecolor{gtgray}{RGB}{120,120,120}   % secondary / captions
\definecolor{gtred}{RGB}{175,40,55}      % "focus" highlight
```

**Semantic use:**
- `gtgold` — accents, rules, box frames, bullet markers
- `gtnavy` — "takeaway" boxes (conclusions, punchlines, open questions, the thesis of the talk)
- `gtlight` — soft inner fills for content boxes
- `gtdark` — body text, headings
- `gtgray` — captions, publication venues, de-emphasis, italic annotations
- `gtred` — items you want the audience to focus on *today* (e.g. two of four projects on a trajectory diagram)

### Beamer theme skeleton

- Outer: `infolines`; Inner: `rectangles`
- Footline rewritten: author | title | `frame/total` on dark bar
- Frame title: large bold, gold underline rule (`\rule{0.6\paperwidth}{1.2pt}`)
- Title slide: dark header bar + gold stripe, centered title, institute in gray
- Section page: big gold number block + section title + gold rule, auto-inserted via `\AtBeginSection`

### Reusable tcolorboxes

```latex
\newtcolorbox{goldbox}[1][]{...colback=gtlight, colframe=gtgold,
  colbacktitle=gtgold, coltitle=white, ...}
\newtcolorbox{navybox}[1][]{...}
\newtcolorbox{lessonbox}{...dark background, gold frame, white text...}
\newcommand{\gtemph}[1]{{\color{gtgold}\bfseries #1}}
\newcommand{\gthot}[1]{{\color{gtred}\bfseries #1}}
```

**Semantic use:**
- `goldbox` — content cards (hypothesis, method, definition, one-sentence framing)
- Navy-filled `tcolorbox` — the single punchline/takeaway per slide, centered at bottom (or sometimes top, as a thesis statement)
- `\gtemph` — key nouns in running text (`\gtemph{<key term>}`)
- `\gthot` — "look here first" emphasis on trajectory / roadmap diagrams

---

## 2. Narrative Structure

### Canonical order (research talk, 10–15 min)

1. **Title** (no long institute; two lines: department / university; drop the event label unless it adds signal)
2. **Outline** (2 columns × 2 items, numbered with gold)
3. **Research Interests** (long-term themes + *thesis-statement navy box at top* tying them together; education chronology as a side column, no box; italic one-line cap at the bottom tracing the trajectory)
4. **Research Path / Trajectory** (Venn diagram of prior fields → arrow → current focus; red = what you'll present today, black = preview at end; explicit legend below: *Red = main focus; black = brief preview*)
5. **Section: main work**
6. **Motivation** — the phenomenon, with a punchline open question
7. **Framing** — a single-word title like "Framing" or "Motivation" (short!); two small goldboxes (Goal / Substrate) + one navy takeaway question; **keyword-only slide**
8. **Background principles** — one slide with an italic top-line establishing the core premise; bullets of the principles; bottom navy box transitioning to the prediction question
9. **Problem setup** — data + task + constraint, one slide
10. **Method 1 primary** — formula + parameters, each parameter with a one-line domain-level reading
11. **Method 1 supplementary** — completion/prediction, minimal
12. **Method 1 result on reference dataset** — figure + one navy box with correlations
13. **Method 2 primary** — architecture + inductive bias
14. **Method 2 regularizers** — italic top-line explicitly reminding the audience these are the earlier principles in differentiable form; each regularizer title is `Mechanism ⇒ principle it implements`
15. **Method 2 pipeline (summary)** — one figure, italic one-line narration of the flow
16. **Method 2 result on reference dataset** — figure + one navy box with correlations
17. **What both methods recover** — a single top line stating the emergent properties; side-by-side goldboxes for each method's *distinguishing strength*, with the elaboration as gray scriptsize under the main phrase; single navy conclusion
18. **Section: future**
19. **Future directions** — 3 goldboxes left, 1 small right-aligned hypothesis **in plain text, no box** under a gold rule; hypothesis compressed to 2–3 lines max
20. **Section: ongoing**
21. **Ongoing projects overview** — two small goldboxes naming the project axes (no caption below; let the next slides explain)
22. **Ongoing I** — hypothesis as a *yes/no question* + method as 2 bullets; bottom navy box states what the prediction reveals *about the system* (never claim downstream utility you can't defend)
23. **Ongoing II** — same skeleton + a motif figure on the right
24. **Thank You + Skills (merged)** — skill set in a goldbox (left), a short evocative navy thesis (right); contact at bottom; **no "looking forward to..." boilerplate** — that kind of sentiment belongs in the script only

### Section rule

Use at most 4 sections. Section pages are worth it when a talk is long enough to justify orientation, but **don't split sections smaller than 2 content slides** — it costs context more than it helps.

---

## 3. Content Density Rules

### Slides are keyword scaffolds; the script carries the sentences

- A slide should be **readable in 5 seconds** from the back of the room. If it takes longer, move prose to the script.
- Allowed on a slide: short phrases, single math display, one figure, a 1-line takeaway in a navy box.
- Not allowed: multi-line paragraphs, three-clause sentences, "because X, we Y, so Z" prose.

### Two-object rule per slide

Most slides should have *exactly two* salient objects: e.g. {figure, takeaway box}, {formula, parameter bullets}, {hypothesis box, method box}. Three-object slides are acceptable when paralleling (rule1 | rule2 | prediction). Four-object slides mean something should be merged or cut.

### Primary vs. elaboration

When a box contains a key claim followed by supporting detail, style them differently:

```latex
\gtemph{<key claim>}\\[0.3em]
{\scriptsize\color{gtgray}<supporting detail>}
```

The claim is normal size, gold-emphasized; the elaboration drops to `\scriptsize` and `gtgray`. This matches the reality that the claim earns its place on the slide; the elaboration is almost a footnote.

### Accessibility for non-specialists

Before finalizing, re-read slides + script as if you don't know the field:
- Is every jargon term either defined in the script within its first use, or visibly irrelevant to the argument?
- Does the motivation slide explain *why this problem is worth solving*?
- Is the big-picture approach named in one phrase before any formula?
- Is the core premise (the "this thing comes from that thing" story, if any) stated *before* the principles that govern it?
- Can a listener who missed one slide still follow the next?

### What to cut ruthlessly

- Low-level technical justifications that don't change the audience's decision.
- Benchmark numbers on held-out data that don't cross a qualitative threshold.
- Speaker self-positioning ("As someone who has spent years on X") — verbal only, not on slide.
- Dataset statistics the audience already knows.
- Submission-status phrases ("targeting submission soon") — verbal only.
- Boilerplate enthusiasm ("looking forward to contributing", "excited to work with you") — verbal only, never on closing slide.
- Section previews ("details in the next two slides") — redundant when the next slides obviously provide the details.

### Hypothesis formulation

Phrase ongoing-project hypotheses as **yes/no questions**, not declarative statements. The question frames the *experiment*; the statement preempts the answer.

### Parameter / mechanism explanation

Prefer `name: one-line reading` over full sentences. Each parameter gets its own line in the form `<symbol>: <plain-language meaning>`. Do not narrate the parameters in prose on the slide; the script handles that.

### Results reporting

- Correlation / accuracy numbers belong in a single **navy tcolorbox at the bottom** of the results slide, not in bullet points.
- Format: `\bfseries <metric name> $\rho = <value>$ \qquad\qquad <metric name> $\rho = <value>$`
- Compared methods on one summary slide: short bullets describing *what each method recovers and what it adds*, not which has the lower error.

### Claim discipline on ongoing work

If a project's downstream application hasn't been demonstrated yet, don't put it in the navy takeaway box. Replace with what the method *does show* — a claim about the method's premise, not a claim about downstream utility. The script can still gesture at the potential application, but the slide must not.

---

## 4. Layout Patterns

### Two-column split (hypothesis + method)

```latex
\begin{columns}[T]
  \begin{column}{0.48\textwidth}
    \begin{goldbox}[title=Hypothesis]
      \centering\small
      <one-sentence yes/no question>
    \end{goldbox}
  \end{column}
  \begin{column}{0.48\textwidth}
    \begin{goldbox}[title=Method]
      \small
      \begin{itemize}\setlength\itemsep{0.2em}
        \item <method bullet 1>
        \item <method bullet 2>
      \end{itemize}
    \end{goldbox}
  \end{column}
\end{columns}
```

### Figure with takeaway box

```latex
\begin{center}
  \includegraphics[width=0.6\linewidth]{figures/<figure>.png}
\end{center>
\begin{center}
  \begin{tcolorbox}[colback=gtnavy, colframe=gtgold, coltext=white,
      width=0.72\textwidth, halign=center]
    \small\bfseries <one-line takeaway with key numbers>
  \end{tcolorbox}
\end{center>
```

### Three-goldboxes + hypothesis sidebar (future directions)

```latex
\begin{columns}[T]
  \begin{column}{0.62\textwidth}
    \begin{goldbox}[title=Direction 1] ... \end{goldbox}
    \begin{goldbox}[title=Direction 2] ... \end{goldbox}
    \begin{goldbox}[title=Direction 3] ... \end{goldbox}
  \end{column}
  \begin{column}{0.34\textwidth}
    \vspace{1.4em}
    {\bfseries\color{gtdark} Working hypothesis}\\[0.5em]
    {\color{gtgold}\rule{0.9\linewidth}{0.8pt}}\\[0.7em]
    {\small\color{gtdark}
      <2--3 line working hypothesis>
    }
  \end{column}
\end{columns}
```

**Key pattern:** the sidebar hypothesis is **not in a box**. It's plain text under a gold rule. Reserving the box for the *left* column creates visual hierarchy: "here are the directions, and on the side is the thought tying them together." Compress the hypothesis to 2–3 lines; don't let it overflow the column.

### Research trajectory TikZ diagram

Two overlapping circles (the speaker's prior fields) with individual papers; a short tapered arrow points down to a stadium-shaped pill labeled with the current field; below the pill, 4 items with wide horizontal spacing, with the items the speaker will cover today highlighted in `gtred`, and items previewed at the end in `gtdark`. A small caption under the figure explains the color key.

**Spacing discipline:** the whole TikZ diagram should leave at least ~1cm breathing room above the frame's footline. If items drift too low, raise the whole figure by adjusting the circle centers upward (`\fill[...] circle` at higher y) and reshortening the arrow, rather than shrinking fonts.

### Thesis-statement navy box at top

The thesis of a talk, or the common thread across a slide, sometimes belongs at the **top** of a slide, not the bottom:

```latex
\begin{center}
  \begin{tcolorbox}[colback=gtnavy, colframe=gtgold, coltext=white,
      width=0.88\textwidth, halign=center]
    \small\bfseries One-line thesis tying the slide's content together
  \end{tcolorbox}
\end{center>
```

Use this when the slide's content boxes elaborate on a unifying claim — the claim goes first, the columns expand on it.

---

## 5. Vertical Positioning Discipline

**IMPORTANT — HARD-LEARNED LESSON:** Beamer's default vertical padding below the frame title is generous, and `\begin{columns}[T]` adds *its own* top alignment strut. Content almost always sits lower than intended. **Default to negative vspace, not positive.**

### Default vspace values for this template

Use these as starting points, not `\vspace{0.3em}` or `\vspace{0.5em}`:

| Position                                 | Default vspace    |
|------------------------------------------|-------------------|
| Just below frame title (before content)  | `\vspace{-0.6em}` to `\vspace{-1.0em}` |
| Top of each `\begin{column}` inside `[T]` columns | `\vspace{-0.4em}` to `\vspace{-0.8em}` |
| Between stacked goldboxes                | `\vspace{0.5em}`  |
| Before a bottom navy takeaway box        | `\vspace{0.7em}`  |
| Before a centered standalone figure      | `\vspace{-0.3em}` |

If a slide ends up with the boxes visually centered or floating toward the bottom, it's because these negatives weren't applied. **Start with them, not without them.**

### Box width discipline (narrowing a content row)

When two side-by-side goldboxes look too wide — they stretch edge-to-edge and the slide feels cluttered — **wrap the pair in padding columns** rather than shrinking each box individually:

```latex
\begin{columns}[T]
  \begin{column}{0.12\textwidth}\end{column}
  \begin{column}{0.36\textwidth}
    \begin{goldbox}[title=...]  ...  \end{goldbox}
  \end{column}
  \begin{column}{0.36\textwidth}
    \begin{goldbox}[title=...]  ...  \end{goldbox}
  \end{column}
  \begin{column}{0.12\textwidth}\end{column}
\end{columns}
```

The `0.12 / 0.36 / 0.36 / 0.12` split replaces `0.48 / 0.48`. Padding columns on both sides visually center the pair and shrink each box by ~25% without losing its internal layout.

**Common splits (symmetric pairs):**
- `0.48 / 0.48` — edge-to-edge, use when the two boxes carry equal weight as the slide's primary content
- `0.12 / 0.36 / 0.36 / 0.12` — medium-narrow, use when the two boxes are a conclusion or preview rather than the main argument
- `0.17 / 0.66 / 0.17` (single centered column) — narrow, use for single-column "ongoing work" or summary slides where the content needs emphasis but not full width

**Asymmetric splits (content + sidebar):**
- `0.62 / 0.38` — three content boxes on left, small hypothesis sidebar on right (future-directions pattern)
- `0.58 / 0.38` with gap — slight narrowing when the left column has three stacked boxes that need to breathe

Don't shrink individual box widths by passing `width=0.xy\textwidth` inside the goldbox. That fights the column system. Always shrink via column widths + padding columns.

### Page numbering

Show the **current slide number only**, not `current / total`. Total counts make a talk feel like a countdown and discourage the speaker from spending time on any one slide. Footer pattern:

```latex
\begin{beamercolorbox}[wd=.10\paperwidth,ht=2.5ex,dp=1ex,right]{date in head/foot}
  \insertframenumber{}\hspace{1em}
\end{beamercolorbox}
```

Not:

```latex
\insertframenumber{}/\inserttotalframenumber
```

If an audience member wants to say "back to slide 12," the number alone is enough. They don't need to know whether the talk has 22 or 27 slides.

### Positive-space rules

- **Raise content toward the frame title by default.** After the title's gold rule, the natural reading start is *immediately* below it, not two blank lines down.
- On TikZ diagrams with items below a central figure, raise the central element if the bottom items are clipped.
- On closing slides ("Thank You" etc.), the greeting block sits high and the skill-set / thesis boxes slot immediately below with minimal gap.

### Box-up recipe (in order)

When a box is sitting too low on a slide, apply these moves in this order — *don't skip steps*:

1. **Add `\vspace{-0.6em}`** directly below `\begin{frame}{...}` before any content.
2. **Add `\vspace{-0.4em}`** at the top of each `\begin{column}{...}` inside a `[T]` columns environment. Do this *even if the column is the tallest one* — the strut still applies.
3. **Cut `\vspace{...}`** that was explicitly placed above the box (old content left over from earlier iterations).
4. **Shrink the element above the box** (figure, rule, heading) rather than shrinking the box itself.
5. **Inspect `\\[...em]` separators** above the box and reduce (`\\[1.0em]` → `\\[0.4em]`).

Things that do NOT move a box up: `halign=center`, `boxrule`, `colback` — those don't affect vertical position.

### Symmetric box-down pattern

When a box needs to sit *lower* on the slide (e.g. a right-column element aligning with the middle of a tall left column):

- Apply **positive `\vspace{1.0em}`** or more at the top of that specific column.
- Use **`\vfill`** before the box inside its column if the column's content allows it.

### Single-slide opinionated default

For a standard two-column content slide in this template, start with:

```latex
\begin{frame}{Title}
  \vspace{-0.6em}  % <-- default, keep this
  \begin{columns}[T]
    \begin{column}{0.48\textwidth}
      \vspace{-0.4em}  % <-- default, keep this
      \begin{goldbox}[title=...]
        ...
      \end{goldbox}
    \end{column}
    \begin{column}{0.48\textwidth}
      \vspace{-0.4em}  % <-- default, keep this
      ...
    \end{column}
  \end{columns}
\end{frame}
```

If after generating a slide the user says "box too low" or "move up," the correct fix is almost always one of these three negatives being missing. Don't reach for `vspace{0.3em}` as a neutral default — `0em` means *you already added positive space that needs to come back out.*

---

## 6. Script Pairing

Every slide gets a block in a companion markdown file (`presentation_script.md` or `interview_script.md`):

```markdown
## Slide N — Title (≈Xs)

Spoken paragraph, full sentences, written to be read directly aloud.
```

### Global rules

- Each script block targets 30–90 seconds at normal speaking pace (longer is fine for method slides that carry the core narrative).
- Total target: aim ~1 min per slide as a first pass; actual cuts happen at the trim pass.
- Time budgets at the top of each block; total timing summary + "protected block" + "trimmable block" at the bottom so the speaker can cut on the fly.

### Script page numbers must match actual slide page numbers

This is a hard requirement and one of the most common mistakes.

The script's slide numbers must line up exactly with what Beamer renders as the page number in the footer. That means **every page in the compiled PDF gets an entry in the script**, including:

- The title slide (page 1)
- The outline slide
- Every section page inserted by `\AtBeginSection[]` — these count as numbered pages in the PDF
- Every content frame
- The closing "Thank You" frame

Don't skip section pages in the script just because their spoken content is one sentence. The speaker uses page numbers during the talk ("let me go back to slide 11") and during Q&A, so a misaligned script is disorienting mid-presentation.

For section pages, write a short transition placeholder:

```markdown
## Slide 6 — Section: Geometric Models (≈5s)

Okay, let me start with the main project.
```

```markdown
## Slide 19 — Section: Future Directions (≈5s)

Let me talk about where this goes next.
```

If the deck is reorganized and one content slide moves, every downstream slide number in the script shifts. Renumber the entire script, don't just patch the changed slide. The sanity check is to count `\begin{frame}` (plus any section-auto-inserts) in the `.tex` and confirm the script has the same count with matching titles.

### Sentence-level style discipline

These are hard rules. Every generated script must obey them.

1. **No colons (`:`) as punctuation.** A colon is read as a pause-then-enumerate, which sounds like a memo being read aloud, not speech. Replace with full sentence connectives.

2. **No em-dashes (`—`).** The speaker's voice can't render an em-dash. Replace with commas, parentheses, or a period break.

3. **No noun-phrase fragments as standalone sentences.** Every sentence must have a subject and a verb. Disconnected bullet-style fragments sound abrupt and off-script. Reframe each fragment by adding a subject and verb (e.g. start with "Let me...", "I have...", "For the...").

4. **First-person, active voice.** Prefer "We adopt / I want / let me walk through" over "The method is / The approach uses".

5. **No meta markers** like `Method:`, `Algorithm:`, `Hypothesis:` leading a sentence. The slide already labels the structure; the speaker narrates.

6. **No references to slide layout.** The audience can see the slide. Don't say "the navy box at the bottom," "on the left," "the figure on the right," "in the goldbox up top." Deliver the content; the eye does the rest.

### Method explanation structure: big picture → concrete algorithm

Every method slide in the script is a two-pass explanation. Announce the pass, don't write `Big picture:`.

Pattern:

> "Let me walk through this in two passes. For the big picture, [intuition, what the model is doing at a high level, why it fits the problem]. Now let me get into the concrete form. [Actual algorithm: inputs, formulas, parameters, steps]."

The first pass earns the audience's attention; the second pass cashes it in with the technical detail. Don't reverse the order.

### Opening (interview context)

If the talk is an interview, the first slide's script should include:
- Greeting and self-introduction (advisor, department, university)
- Thanks for the interview opportunity, expressed naturally (not with "honored to be here" if it sounds canned)
- Brief framing of what's about to come as "I'd like to introduce you to my research," not "I'll walk through my most exciting project"

This frames it as a conversation with a host, not a monologue.

### Closing

- Skip "looking forward to contributing" or "excited to work with you" on the *slide*; say them verbally in the closing script instead.
- End with "Thank you so much. I'm happy to take any questions." or similar natural close.
- If the closing slide merges a skill summary, explicitly walk the audience through the skill bullets verbally, using first-person active ("I work with... / I have experience with..."), not passive descriptions.

### Transition language

Use natural bridges between sections. Keep them under a sentence each.
- "Okay, let me start with the main project."
- "Alright, we're in the last stretch."
- "Let me tell you about the data first."
- "This is the slide I care about most."
- "That's what I wanted to share today."

Avoid stilted phrases like "Moving on to the next section," "At this juncture," or "With that said."

### What the script must NOT do

- It must not summarize or paraphrase what's literally written on the slide ("here on this slide you can see..."). The script extends the slide, it doesn't describe it.
- It must not narrate its own structure ("Now I will transition to..."). Just make the transition.
- It must not use lists of bare items ("first X, second Y, third Z") without embedding them in sentences with verbs.

### Iteration test

Read the script out loud, at presentation pace, without looking at the slide. If you hear any of the following, revise:
- A sentence that begins with a noun and has no verb.
- A colon or dash that would require an unnatural pause.
- A phrase that refers to slide geometry ("in the box on the left").
- A sentence you'd never actually say in conversation.

---

## 7. Iteration Checklist

Before finalizing any deck, ask:

- [ ] Would a non-specialist understand the *motivation* from slides alone?
- [ ] Does every slide earn its place? (If the script for slide N is "and here we also did X," cut N.)
- [ ] Are hypothesis slides **questions**, not assertions?
- [ ] Is the *big-picture name* of the approach on the slide before any formula?
- [ ] Is the core premise stated before the principles that govern it?
- [ ] Is the color semantics consistent (red = focus today, navy = takeaway, gold = content)?
- [ ] Does the last slide say *both* "thank you" *and* something about the bridge — without boilerplate enthusiasm?
- [ ] Does the footline show slide numbers, so someone can ask "go back to slide 12"?
- [ ] Does text fit without being cut off? (Reduce figures *before* shrinking text.)
- [ ] Are boxes positioned with breathing room above the frame's bottom edge?
- [ ] Are the two ongoing / preview items visually de-emphasized vs. the main focus?
- [ ] Did the author move boxes *up* by reducing vspace above rather than below?

---

## 8. Stylistic Conventions

- **No em-dashes** (writer preference). Use commas, colons, or parentheses.
- Avoid emojis in slides unless explicitly requested.
- Abbreviate venue names on research-trajectory slides (e.g. abbreviated conference acronym + year); never write out full journal titles.
- Use "under review" in gray italic for unpublished work you did; use "ongoing" in red italic for in-progress work you're previewing (and reverse, if the "presenting today" work is the under-review one — whichever is the *focus of this specific talk* gets the red).
- On figures: always include a thin gray caption under the image when the figure isn't self-explanatory. No caption when the frame title already names the figure.
- Prefer the form `Mechanism ⇒ principle` in regularizer / method box titles, to keep the slide honest about what maps to what.

---

## 9. Closing Slide Aesthetic

The final slide is the last thing the audience sees. Don't waste it on boilerplate.

- **Keep the greeting block tight at the top** (Thank You + Questions? + two gold rules, total ~2cm of vertical space).
- **Left: skill set goldbox** (5 scriptsize bullets max; this answers "what can you do?").
- **Right: one evocative navy-box thesis** — something like `Two short clauses naming the bridge your work builds.` Abstract enough to be memorable, concrete enough to be *you*.
- **Contact line at the bottom**: name, email, homepage.
- Never put "I'm excited to work with you" or "looking forward to contributing" on the slide. Deliver that verbally, where the sincerity lands.

---

## 10. Variants

- **Group meeting deck** (30–45 min): same skeleton, but expand each method section to 3–4 slides and add a detailed results section before "what both methods recover."
- **Conference talk** (12 min): collapse the research-path diagram into the title slide; skip the motivation → framing split and merge into one slide.
- **Thesis defense** (60 min): add a "Chapter contributions" slide after outline, and a "Publications / artifacts" slide before the thank-you.

---

## 11. Trigger patterns

Invoke this skill when the user says things like:
- "Make slides for my talk / interview / defense"
- "Make an interview ppt" / "make a group meeting deck"
- "Turn this paper into a presentation"
- "Continue from my presentation.tex / interview.tex"
- "Use my GT template"

Skip this skill when:
- The user explicitly requests a different institutional palette / theme.
- The output must be a generic `.pptx` deck rather than LaTeX/PDF.
- The user wants a poster (skill is slide-specific).
