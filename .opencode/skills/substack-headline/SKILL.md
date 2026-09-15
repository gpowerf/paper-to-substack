---
name: substack-headline
description: Use when writing, choosing, or improving the headline or title of a Substack post, including the candidate titles in the paper-to-substack pipeline. Triggers on "headline", "title options", "write a title", "better headline", "rewrite the title", "A/B test the title". Do NOT use for section headings, slugs, or Substack notes.
---

# substack-headline

A headline framework for Substack posts, adapted for research-paper articles. Based on Timo Mason and Vinayak Ramesh, "How To Write The Perfect Substack Headline Readers Physically Can't Ignore" (Write Your Way To Wealth, Sep 2026):
https://timomason.substack.com/p/how-to-write-the-perfect-substack-headline

Premise: the reader never sees the writing first. They see the headline shoulder to shoulder with ~20 other newsletter emails in an inbox. The headline's only job is to win that fight. Search is handled separately, by the SEO title (see Publishing notes at the end).

## The 3-question test

A strong headline answers three questions at once, without giving away the answer:

1. **WHAT**: what is the piece about?
2. **WHO**: who is it for? (the exact reader, not "everyone")
3. **PROMISE**: what problem gets solved, or what do they walk away with?

Answer 1 of 3 and the headline is weak. Answer 2 and it's good. Answer all 3 and readers can't help but click.

## Anatomy

- **Hook word**: the first 2-3 words do most of the work while scrolling. Lead with the most arresting element you have (a number, a contradiction, a name). "The 1 mistake..." works because "The 1" signals a low barrier to entry.
- **WHO + promise inside the first 8-10 words.** If a reader can't tell what, who, and what's in it for them by word ten, they're gone.
- **Pipeline cap: 90 characters.** Aim under 70 for mobile inbox previews.

## The 10 formats (adapted for research-paper posts)

Formats are attack angles for a boring first draft, not straitjackets. The strongest headlines combine 2-3 formats naturally: a number creates specificity, a contradiction creates tension, a clear promise gives the click a reason. If the combination feels engineered, cut back.

1. **Big numbers**: lead with the paper's most arresting real number (sessions analyzed, sample size, % effect). "I analyzed 16,000 articles to find AI writing on Substack". Never round up or approximate.
2. **Dollar signs**: money does half the work before the reader finishes the sentence. Only when the paper has real cost, salary, or savings figures; never fabricate one.
3. **Credible names**: institution, lab, or named system when genuinely central. "The Alan Turing Institute built an AI that refuses to write for you." The name has to belong there; no name-dropping.
4. **This just happened**: new paper, new release, new finding. The built-in expiry date is the point: it buys a read-now instead of a save-for-later.
5. **Success story**: the measured result visible before the click. "One session made their stories measurably more meaningful." Name the effect, not the mechanism.
6. **Things that shouldn't go together**: contradiction and tension. Usually the strongest format for a surprising finding. "This AI tool refuses to write for you. That's the whole point."
7. **Call out the exact reader**: "For Substack writers:", "If you review AI agents' work". You don't need everyone curious if the right person instantly knows the post is for them.
8. **The topic within the topic**: zoom past the field to the one specific finding. "How the paywall works on Substack (and when to use it)", not "How to make money on Substack".
9. **Question / answer**: the paper's research question, asked for real. Works best as "What nobody tells you about X" (a question wearing a different shirt). The article must actually answer it. Question titles are allowed; the style guide's no-rhetorical-questions rule applies to the article's hook, not the title.
10. **X number**: a listicle count of concrete takeaways ("10 workflow changes"). Only when the article delivers exactly that count.

## Hard guardrails

- **No clickbait.** The headline must be a promise the article fully delivers. This is already a forbidden tactic in the paper-to-substack style guide; the headline framework does not override it.
- **Numbers are verbatim from the source.** Never invent, round, or stretch a figure to fit a format. If the paper has no number worth leading with, that format is out.
- **No em dashes in titles.**
- **Headline is not SEO title.** The headline wins the inbox with curiosity; don't cram keywords into it.

## Workflow

### Mode A: generate headlines for an article

Input: a draft or finished article (file path), or an outline with candidate titles.

1. Extract the three answers in one sentence each: WHAT (core finding), WHO (exact reader), PROMISE (what they walk away with).
2. List the article's 2-3 most arresting real elements: a number, a contradiction, a name, a timely angle.
3. Write a boring first-draft headline, then rewrite it in 3-4 DIFFERENT formats (that count matches Substack's A/B test limit). Layer formats where natural.
4. Score each candidate against the 3-question test and the anatomy rules. Kill anything that answers only one question.
5. Deliver: candidates ranked best-first, each tagged with its format(s), plus a separate SEO title (topic-explicit, keyword-carrying) for Substack's SEO-title setting.

SEO pairing example from the source article: headline "How I Got 352 Subs From One Substack Feature 99% Of Creators Ignore", SEO title "Substack Recommendations: How To Get More Subscribers".

### Mode B: audit or fix an existing headline

1. Run the 3-question test; name which question(s) go unanswered.
2. Check anatomy: what are the first 2-3 words? Do WHO and promise land inside the first 8-10 words?
3. Diagnose the failure (vague topic, no reader, no promise, buried number) and rewrite in 2-3 different formats.
4. Show before/after, each option tagged with its format(s).

## Where this plugs into the pipeline

- **outliner**: the 3 candidate titles are 3 different formats from this skill (instead of the old analytical/provocative/plain trio), each tagged and 3-question-tested.
- **editor**: the chosen title must pass the 3-question test; if it fails, sharpen it within its format per the anatomy rules.
- **orchestrator**: when reporting the final article, up to 4 headline variants can be offered for Substack's built-in title A/B test. The feature unlocks at 200+ subscribers, works only in the web editor, and in nearly 60% of Substack's own tests the winning headline wasn't the writer's original pick (per the source article).
- **Publishing**: before sending, set the SEO title separately in Substack's post settings. Headline for the inbox, SEO title for the search bar.
