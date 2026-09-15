# paper-to-substack

An agentic framework that transforms academic papers into engaging Substack-style articles, preserving key insights while adapting tone and structure for a general audience.

This framework is built natively inside [opencode](https://opencode.ai). There is no separate CLI or library to install - everything is wired up through opencode config, agents, commands, and a skill.

## Architecture

The framework is a multi-agent pipeline orchestrated by a single primary agent:

```
[input] -> extract source text -> paper-reader -> outliner -> writer -> editor -> output/<slug>.md
            /tmp/paper-source.txt   /tmp/paper-reader-output.md  /tmp/paper-outline.md  /tmp/paper-draft.md  /tmp/paper-final.md
```

- **paper-to-substack** (primary, default agent): orchestrates the pipeline. Accepts an arXiv URL/ID, a local PDF path, or raw pasted text. Extracts source text, delegates to specialized subagents, writes the final markdown.
- **paper-reader** (subagent): faithful extraction of the paper's claims, findings, methods, contributions, limitations. No tone adaptation.
- **outliner** (subagent): designs a Substack-style structure (hook, sections, takeaways) from the reader's summary.
- **writer** (subagent): drafts engaging prose for a general audience from the outline + raw paper.
- **editor** (subagent): refines hooks, headings, title, pacing, flow. Suggests pull quotes. Returns final markdown.

See `.opencode/agent/` for the full prompts of each agent.

## Architecture: idea-to-substack

A second primary agent generates a Substack-style article from a structured article idea or outline (no source paper). It reuses the writer and editor subagents in a generalized "idea mode" and adds two new subagents:

```
[structured idea/outline] -> /tmp/idea-input.txt -> idea-planner -> [idea-researcher] -> writer -> editor -> output/<slug>.md
                                                 /tmp/paper-outline.md   /tmp/idea-research-notes.md  /tmp/paper-draft.md  /tmp/paper-final.md
```

- **idea-to-substack** (primary agent): orchestrates the idea pipeline. Accepts a structured idea/outline (file path, pasted text, or loosely-structured idea). Writes to `/tmp/idea-input.txt`, delegates to subagents, writes the final markdown.
- **idea-planner** (subagent): normalizes the user's outline into the exact Substack outline format the writer expects (candidate titles, hook, 5-8 section bullets, tone notes, pull-quote plan), and flags gaps that would benefit from supporting research in a `## Research needed?` section. Writes to `/tmp/paper-outline.md` (same path the outliner uses, so writer/editor need no path changes).
- **idea-researcher** (subagent, optional): runs only when the planner flags gaps AND the user opts in. Fetches supporting links/facts via webfetch and writes notes to `/tmp/idea-research-notes.md` for the writer/editor to ground claims.
- **writer** / **editor** (generalized): run in "idea mode" when no source paper path is provided. They relax source-fidelity rules (no "preserve numbers exactly from source", no "only source quotes", no equations removal, no "point to source paper" closing line) while keeping all style + no-fabrication rules. See the writer and editor agent prompts for the mode-specific instructions.

The research step is optional. The orchestrator asks the user ONCE before spawning the researcher (web fetches are a side effect and "optional" means per-run consent). If skipped, the writer runs purely from the planner's outline.

### Idea-input schema (flexible)

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

### Frontmatter for idea-driven articles

```yaml
---
title: "Chosen Title"
source: "idea"
authors: []
date: YYYY-MM-DD
---
```

`authors: []` is kept for schema consistency with paper outputs.

## File-based handoffs

Every stage of the pipeline communicates via files in `/tmp/`, not inline content. The orchestrator passes file paths to each subagent, and each subagent reads its inputs with the Read tool and writes its output to a file with the Write tool. This is mandatory: large inline payloads cause 504 upstream idle timeouts on the Task and Write tools, and inline responses are unreliable for long markdown (empty or truncated responses lose the work).

The intermediate files form a chain:

| Stage      | Reads from                                              | Writes to                 |
| ---------- | ------------------------------------------------------- | ------------------------- |
| paper-reader | `/tmp/paper-source.txt`                               | `/tmp/paper-reader-output.md` |
| outliner   | `/tmp/paper-reader-output.md`, `/tmp/paper-source.txt`  | `/tmp/paper-outline.md`   |
| writer     | `/tmp/paper-outline.md`, `/tmp/paper-reader-output.md`, `/tmp/paper-source.txt` | `/tmp/paper-draft.md` |
| editor     | `/tmp/paper-draft.md`, `/tmp/paper-source.txt`          | `/tmp/paper-final.md`     |

The idea-to-substack pipeline has its own chain (no source paper):

| Stage          | Reads from                                             | Writes to                 |
| -------------- | ------------------------------------------------------ | ------------------------- |
| idea-planner   | `/tmp/idea-input.txt`                                  | `/tmp/paper-outline.md`   |
| idea-researcher (optional) | `/tmp/paper-outline.md`                     | `/tmp/idea-research-notes.md` |
| writer (idea mode) | `/tmp/paper-outline.md`, optionally `/tmp/idea-research-notes.md` | `/tmp/paper-draft.md` |
| editor (idea mode)  | `/tmp/paper-draft.md`, optionally `/tmp/idea-research-notes.md` | `/tmp/paper-final.md` |

The orchestrator writes the source text via bash (heredoc for pasted text, stdout-redirect for `pdftotext`/`pymupdf`), NOT the Write tool, because the Write tool times out on large payloads. For very large pasted text (over ~20KB), the heredoc is split across multiple `cat >>` appends. The final output file is assembled with `cat` (frontmatter + `/tmp/paper-final.md`), not re-transmitted through the Write tool. The orchestrator cleans up the `/tmp/` intermediate files after writing the final output.

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

The orchestrator extracts source text from the input before handing it to the paper-reader. All extraction writes DIRECTLY to `/tmp/paper-source.txt` via bash (stdout-redirect for `pdftotext`/`pymupdf`, heredoc for pasted text), NOT the Write tool, which times out on large payloads.

- **arXiv URL/ID**: fetch metadata via the arXiv API (`https://export.arxiv.org/api/query?id_list=<id>`), then download the PDF and convert to text with `pdftotext > /tmp/paper-source.txt` (fallback: `python3 -c "import fitz; ..." > /tmp/paper-source.txt` if pymupdf is installed).
- **Local PDF**: run `pdftotext <path> > /tmp/paper-source.txt` (or pymupdf fallback redirected to the file).
- **Raw text**: write via bash heredoc to `/tmp/paper-source.txt`. For large pasted text (over ~20KB), split across multiple `cat >>` appends.

Verify `pdftotext` is available with `command -v pdftotext`. If missing, install via `sudo apt-get install -y poppler-utils` OR fall back to `python3 -m pip install --user pymupdf` and use the fitz script below. Ask permission before installing anything.

pymupdf fallback script:

```bash
python3 -c "import sys; import fitz; doc = fitz.open(sys.argv[1]); print('\n'.join(p.get_text() for p in doc))" "<pdf_path>"
```

If the source text is over 100k characters, keep the abstract, intro, methods, results, and conclusion, and trim middle sections. Preserve the bibliography only if the article needs to reference specific prior work.

## Substack style

See `.opencode/skills/paper-to-substack/SKILL.md` for the full style guide. Key principles:

- Conversational but not dumbed down. Audience is tech-interested (follows AI, uses the tools, no CS degree). Remove equations, theorems, and formal notation; translate what the math *means* and keep practitioner-scale numbers (effect sizes, %, speedups). See the SKILL.md math handling section.
- Hook in the first 2-3 sentences.
- Use analogies for technical concepts.
- Section headings that promise value, not label topics.
- Pull quotes for memorable lines.
- End with one clear takeaway, plus a closing line pointing to the source paper for derivations.

## Usage

From inside opencode, run:

```
/paper-to-substack https://arxiv.org/abs/2401.00001
/paper-to-substack /path/to/local/paper.pdf
/paper-to-substack <paste the abstract or text of the paper>
```

Or simply talk to the default agent - it will route paper-conversion requests through the pipeline.

For idea-driven articles, run:

```
/idea-to-substack <paste a structured idea/outline, or a path to one>
```

The `/idea-to-substack` command targets the `idea-to-substack` agent. If the planner flags research gaps, the orchestrator asks once before fetching supporting material.

## Files

- `.opencode/opencode.json` - project config, sets `paper-to-substack` as the default agent.
- `AGENTS.md` - this file, project conventions for agents.
- `.opencode/agent/paper-to-substack.md` - the paper orchestrator.
- `.opencode/agent/paper-reader.md` - faithful extraction subagent.
- `.opencode/agent/outliner.md` - structure design subagent.
- `.opencode/agent/writer.md` - prose drafting subagent (generalized: source-paper and idea modes).
- `.opencode/agent/editor.md` - refinement subagent (generalized: source-paper and idea modes).
- `.opencode/agent/idea-to-substack.md` - the idea orchestrator.
- `.opencode/agent/idea-planner.md` - idea outline-normalization subagent.
- `.opencode/agent/idea-researcher.md` - optional idea research subagent (webfetch).
- `.opencode/command/paper-to-substack.md` - the `/paper-to-substack` command.
- `.opencode/command/idea-to-substack.md` - the `/idea-to-substack` command.
- `.opencode/skills/paper-to-substack/SKILL.md` - style guide and workflow (shared by both pipelines).
- `.opencode/skills/substack-headline/SKILL.md` - headline framework for post titles (3-question test, 10 formats, A/B testing and SEO-title notes). Used by the outliner (candidate titles) and editor (final title check).
