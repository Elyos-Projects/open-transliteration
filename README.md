# open-transliteration

> `open-transliteration` is an Elyos good-deed project that produces **open, machine-readable transliteration data and a reference engine** for converting text between writing systems — beginning with *  ·  **Risk tier:** med  ·  **Status:** planning

`open-transliteration` is an Elyos good-deed project that produces **open, machine-readable transliteration data and a reference engine** for converting text between writing systems — beginning with **Latin ↔ Cyrillic, Latin ↔ Devanagari, and Latin ↔ Arabic** — with a hard emphasis on **reversibility** (lossless round-trips) and **automated testing** of that property. The primary deliverables are (1) versioned, provenance-tracked **mapping datasets**, (2) a small **deterministic, rule-based reference engine** (library + CLI), and (3) a **test harness** that *proves* round-trip fidelity rather than claiming it.

**Definition of shipped:** be fully met. Securing a first adopter is the top open dependency (see Open questions and Roadmap M3).

This is an **Elyos** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/elyos

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
elyos browse
elyos next --repo Elyos-Projects/open-transliteration --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
