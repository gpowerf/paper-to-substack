# paper-to-substack

An agentic framework that transforms academic papers into engaging Substack-style articles, preserving key insights while adapting tone and structure for a general audience. It also generates Substack articles from structured ideas or outlines (no source paper required).

This framework is built natively inside [opencode](https://opencode.ai). There is no separate CLI or library to install - everything is wired up through opencode config, agents, commands, and a skill.

## Architecture

```
[input] -> extract source text -> paper-reader -> outliner -> writer -> editor -> output/<slug>.md
            /tmp/paper-source.txt   /tmp/paper-reader-output.md  /tmp/paper-outline.md  /tmp/paper-draft.md  /tmp/paper-final.md
```

- **paper-to-substack** (primary, default agent): orchestrates the pipeline.
- **paper-reader** (subagent): faithful extraction of claims, findings, methods, limitations.
- **outliner** (subagent): designs Substack-style structure (hook, sections, takeaways).
- **writer** (subagent): drafts engaging prose for a general audience.
- **editor** (subagent): refines hooks, headings, title, pacing, pull quotes.

## Architecture: idea-to-substack

```
[structured idea/outline] -> /tmp/idea-input.txt -> idea-planner -> [idea-researcher] -> writer -> editor -> output/<slug>.md
                                                 /tmp/paper-outline.md   /tmp/idea-research-notes.md  /tmp/paper-draft.md  /tmp/paper-final.md
```

- **idea-to-substack** (primary agent): orchestrates the idea pipeline.
- **idea-planner** (subagent): normalizes a user outline into the Substack outline format, flags research gaps.
- **idea-researcher** (subagent, optional): fetches supporting links/facts via webfetch when gaps are flagged.
- **writer** / **editor** (generalized): run in "idea mode" when no source paper is provided.

The writer and editor are shared by both pipelines and switch behavior based on whether a source paper path is present.

## Usage

From inside opencode, run:

```
/paper-to-substack https://arxiv.org/abs/2401.00001
/paper-to-substack /path/to/local/paper.pdf
/paper-to-substack <paste the abstract or text of the paper>
```

Or just talk to the default agent - it routes paper-conversion requests through the pipeline.

For idea-driven articles, run:

```
/idea-to-substack <paste a structured idea/outline, or a path to one>
```

If the planner flags research gaps, the orchestrator asks once before fetching supporting material.

Output is written to `output/<slug>.md` with YAML frontmatter (title, source, authors, date).

## Requirements

- `pdftotext` (from `poppler-utils`) for PDF extraction, **or** Python with `pymupdf` (`pip install pymupdf`) as a fallback.
- `curl` for arXiv fetches.

The orchestrator will check for `pdftotext` and fall back automatically.

## Files

- `.opencode/opencode.json` - project config, sets `paper-to-substack` as the default agent.
- `AGENTS.md` - project conventions for agents.
- `.opencode/agent/` - the orchestrators and subagents (paper-reader, outliner, writer, editor, idea-planner, idea-researcher).
- `.opencode/command/paper-to-substack.md` - the `/paper-to-substack` command.
- `.opencode/command/idea-to-substack.md` - the `/idea-to-substack` command.
- `.opencode/skills/paper-to-substack/SKILL.md` - style guide and workflow (shared by both pipelines).
