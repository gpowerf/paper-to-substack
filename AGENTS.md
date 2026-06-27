# paper-to-substack

An agentic framework that transforms academic papers into engaging Substack-style articles, preserving key insights while adapting tone and structure for a general audience.

This framework is built natively inside [opencode](https://opencode.ai). There is no separate CLI or library to install - everything is wired up through opencode config, agents, commands, and a skill.

## Architecture

The framework is a multi-agent pipeline orchestrated by a single primary agent:

```
[input] -> extract source text -> paper-reader -> outliner -> writer -> editor -> output/<slug>.md
```

- **paper-to-substack** (primary, default agent): orchestrates the pipeline. Accepts an arXiv URL/ID, a local PDF path, or raw pasted text. Extracts source text, delegates to specialized subagents, writes the final markdown.
- **paper-reader** (subagent): faithful extraction of the paper's claims, findings, methods, contributions, and limitations. No tone adaptation.
- **outliner** (subagent): designs a Substack-style structure (hook, sections, takeaways) from the reader's summary.
- **writer** (subagent): drafts engaging prose for a general audience from the outline + raw paper.
- **editor** (subagent): refines hooks, headings, title, pacing, flow. Suggests pull quotes. Returns final markdown.

See `.opencode/agent/` for the full prompts of each agent.

## Output

Final articles are written to `output/<slug>.md` where `<slug>` is a kebab-case slug derived from the paper title (max 8 words, articles and common prepositions removed).

Each output file begins with YAML frontmatter:

```yaml
---
title: "Chosen Title"
source: "<arxiv-url | pdf-path | pasted>"
authors: ["...", "..."]
date: YYYY-MM-DD
---
```

## Source text extraction

The orchestrator extracts source text from the input before handing it to the paper-reader.

- **arXiv URL/ID**: fetch metadata via the arXiv API (`https://export.arxiv.org/api/query?id_list=<id>`), then download the PDF and convert to text with `pdftotext` (fallback: `python3 -c "import fitz; ..."` if pymupdf is installed).
- **Local PDF**: run `pdftotext <path> -` (or pymupdf fallback).
- **Raw text**: pass through directly.

Verify `pdftotext` is available with `command -v pdftotext`. If missing, install via `sudo apt-get install -y poppler-utils` OR fall back to `python3 -m pip install --user pymupdf` and use the fitz script below. Ask permission before installing anything.

pymupdf fallback script:

```bash
python3 -c "import sys; import fitz; doc = fitz.open(sys.argv[1]); print('\n'.join(p.get_text() for p in doc))" "<pdf_path>"
```

If the source text is over 100k characters, keep the abstract, intro, methods, results, and conclusion, and trim middle sections. Preserve the bibliography only if the article needs to reference specific prior work.

## Substack style

See `.opencode/skills/paper-to-substack/SKILL.md` for the full style guide. Key principles:

- Conversational but not dumbed down.
- Hook in the first 2-3 sentences.
- Use analogies for technical concepts.
- Section headings that promise value, not label topics.
- Pull quotes for memorable lines.
- End with one clear takeaway.

## Usage

From inside opencode, run:

```
/paper-to-substack https://arxiv.org/abs/2401.00001
/paper-to-substack /path/to/local/paper.pdf
/paper-to-substack <paste the abstract or text of the paper>
```

Or simply talk to the default agent - it will route paper-conversion requests through the pipeline.

## Files

- `.opencode/opencode.json` - project config, sets `paper-to-substack` as the default agent.
- `AGENTS.md` - this file, project conventions for agents.
- `.opencode/agent/paper-to-substack.md` - the orchestrator.
- `.opencode/agent/paper-reader.md` - faithful extraction subagent.
- `.opencode/agent/outliner.md` - structure design subagent.
- `.opencode/agent/writer.md` - prose drafting subagent.
- `.opencode/agent/editor.md` - refinement subagent.
- `.opencode/command/paper-to-substack.md` - the `/paper-to-substack` command.
- `.opencode/skills/paper-to-substack/SKILL.md` - style guide and workflow.
