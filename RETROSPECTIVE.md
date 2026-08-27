# Retrospective: Written for Week 1 Me

*~650 words*

---

Hey, you're about to start the FlyRank ML internship, and honestly, you don't fully know what you're getting into. Right now you know the basics of ML and data analysis, and you're hoping this will help you understand models more deeply. That part turns out to be true, but the bigger change isn't about models at all.

**What you set out to do:** You picked Lane 3, Structured Content Archetype Clustering, because it felt like the right kind of hard: no label to lean on, no shortcut, just real data and a genuine question about what patterns actually exist in it. You'll spend eight weeks building that out, from a research question to a deployed paper built on 79 million rows of real production search data.

**What actually changed, and it's not what you'd expect:** You came in using AI the way most people do at first, weak prompts, weak answers, and worse, treating whatever came back as the finished answer instead of a draft. That's the thing that's genuinely different by the end. You stop asking "what's the answer" and start asking "is this actually right," and that shift changes everything downstream of it.

Concretely: around week 5, you'll catch a real bug in your own clustering model, a random seed that silently converged to a meaningfully worse solution, hidden inside a stability check that looked clean on the surface (a misleading mean ARI of 0.98 masking one bad seed dragging the number up while individually disagreeing with everything else). You only catch it because you insist on checking the actual pairwise numbers instead of trusting the summary statistic. Two weeks later, during final paper verification, you catch a second, unrelated bug, a baseline-lift calculation that had silently gone stale, giving you numbers that looked plausible but were wrong in a way that would have flipped your paper's headline finding if it had shipped uncorrected.

Neither of those gets caught because you got smarter at ML overnight. They get caught because you changed your posture toward AI-assisted work: you started treating every generated answer as something to verify, not something to accept. You learned to tell the difference between work you could hand off entirely (drafting SQL, building a first-pass pipeline, proposing a validation design) and work that had to stay yours no matter what (deciding whether a number was actually correct, deciding what the honest interpretation of a surprising result was). That distinction is the real skill, more than any specific clustering technique.

**What you'd build next:** if you kept going on this exact project, the natural extension is an AI-driven website auditor, something that scans an underperforming site directly and recommends concrete fixes, whether that's content quality, structure, or something else entirely, going beyond the review-queue output this capstone produces and toward something closer to a full diagnostic tool.

**The three most transferable things, and none of them are really about ML:**

1. **Verification is a habit, not a step.** The most valuable moments in this whole project were the two times a number looked right and wasn't. That doesn't happen by being careful once at the end, it happens by building the instinct to re-check anything that looks a little too clean, every time, not just when something feels obviously wrong.

2. **Knowing what to delegate and what to keep.** AI can draft code, propose a structure, catch an inconsistency you point it at, but deciding whether a result is honest, and deciding what a surprising finding actually means, has to stay yours. Getting that boundary right is a skill in itself, separate from technical ability.

3. **A caught mistake is worth more than a clean result.** Every genuinely strong part of this final paper, the split-design comparison, the corrected lift table, the honest limitations section, exists because something went wrong first and got fixed in the open, not hidden. That's true of the ML work, and it's probably true of most real work.

You came in a rookie. You're leaving with a real, defensible, independently-verified piece of work, and a completely different relationship with the tools you used to build it.
