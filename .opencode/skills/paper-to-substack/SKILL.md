---
name: paper-to-substack
description: Use when the user wants to transform an academic paper (arXiv URL/ID, local PDF, or pasted text) into an engaging Substack-style article. Triggers on phrases like "convert this paper", "make a Substack post", "summarize for Substack", or when an arXiv URL or PDF path is provided. Do NOT use for general summarization or non-Substack writing tasks.
---

# paper-to-substack skill

This skill defines the workflow, style guide, and conventions for transforming academic papers into Substack-style articles. It is loaded into the paper-to-substack orchestrator agent and its subagents.

# Pipeline

```
[input] -> extract source text -> paper-reader -> outliner -> writer -> editor -> output/<slug>.md
            /tmp/paper-source.txt   /tmp/paper-reader-output.md  /tmp/paper-outline.md  /tmp/paper-draft.md  /tmp/paper-final.md
```

**File-based handoffs are mandatory at every stage.** Every subagent reads its inputs from files and writes its output to a file. The orchestrator passes file PATHS to subagents, never inline content. Large inline payloads cause 504 upstream idle timeouts on the Task and Write tools; file-based handoffs avoid this entirely.

Similarly, the orchestrator writes the source text via bash (heredoc for pasted text, stdout-redirect for `pdftotext`/`pymupdf`), NOT the Write tool, because the Write tool times out on large payloads. For very large pasted text (over ~20KB), split the heredoc across multiple `cat >>` appends. The final output file is assembled with `cat` (frontmatter + `/tmp/paper-final.md`), not re-transmitted through the Write tool.

The orchestrator cleans up the `/tmp/` intermediate files after writing the final output.

All subagents that produce long markdown (reader, outliner, writer, editor) write to a file rather than returning content in their response. Inline responses are unreliable for long markdown — empty or truncated responses lose the work, and large payloads cause upstream timeouts. The orchestrator reads the final file from `/tmp/paper-final.md` and wraps it with YAML frontmatter via bash concatenation.

See `AGENTS.md` at the project root for the full architecture and source-extraction details.

# idea-to-substack pipeline

A second primary agent, `idea-to-substack`, generates a Substack-style article from a structured article idea or outline (no source paper). It reuses the writer and editor subagents in a generalized "idea mode" and adds two new subagents:

```
[structured idea/outline] -> /tmp/idea-input.txt -> idea-planner -> [idea-researcher] -> writer -> editor -> output/<slug>.md
                                                 /tmp/paper-outline.md   /tmp/idea-research-notes.md  /tmp/paper-draft.md  /tmp/paper-final.md
```

- **idea-to-substack** (primary agent): orchestrates the idea pipeline. Accepts a structured idea/outline (file path, pasted text, or loosely-structured idea). Writes to `/tmp/idea-input.txt`, delegates to subagents, writes the final markdown.
- **idea-planner** (subagent): normalizes the user's outline into the exact Substack outline format the writer expects (candidate titles, hook, 5-8 section bullets, tone notes, pull-quote plan), and flags gaps that would benefit from supporting research in a `## Research needed?` section. Writes to `/tmp/paper-outline.md` (same path the outliner uses, so writer/editor need no path changes).
- **idea-researcher** (subagent, optional): runs only when the planner flags gaps AND the user opts in. Fetches supporting links/facts via webfetch and writes notes to `/tmp/idea-research-notes.md` for the writer/editor to ground claims.
- **writer** / **editor** (generalized): run in "idea mode" when no source paper path is provided. They relax source-fidelity rules (no "preserve numbers exactly from source", no "only source quotes", no equations removal, no "point to source paper" closing line) while keeping all style + no-fabrication rules. See the writer and editor agent prompts for the mode-specific instructions.

The research step is optional. The orchestrator asks the user ONCE before spawning the researcher (web fetches are a side effect and "optional" means per-run consent). If skipped, the writer runs purely from the planner's outline.

File-based handoffs, `/tmp/` cleanup, slug derivation, and frontmatter assembly are identical to the paper pipeline. The Substack style guide below applies to BOTH modes.

## Idea-input schema (flexible)

The planner accepts anything reasonable, but the user is encouraged to provide:

```markdown
# <Working title or topic>

## Audience
<who this is for, e.g. "engineers new to k8s", "PMs curious about AI">

## Angle / Thesis
<1-2 sentences: the core argument or insight>

## Key points
- <point 1>
- <point 2>

## Tone
<e.g. "curious and practical", "wry", "urgent">

## Sources (optional)
- <url or note>

## Notes
<length target, things to avoid, specific examples to include>
```

## Frontmatter for idea-driven articles

```yaml
---
title: "Chosen Title"
source: "idea"
authors: []
date: YYYY-MM-DD
---
```

`authors: []` is kept for schema consistency with paper outputs.

# Substack style guide

## Voice

- Conversational but precise. The reader is tech-interested: they follow AI, use the tools, but don't have a CS degree or engineering background. Explain to them, not to a peer, not to a child.
- The author is a real person with opinions, not a neutral summarizer. Allow first-person sparingly where it adds candor or signals personal judgment.
- Not every paragraph needs a textbook structure (topic, evidence, transition). A one-sentence paragraph for emphasis is fine. Uneven paragraph lengths read as human.
- Occasional informal constructions are natural: starting sentences with "And," "But," or "So," using parenthetical asides, brief self-interruption. Keep these but don't force them.
- Avoid academic hedges ("it could be argued that...").
- Avoid blog clichés ("In today's fast-paced world...").

## Structure

- **Hook** (2-3 sentences): pulls the reader in, hints at the finding.
- **Source attribution** (in the intro, for articles based on a source paper): name the paper's full title and lead author on first mention so readers can find the original. Keep it natural, woven into a sentence, not a formal citation. Example shape: "A recent paper, Jane Smith and colleagues' 'Paper Title,' ..." This only applies to the paper-to-substack pipeline; idea-driven articles have no source paper to name.
- **Body** (5-8 sections): each with a heading that promises value.
- **Pull quotes** (1-2): verbatim from the paper, blockquoted and attributed.
- **Closing takeaway** (1-2 sentences): the single thing to remember.

## Headings

- Promise value, not label topics.
  - "Why the model breaks on long context" not "Limitations"
  - "What the researchers actually built" not "Methodology"
- Max 8 words.

## Jargon

- Gloss every technical term on first use, including terms that feel common (model, dataset, training, inference, API, commit). Assume curiosity, not background.
- Format: "term (plain-language gloss)" or "term: plain-language gloss".

## Analogies

- Use 2-4 analogies per article for the most abstract concepts.
- Anchor analogies in concrete, everyday images.
- Don't mix metaphors.

## Math handling

The reader is tech-interested, not a researcher. They want to know what the math *means*, not see the math itself.

- **Remove** all equations, formulas, theorems, proofs, and formal notation (e.g. `A(θ) + α·C(θ) = κ`, `Θ(2^(n²))`, `S = (C, D, E)`). Replace each with a plain-language statement of what it implies.
- **Keep** practitioner-scale numbers: effect sizes, percentages, speedups, big-O in passing prose (e.g. "doubles faster than you can count"). These communicate scale without being formal notation.
- **If a concept needs the math to be understood**, use an analogy instead. The analogy carries the meaning; the equation would only alienate.
- **Never reproduce a formula and then restate it in English.** Pick one: the English version. The formula is for the source paper, not the article.
- **Close the article with a single line** after the takeaway pointing readers to the original paper for full derivations and formal results.

## Punctuation

- Do NOT use em dashes (—) in prose. Readers dislike them and they read as a stylistic tic.
- Use commas, colons, or parentheses instead:
  - Appositive or aside → commas or parentheses ("The model, trained on web text, fails on...").
  - Emphasis or pause → a period and a short sentence, or a colon.
  - Definition or elaboration → a colon ("the metric: how often the model hallucinates").
- The ONLY allowed em dash is the single `— ` that prefixes a pull-quote attribution (standard journalistic convention).

## Length

- Target 1200-2500 words.
- Note word count in the final output.

## Forbidden phrases

- "delve", "tapestry", "navigate the landscape", "quietly"
- "in today's world", "it's worth noting"
- "At the end of the day"
- "Let's dive in"
- "Picture this"
- Rhetorical questions as hooks

## Forbidden tactics

- Clickbait titles that don't deliver.
- Listing the paper's contributions as bullets.
- Inventing quotes, numbers, or findings.
- Adding references or inline citations.
- YAML frontmatter (the orchestrator adds that).

# Output conventions

- File path: `output/<slug>.md`
- Slug: kebab-case, derived from the paper title, max 8 words, remove articles and common prepositions.
- Frontmatter (added by orchestrator, NOT by writer/editor):

```yaml
---
title: "Chosen Title"
source: "<arxiv-url | pdf-path | pasted>"
authors: ["...", "..."]
date: YYYY-MM-DD
---
```

- Word count noted in an HTML comment at the top of the file (added by editor).

# Source-text extraction

See `AGENTS.md`. Summary:

1. **arXiv URL/ID**: fetch metadata via the arXiv API (`https://export.arxiv.org/api/query?id_list=<id>`), download the PDF, `pdftotext` it.
2. **Local PDF**: `pdftotext <path> -` (or pymupdf fallback).
3. **Raw text**: pass through.

Always run `command -v pdftotext` first. Fallback: `python3 -c "import fitz; ..."`. Install one if neither is present (ask permission before installing).

# Quality checklist

Before the orchestrator writes the final file, confirm:

- [ ] Title is from the candidate titles list (or one alternative proposed by editor).
- [ ] Title passes the 3-question headline test (see `.opencode/skills/substack-headline/SKILL.md`).
- [ ] Hook is 2-3 sentences, no rhetorical question.
- [ ] Source paper's full title and lead author are named in the intro (paper-to-substack pipeline only).
- [ ] Every section heading promises value (no label topics).
- [ ] Every jargon term is glossed on first use.
- [ ] No equations, derivations, or formal notation (practitioner numbers OK).
- [ ] Closing line points to the source paper for derivations.
- [ ] Pull quotes are verbatim and attributed.
- [ ] No banned phrases remain.
- [ ] No em dashes in prose (only the pull-quote attribution em dash is allowed).
- [ ] No fabricated numbers, quotes, or findings.
- [ ] Word count is 1200-2500.
- [ ] Frontmatter is present and complete.
- [ ] Slug is kebab-case, max 8 words, articles and prepositions removed.
