# paper-to-substack

An agentic framework that transforms academic papers into engaging Substack-style articles, preserving key insights while adapting tone and structure for a general audience.

This framework is built natively inside [opencode](https://opencode.ai). There is no separate CLI or library to install - everything is wired up through opencode config, agents, commands, and a skill.

## Architecture

```
[input] -> extract source text -> paper-reader -> outliner -> writer -> editor -> output/<slug>.md
```

- **paper-to-substack** (primary, default agent): orchestrates the pipeline.
- **paper-reader** (subagent): faithful extraction of claims, findings, methods, limitations.
- **outliner** (subagent): designs Substack-style structure (hook, sections, takeaways).
- **writer** (subagent): drafts engaging prose for a general audience.
- **editor** (subagent): refines hooks, headings, title, pacing, pull quotes.

## Usage

From inside opencode, run:

```
/paper-to-substack https://arxiv.org/abs/2401.00001
/paper-to-substack /path/to/local/paper.pdf
/paper-to-substack <paste the abstract or text of the paper>
```

Or just talk to the default agent - it routes paper-conversion requests through the pipeline.

Output is written to `output/<slug>.md` with YAML frontmatter (title, source, authors, date).

## Requirements

- `pdftotext` (from `poppler-utils`) for PDF extraction, **or** Python with `pymupdf` (`pip install pymupdf`) as a fallback.
- `curl` for arXiv fetches.

The orchestrator will check for `pdftotext` and fall back automatically.

## Files

- `.opencode/opencode.json` - project config, sets `paper-to-substack` as the default agent.
- `AGENTS.md` - project conventions for agents.
- `.opencode/agent/` - the orchestrator and 4 subagents.
- `.opencode/command/paper-to-substack.md` - the `/paper-to-substack` command.
- `.opencode/skills/paper-to-substack/SKILL.md` - style guide and workflow.
