# Building an Honest Content Archetype Model — What I Built, and the Mistake That Made It Better

I spent the last eight weeks building an unsupervised clustering model on real production
search data — 79 million raw rows, aggregated down to roughly 177,000 content items — to
discover recurring content archetypes and check whether an existing rule-based flagging
system actually targets them correctly.

**The deployed paper:** https://ojaswigautam.github.io/FlyrankAI/

---

## One real decision

I validated the model with a **client-grouped train/validation split**, not a random one.

Pages from the same client tend to share a CMS, an editorial style, and a consistent
tracking setup. A random split lets that similarity leak between training and validation,
which quietly inflates your reported numbers — the model looks like it's generalizing when
it might just be recognizing a client's house style.

I built the "before" version to prove this mattered, not just assert it. A naive random
split let 45 of 47 clients appear in *both* the training and validation sets, and it
reported a different result than the honest, zero-overlap grouped split.

**The honest number is lower.** I reported it anyway, because the point of validation is
to find out if the model actually works, not to produce the best-looking number.

---

## One real limitation

Two of the four archetypes my model discovered are partly defined by **missing data**, not
real behavioral differences.

I only found this by pulling actual example rows from each cluster and reading them
directly — not by trusting the aggregate statistics, which looked fine on their own. One
cluster's "content depth" signal turned out to be the exact same imputed placeholder value
across every row I sampled, not genuine variation in real content.

This is disclosed plainly in the paper's Limitations section. A cluster driven by a
tracking gap is a fundamentally different finding than a cluster driven by real content
behavior — and treating them the same would make the analysis look more impressive while
actually making it less useful to anyone relying on it.

---

## Links

- **Deployed paper:** https://ojaswigautam.github.io/FlyrankAI/
- **Repo (notebooks, code, every verified number):** https://github.com/OjaswiGautam/FlyrankAI
- **README:** https://github.com/OjaswiGautam/FlyrankAI/blob/main/README.md

---

*Built as part of the FlyRank ML Engineering Internship, on the FlyRank ML Internship
dataset: https://flyrank.ai*
