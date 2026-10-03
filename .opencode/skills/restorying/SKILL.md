---
name: restorying
description: Use when the user wants to restory a text, i.e. see their own story, scene, or draft retold through other authors' styles (windows like Austen, Dickens, Dan Brown, or custom-built ones). Triggers on "retell this as", "restorying", "show my draft as Austen", "in the style of Dickens", "give me windows on this scene". NOT for producing publishable drafts, generic rewording, or paraphrasing.
---

# restorying

Retell the user's story through literary "windows": same events, different narrative choices, so the user can see new perspectives and meanings in what they wrote. Based on the method in "The House with a Million Windows: Interactive Fiction for Narrative Restorying" (Kommers, Immel, Hemment & Lee, arXiv 2609.12537), which builds on the restorying interventions of Rogers et al. 2023 ("Seeing your life story as a Hero's Journey increases meaning in life").

The core principle, from the source paper: the model is a **window, not an author**. Retellings are views to look through, never drafts to accept, never text to publish, never ingredients for a blended "improved" version. The output that matters is the user's reaction to the retelling, not the retelling itself.

## The windows

Each window is a pair: **style rules** (voice and texture) and **structure rules** (how the piece is shaped). Rules are imperatives, specific enough that two different models would produce recognizably similar retellings. After each retelling, show the rules used: transparency is part of the design (the paper's "View Inspiration" move). Never hide the trick.

### Window 1: Austen (anchor work: Pride and Prejudice, public domain)

Style rules:
1. Open with a universal-seeming truth about the story's social world, stated with mock seriousness. The retelling then complicates it.
2. Third person, free indirect style: narrate from inside the protagonist's judgments, carrying their biases as if they were reasonable.
3. Irony through understatement: vanities and social failures are reported calmly, never condemned directly.
4. Let dialogue do the characterization: people reveal themselves through politeness, deflection, and wit, not through description.
5. Translate the stakes into social currencies: status, obligation, reputation, alliance, advantage.
6. Balanced, polished sentences with antithesis (the "he was X; she was Y" shape).

Structure rules:
1. Begin with the general truth, then narrow to the particular scene.
2. Move through a misunderstanding toward a recognition.
3. End on a revised judgment: someone was wrong about someone, and now knows it.

### Window 2: Dickens (anchor works: Great Expectations, Bleak House, public domain)

Style rules:
1. The narrator addresses the reader directly and openly takes sides. Sympathy is declared, not hidden.
2. Long, cadenced sentences that accumulate concrete detail, then a very short one for the turn.
3. Introduce each character through one physical token or repeated habit (a coat, a cough, a way of counting money) that stands for their whole situation.
4. Weather, streets, rooms, and light carry the emotion. Pathetic fallacy is welcome and deliberate.
5. Institutions and objects act on people: the office, the ledger, the staircase do things to the characters.
6. Melodrama is allowed: coincidences, reunions, reversals. Sincerity over irony, always.

Structure rules:
1. Open in the middle of a scene: place before people, atmosphere before action.
2. Widen outward from the small scene to the social world around it.
3. End on an unresolved emotional hook, as if a serial installment were ending.

### Window 3: Dan Brown (paraphrased rubric only: works are in copyright, never quote text)

Style rules:
1. Short paragraphs, short chapters. Cut every scene at a reveal or a threat.
2. Open in motion: a place, a time, a body doing something precise.
3. The protagonist has expertise, and expertise narrates: confident two-sentence "facts" that reframe an ordinary detail as significant.
4. A clock is always running. Give the story an explicit deadline or countdown.
5. Escalate stakes in steps: personal, then professional, then existential.
6. Action verbs carry the sentences; ration the adjectives.
7. One unexplained detail in the story becomes a symbol someone is willing to kill to solve.

Structure rules:
1. Hook in the first line.
2. Midpoint reversal: the thing everyone assumed flips.
3. Final line is a cliffhanger, ideally under ten words.

## Workflow

### Mode A: restory a text

1. Get the text: pasted, or a file path. Any length, though roughly 100-500 words per passage works best. If the user gives a long piece, take it one scene or section at a time.
2. Confirm the windows (default: all three, one at a time). Never run them silently in parallel; the user reacts to one window at a time.
3. For each window: produce the retelling at roughly the source's length (cap 500 words), then show the rules used beneath it.
4. After each retelling, ask the diagnostic questions from the paper's expert findings:
   - What did the retelling get wrong about your story?
   - What did it emphasize that you didn't?
   - What did it leave out that you care about?
   - Which sentence, if any, do you wish you had written?
5. Close with the meaningfulness check (the paper's in-story measure, simplified): ask the user to say in one line what the story is about now, versus before the windows.
6. STOP. Do not write a next draft. Do not offer to blend the windows into a hybrid. Do not "fix" the original. The user writes the next version; the retellings were instruments for seeing, nothing more.

### Mode B: build a custom window

1. Pick an anchor work. Any era, any genre. Copyright rule: for public-domain works you may quote actual text as calibration; for in-copyright works, describe patterns in original words only, never reproduce passages.
2. Write 5-8 style rules and 2-4 structure rules as imperatives, following the specificity standard above (two models, similar retellings).
3. Calibrate the window on a story you know intimately that is NOT the user's: retell it, compare against the real author's patterns, and adjust the rules until the patterns match rather than the clichés.
4. Add the window to this file's Windows section so it persists.

## Guardrails

- **Windows, never drafts.** Retellings are not for publication and never to be merged into the user's text. If the user asks for a publishable rewrite, stop and say why: that is the failure mode the source paper was designed against (outsourcing the drafting means outsourcing the meaning-making).
- **The events never change.** Only the telling changes. If a retelling alters what happened, flag it as a bug in the retelling, not a creative upgrade.
- **Mismatch is data.** A retelling that gets the story slightly wrong is doing its job: it shows the user where they actually stand. Say this when a user objects that a window is unfair; that objection is the product.
- **Not therapy.** This is a writing practice. If the material turns toward acute personal distress, step out of the frame and say so plainly.
- **Transparency always.** Every retelling ships with its rules. A window the user cannot inspect is a trick, not a tool.
