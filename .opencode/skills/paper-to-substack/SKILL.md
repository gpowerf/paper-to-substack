---
name: paper-to-substack
description: Use when the user wants to transform an academic paper (arXiv URL/ID, local PDF, or pasted text) into an engaging Substack-style article. Triggers on phrases like "convert this paper", "make a Substack post", "summarize for Substack", or when an arXiv URL or PDF path is provided. Do NOT use for general summarization or non-Substack writing tasks.
---

# paper-to-substack skill

This skill defines the workflow, style guide, and conventions for transforming academic papers into Substack-style articles. It is loaded into the paper-to-substack orchestrator agent and its subagents.

# Pipeline

```
[input] -> extract source text --> paper-reader --> outliner --> writer --> editor --> output/<slug>.md
```

See `AGENTS.md` at the project root for the full architecture and source-extraction details.

# Substack style guide

## Voice

- Conversational but precise. Imagine explaining to a smart friend who isn't in your field.
- The author is a knowledgeable guide, not a lecturer.
- Avoid academic hedges ("it could be argued that...").
- Avoid blog clichés ("In today's fast-paced world...").

## Structure

- **Hook** (2-3 sentences): pulls the reader in, hints at the finding.
- **Body** (5-8 sections): each with a heading that promises value.
- **Pull quotes** (1-2): verbatim from the paper, blockquoted and attributed.
- **Closing takeaway** (1-2 sentences): the single thing to remember.

## Headings

- Promise value, not label topics.
  - "Why the model breaks on long context" not "Limitations"
  - "What the researchers actually built" not "Methodology"
- Max 8 words.

## Jargon

- Gloss every technical term on first use.
- Format: "term (plain-language gloss)" or "term — plain-language gloss".
- Don't over-explain common technical terms (API, dataset, model).

## Analogies

- Use 1-2 analogies per article for the most abstract concepts.
- Anchor analogies in concrete, everyday images.
- Don't mix metaphors.

## Length

- Target 1200-2500 words.
- Note word count in the final output.

## Forbidden phrases

- "delve", "tapestry", "navigate the landscape"
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
- [ ] Hook is 2-3 sentences, no rhetorical question.
- [ ] Every section heading promises value (no label topics).
- [ ] Every jargon term is glossed on first use.
- [ ] Pull quotes are verbatim and attributed.
- [ ] No banned phrases remain.
- [ ] No fabricated numbers, quotes, or findings.
- [ ] Word count is 1200-2500.
- [ ] Frontmatter is present and complete.
- [ ] Slug is kebab-case, max 8 words, articles and prepositions removed.
