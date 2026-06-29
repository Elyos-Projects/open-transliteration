# PLAN — open-transliteration

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: J. Carter (acting maintainer) · Lane: donated · Risk tier: medium

> **Positioning:** The open, **reversible**, **tested** commons for moving text between scripts —
> Latin ↔ Cyrillic, Devanagari, Arabic and beyond — built from openly-licensed and public-domain
> sources, with round-trip fidelity that is *proven by tests*, not asserted. Where the world's best
> transliteration data is locked inside copyrighted standards (ISO, DIN) or buried in large libraries
> without clean reversibility guarantees, open-transliteration ships the mappings, the reference
> engine, and the proof — all freely reusable.

> **In one line:** anyone can romanize text; almost no one ships an *openly-licensed* scheme that you
> can run *backwards* and verify character-for-character. That gap is the project.

## Executive summary

`open-transliteration` is an Elyos good-deed project that produces **open, machine-readable
transliteration data and a reference engine** for converting text between writing systems —
beginning with **Latin ↔ Cyrillic, Latin ↔ Devanagari, and Latin ↔ Arabic** — with a hard emphasis
on **reversibility** (lossless round-trips) and **automated testing** of that property. The primary
deliverables are (1) versioned, provenance-tracked **mapping datasets**, (2) a small **deterministic,
rule-based reference engine** (library + CLI), and (3) a **test harness** that *proves* round-trip
fidelity rather than claiming it.

The work runs in the **donated lane**: a human runs their own coding agent interactively to draft
mappings, rules, tests, and docs, then opens PRs; the Elyos CLI only prepares workspaces and opens
PRs. Because correct transliteration **needs domain accuracy** (native-script and
linguistic competence per script pair), the project is **medium risk tier**: every scheme is gated
by a **qualified script/linguistics reviewer** plus a **per-source license check** before it can be
marked delivered.

The **defining constraint** of this project — its identity — is **license and provenance discipline
around standards**. The authoritative descriptions of most transliteration schemes (ISO 9, ISO 233,
ISO 15919, DIN 31635) are **copyrighted documents sold by standards bodies**; their *tables must not
be copied verbatim* into an open repository. The mapping **facts** (which grapheme maps to which) are
not themselves copyrightable, but a particular table's selection and arrangement can attract thin
compilation rights, and slavish copying is both legally risky and ethically wrong. We therefore
**build every mapping from openly-licensed or public-domain sources** (UNGEGN/UN romanization
systems, BGN/PCGN and other US-government works, Unicode CLDR/ICU transform data, CC-licensed
references), **re-derive** schemes where only a paywalled standard describes them, and **cite the
standard as a conformance reference — never as a copied source.** A scheme whose provenance cannot
be cleanly established and openly licensed is **not shipped.**

Honest status note: this is **public-good infrastructure with no specific adopting partner yet
secured.** The *category* need is well-evidenced (see below), but a **named first adopter** — a
Wikimedia tool, a library/GLAM cataloguing system, an NLP pipeline, or a language-community
organization — is **TO BE SECURED**. Until one is, `verifiedNeed` is recorded as `false` on tasks
whose value depends on a named beneficiary, and the project's Definition of Shipped (adoption) cannot
be fully met. Securing a first adopter is the top open dependency (see Open questions and Roadmap M3).

**Locked build decisions** (constraints chosen up front, so later phases inherit them):
- **Data-first, rule-based, deterministic.** Mappings are declarative, versioned data; the engine is
  a thin, deterministic interpreter. **No machine learning, no statistical guessing** in the core —
  reversibility must be provable, which rules out non-deterministic models.
- **Reversibility is a typed, tested property**, not a marketing word. Every scheme is tagged
  `reversibility: strict | normalized | lossy`, and the tag is **enforced by the test harness**.
- **Unicode-correct by construction.** All processing operates on **NFC-normalized text segmented
  into extended grapheme clusters**; the normalization contract is part of every scheme's definition.
- **Stack:** TypeScript + ESM, pnpm workspaces; data in YAML (authored) compiled/validated to JSON;
  JSON Schema (AJV draft-07) in `packages/schema`; property-based testing (`fast-check`) + golden
  vectors. **Code: MIT. Data: CC-BY-4.0** (with CC0 considered for pure factual mapping tables).
- **Cold-start on the easiest *provably-reversible* case first:** **Cyrillic ↔ Latin via an ISO 9
  System A–conformant bijective scheme**, because a strict character-for-character bijection is the
  cleanest way to stand up and trust the round-trip harness before tackling harder scripts.

## Problem & beneficiaries

**Who is helped (direct, technical beneficiaries).** Builders of public-interest software and
knowledge infrastructure who need to move text reliably between scripts:
- **Librarians / cataloguers (GLAM)** producing romanized bibliographic records.
- **Wikimedia / Wikipedia / Wikisource editors and tools** that transliterate names and terms.
- **NLP and digital-humanities researchers** needing reproducible, openly-licensed romanization for
  corpora, search, and indexing.
- **Search/indexing and accessibility systems** that match queries across scripts.
- **Language-learning, keyboard/input-method, and language-revitalization tool builders.**

**Who is ultimately helped (end beneficiaries).** Speakers and readers of **under-served-language
communities** and **diaspora/migrant populations** whose names and text are routinely **mangled by
ad-hoc, lossy, inconsistent romanization** — in official documents, search results, citations, and
databases. A *reversible* scheme means a person's name romanized for an English-only system can be
**recovered exactly** in the original script; a *tested* scheme means the recovery is trustworthy.

**The need (category-level, well-evidenced).** Good transliteration data exists, but the openly
reusable, *reversible*, *tested* slice is thin:
- The authoritative schemes (ISO 9/233/15919, DIN 31635, ALA-LC) are **copyrighted or
  access-restricted**, so they cannot be redistributed as open data, and most are distributed as
  human-readable PDFs, not validated machine-readable mappings.
- Common romanizations (BGN/PCGN, popular ad-hoc schemes) are **pronunciation-oriented and lossy** —
  you cannot recover the original script — yet they are used as if they were lossless.
- Existing open engines (ICU/CLDR transforms, `uroman`, Aksharamukha) are valuable prior art but
  vary in license, reversibility guarantees, and *automated round-trip proof*. There is no clean,
  small, **provenance-and-license-audited, reversibility-typed, test-proven** commons.

The gap this project fills is precisely **open + reversible + tested + provenance-clean**, together.

**Verified need: TO BE SECURED.** The infrastructure gap is real and documented, but a **specific
adopting project/organization** that will integrate and depend on these datasets is **not yet
secured**. Tasks are written so the foundations (schema, engine, first schemes, harness) can be built
*without* a partner, while flagging that **adoption/delivery tasks are blocked** until a named adopter
is confirmed (`verifiedNeed: false` until then).

**Partner / adopter: TO BE SECURED.** Candidate adopter types: Wikimedia Foundation / Wikimedia
language tooling, library/GLAM systems and ALA/IFLA working groups, Unicode CLDR (as an upstream/
peer), Translators without Borders / CLEAR Global, academic NLP groups, and language-revitalization
NGOs. None is committed as of this draft.

## Goals and non-goals

**Goals**
- Ship **openly-licensed, machine-readable, versioned mapping datasets** for Latin ↔ Cyrillic,
  Devanagari, and Arabic, each with **clean provenance** and a declared **reversibility class**.
- Ship a **small deterministic reference engine** (TS/ESM library + CLI) that applies the data
  faithfully in both directions.
- **Prove reversibility by automated tests**: per-scheme **property-based round-trip tests** over open
  corpora and code-point ranges, **golden test vectors**, and a **static collision/bijection checker**
  that flags whether a scheme can actually be strictly reversible.
- Establish a **provenance + license model** that lets anyone audit where each mapping came from and
  reuse it safely.
- Provide **qualified per-script reviewer sign-off** (medium risk tier) on every scheme.
- Get the datasets **adopted by at least one real downstream project** (the true success event).

**Non-goals (constraints create identity — this project refuses, on principle)**
- **Not a phonetic transcription or pronunciation service.** We do graphemic *transliteration* and
  label pronunciation-oriented schemes honestly as **lossy / non-reversible**; we do not pretend a
  pronunciation romanization is reversible.
- **Not a machine-translation tool.** Transliteration changes *script*, not *language*. We never
  translate meaning.
- **No ML / statistical guessing in the reversible core.** Determinism is a hard requirement;
  reversibility cannot rest on a model's best guess.
- **We do not copy or redistribute copyrighted standards.** No verbatim ISO/DIN/ALA-LC tables. We
  cite conformance; we re-derive from open sources.
- **Not a certified, official, or legal transliteration service.** Our output is **not** a certified
  translation, a legal name change, or an official document; deliverables carry that disclaimer.
- **No surveillance / identity-matching tooling.** We will not build or optimize for
  name-deanonymization, cross-script person-matching for tracking, or any identification use.
- **We do not adjudicate politically contested romanizations or place names.** We present
  standards-conformant schemes neutrally, document alternatives, and use neutral/PD examples; we take
  no position on politically loaded script or naming disputes.
- **Not** a hosted website/portal, a font/rendering project, or an OCR/handwriting project (those are
  separate Elyos tracks).

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Baselines are ~0 (new project). **Outcome** targets are for
the first ~6 months after a first adopter is secured; **interim foundation metrics** are tracked from
M0 so progress is visible *before* an adopter exists. We explicitly **do not** count "PRs merged,"
"lines of mapping," or "commits" as success.

**Outcome metrics (post-adopter)**

| Metric | Baseline | Target | How measured |
|---|---|---|---|
| Datasets **adopted/integrated by a downstream project** | 0 | ≥ 1 project integrates ≥ 1 scheme | Adopter written confirmation in PR/receipt |
| Script pairs delivered **end-to-end** (provenance-clean → reviewed → tested → released) | 0 | ≥ 3 (Cyrillic, Devanagari, Arabic ↔ Latin) | Project registry |
| **Round-trip fidelity** on a held-out open corpus, for schemes tagged `strict` | n/a | **100%** (any failure = bug, blocks release) | Property + corpus tests in CI |
| Documented **loss profile** for schemes tagged `normalized`/`lossy` | n/a | Published & accurate for 100% of such schemes | Per-scheme loss report |
| **License/provenance compliance** of released schemes (no copyrighted-table copying; source + license recorded) | n/a | 100% (hard gate) | License-check task / CI per scheme |
| **Mapping defects found after release** (incorrect grapheme mapping) | n/a | 0 critical; trend down | Post-release defect log |
| Adopter-reported **usefulness** (qualitative) | n/a | Positive from ≥ 1 adopter | Adopter feedback |

**Interim foundation metrics (M0–M2, adopter-independent)**

| Metric | Baseline | Target | How measured |
|---|---|---|---|
| Open/PD **sources verified** and recorded on the provenance allow-list | 0 | ≥ 4 by end of M0 | Allow-list entries with `verifiedBy`/`verifiedDate` |
| Schemes with a **passing round-trip harness** (per their declared class) | 0 | ≥ 1 (M0), ≥ 2 (M1), ≥ 3 (M2) | CI test results |
| **Golden test vectors** per scheme | 0 | ≥ 50 per scheme | Vector file counts |
| Per-script **qualified reviewers** onboarded | 0 | ≥ 2 by end of M1 | Reviewer roster |
| **Collision/bijection checker** coverage (schemes analyzed) | 0 | 100% of `strict`-tagged schemes | Checker report in CI |

**Round-trip fidelity — exact definition (so "100%" is honest).** For a scheme tagged `strict`, the
test asserts `reverse(forward(s)) == NFC(s)` for **every** `s` in (a) curated golden vectors, (b) an
open PD/CC corpus in the source script, and (c) property-generated valid grapheme sequences over the
script's code-point inventory. For `normalized` schemes the equality target is a **declared
normalized form** (documented per scheme), not byte-identity. For `lossy` schemes round-trip equality
is **not** asserted; instead a **loss report** enumerates exactly what information is dropped. The
"100%" headline applies to `strict` schemes; below a sample of < 1,000 generated cases per scheme we
also report raw pass counts to avoid over-claiming on small samples.

## Scope

**Definition — "reversibility class" (objective, three tiers).**
- **`strict` (bijective):** `reverse(forward(s)) == NFC(s)` for all valid source text; the inverse is
  unambiguous (no two source tokens map to the same target token). Goal class for Cyrillic ↔ Latin
  (ISO 9 System A–conformant).
- **`normalized`:** round-trips to a **declared normalized form** of the source (e.g., recovers NFC,
  collapses presentation-form variants, restores canonical conjunct ordering) — losslessly *up to that
  documented normalization*. Expected class for much of Devanagari ↔ Latin.
- **`lossy`:** information is intentionally dropped (e.g., unwritten short vowels in Arabic,
  pronunciation-oriented romanizations). **Tagged `lossy`, never claimed reversible**, and shipped
  with a loss report.

**Prioritization rule (which script pair next).** Order: (1) a **secured adopter's** stated need;
(2) the pair with the **cleanest open/PD provenance** and a **genuinely reversible standard** to
conform to; (3) **reviewer availability** for the pair; (4) **size of the under-served population**
poorly served by existing open data. Cold-start (M0) follows rule (2): **Cyrillic ↔ Latin**, where a
strict bijection is achievable and the harness can be proven before harder scripts.

**In scope**
- **Mapping datasets** (YAML→JSON) per scheme: ordered, context-aware rewrite rules, reversibility
  class, normalization contract, provenance, license.
- A **deterministic reference engine** (forward + inverse), library + CLI, in `packages/`.
- **Test harness**: golden vectors, property-based round-trip tests, collision/bijection checker,
  conformance tests against *open* example sets (e.g., UNGEGN sample words).
- **Provenance allow-list** recording each source's license, URL, version, retrieval date, snapshot
  hash/archive.
- **Per-scheme docs**: what standard it conforms to (cited, not copied), reversibility class, loss
  report (if any), usage, and the "not a certified/official transliteration" disclaimer.

**Out of scope**
- Phonetic/IPA transcription, pronunciation audio, or speech.
- Machine translation of meaning.
- Copying or redistributing any **copyrighted standard's tables** (ISO/DIN/ALA-LC text).
- ML-based or non-deterministic transliteration in the reversible core.
- A hosted website, API service, font/rendering, OCR, handwriting, or keyboard-layout deliverables.
- **Schemes for which no clean open/PD provenance exists** (we re-derive from open sources or skip).
- **Politically contested romanization adjudication** or contested place-name positions.
- Personal-name datasets of real, identifiable individuals (privacy — see below).

## Solution approach & architecture

Primarily a **data + tooling** project: the durable public good is the **mapping datasets and their
proof of correctness**; the engine is the reference implementation that makes the data runnable and
testable. It rides existing Elyos donated-lane mechanics (CLI prepares workspace, human runs agent,
PR opened, human/expert review gates "done").

**Components**
1. **`packages/schema`** — JSON Schemas (AJV draft-07) for the new artifacts: `mappingSchemeSchema`,
   `mappingRuleSchema`, `provenanceSchema`, `reviewSchema`, exported and compiled exactly like the
   existing `taskSchema`/`registrySchema`. Keeps validation **agent-neutral** in core, not adapters.
2. **`packages/core` (transliteration engine)** — a deterministic interpreter:
   `transliterate(text, schemeId, direction)`, `loadScheme(id)`, `checkReversible(schemeId)`. Pipeline:
   **NFC-normalize → segment into extended grapheme clusters → apply ordered context-sensitive rules →
   emit**, with a symmetric inverse path. No vendor/agent-specific logic; no ML.
3. **CLI surface** — a thin command exposing the engine for batch use (`transliterate <scheme>
   <direction> < in > out`) and `check-reversible <scheme>`.
4. **`data/schemes/`** — the authored YAML mapping datasets (the primary deliverable), compiled to
   validated JSON.
5. **Test harness** — golden vectors (`data/vectors/`), property tests (`fast-check`), the
   collision/bijection checker, and conformance tests against open example sets; all wired into
   `pnpm test` so **CI fails on any reversibility, schema, or provenance violation**.

**Data model (mapping scheme).**
```yaml
id: cyrl-latn-iso9a
name: Cyrillic ↔ Latin (ISO 9 System A–conformant, bijective)
sourceScript: Cyrl          # ISO 15924 code
targetScript: Latn
direction: bidirectional
conformsTo: "ISO 9:1995 System A (cited as reference; table re-derived from open sources)"
reversibility: strict        # strict | normalized | lossy
normalizationForm: NFC
provenanceRefs: [unggn-cyrillic, cldr-cyrl-latn]   # ids into provenance allow-list
license: CC-BY-4.0
rules:                        # ordered; first match wins within a grapheme cluster
  - from: "Ж"; to: "Ž"; reversibleInverse: true
  - from: "ж"; to: "ž"; reversibleInverse: true
  - from: "Ц"; to: "C"; contextAfter: "[ИЕ]"; reversibleInverse: true   # context-sensitive
  # ...
lossReport: null             # required (non-null) for normalized/lossy schemes
```
Each **rule** is `{ from, to, contextBefore?, contextAfter?, reversibleInverse }`. For `strict`
schemes the **collision/bijection checker** statically verifies every target token has exactly one
inverse; ambiguity downgrades the achievable class and **fails** the `strict` claim.

**Reversibility — how it is actually proven (the technical heart).**
- **Static check (collision/bijection checker):** analyze the rule set for many-to-one target
  collisions and unresolved context overlaps; emit a report of whether `strict` is achievable.
- **Golden vectors:** curated input↔output pairs from *open* authoritative examples per scheme.
- **Property-based round-trip:** generate valid source grapheme sequences across the script's
  Unicode ranges, assert `reverse(forward(s)) == target-form(s)` per the scheme's class.
- **Open-corpus round-trip:** run a PD/CC corpus in the source script through forward+inverse and
  assert recovery (per class).
- **Known hard cases encoded as explicit tests:** Unicode normalization (NFC/NFD), combining marks,
  Cyrillic case, Greek/Cyrillic final-form analogues, Devanagari conjuncts/virama/anusvara/nukta and
  schwa handling, Arabic short-vowel omission and sun/moon letters, digits (Arabic-Indic ٠١٢٣ and
  Devanagari ०१२३), punctuation/whitespace/foreign-text passthrough, and homoglyph/confusable output.

**Key decisions**
- **Provenance-as-data, license-as-data.** Every scheme links to provenance entries that record the
  open/PD source and license; a CI gate refuses any scheme lacking clean, recorded provenance.
- **Re-derive, never copy.** Where only a paywalled standard describes a scheme, we **construct** the
  mapping from open sources and **cite** the standard for conformance; we never paste its table.
- **Reversibility class is declared and enforced** — the harness fails a `strict` claim the checker
  can't support.
- **Determinism over coverage** — we would rather ship fewer, provably-correct schemes than many
  guessy ones.

## Data, licensing & compliance

**This is the project's most important section. Be conservative; when provenance or license is
unclear, do not ship the scheme.**

**The core legal nuance.** A transliteration *scheme* is a **system of mapping facts**; under
copyright doctrine (e.g., facts and systems are not protected; merger doctrine), the bare fact "this
grapheme corresponds to that grapheme" is **not copyrightable**. However: (a) a standards body's
**published table** (its selection, arrangement, prose, and the document itself) is a **copyrighted
work** that we must not reproduce; (b) some jurisdictions grant **thin compilation/database rights**
(notably the EU *sui generis* database right) over a substantial table even of facts. Our stance is
deliberately conservative: **we do not copy any standard's table; we build mappings from openly-
licensed or public-domain sources and cite the standard only as a conformance reference.**

**Approved source classes (provenance allow-list).** Only sources with verified open/PD/CC/OFL terms
may seed a mapping, each verified and recorded before use:
- **UNGEGN / UN romanization systems for geographical names** — published by the UN, freely available;
  authoritative and citable. **Verify the specific document's reuse terms.**
- **BGN/PCGN and other US-government romanization works** — generally **US public domain** (verify per
  document; watch for embedded third-party material).
- **Unicode CLDR / ICU transform data** — **Unicode License** (permissive, attribution); excellent
  open prior art and a reference/comparison baseline.
- **Open-source transliteration projects** (e.g., `uroman`, Aksharamukha) — **only** where their
  license permits reuse and is recorded; used as references/cross-checks, with license honored.
- **CC-BY / CC0 / public-domain reference material** (e.g., openly-licensed linguistics references,
  PD academic conventions such as IAST for Sanskrit, which is a scholarly convention, not a
  copyrighted table).
- **Open / PD test corpora** for vectors — e.g., Universal Declaration of Human Rights translations,
  Wikisource PD texts, UNGEGN sample word lists. **No scraped or personal data.**

For **each** source we record: id, name, canonical URL, version/date, retrieval date, license name +
URL, a **snapshot of the license text**, whether derivatives are permitted, required attribution
string, and notes. **SHA-256 `snapshotHash`** + a web-archive (`snapshotArchiveUrl`) where available
make later drift detectable; a **source-change check** (minimal/manual in M0, automated in M1)
re-fetches and flags drift.

**BLOCKING prerequisite for any scheme task.** Before a scheme's data task may start, the maintainer/
license reviewer must confirm and record on the provenance allow-list: (a) the mapping can be
**built from open/PD sources** (or re-derived from first principles + open references), **without
copying a copyrighted table**; and (b) the **output license** (CC-BY-4.0 for data, MIT for code) is
**compatible** with every source used. This is a **hard gate, not an open question** — if either is
unconfirmed, the scheme is **not built** until it is.

**Explicitly excluded sources.** **ISO standard documents and tables, DIN 31635, and the ALA-LC
romanization tables as published** are **not** copied or redistributed. We may *cite* "conformant to
ISO 9 System A" and *re-derive* an equivalent mapping from open sources; we never paste their content.
Any scheme that *cannot* be reconstructed from open sources is **deferred or dropped**, not copied.

**Output licensing.** **Code/engine/tooling: MIT.** **Mapping data and docs: CC-BY-4.0** (with
**CC0** considered for the pure factual mapping tables, to maximize reuse, pending maintainer
decision — see Open questions). Attribution to the open sources used is recorded per scheme. We do
**not** relicense or "free" anything we did not have the right to use.

**Privacy / PII.** The tool is script-neutral, but transliteration is frequently applied to **personal
names** — a sensitive, identity-linked use. Therefore: (a) **test/example corpora use PD/openly-
licensed text and geographic-name sample lists, never datasets of real, identifiable individuals**;
(b) we ship **no personal-name dataset**; (c) we add a **"not a certified/official transliteration"
disclaimer** so output is not mistaken for legal identity documentation; (d) we will **not** build or
optimize cross-script person-matching/deanonymization (refusal guardrail). No end-user data is
collected.

**Neutrality.** Romanization is sometimes politically charged (contested scripts, contested place
names). We present **standards-conformant** schemes neutrally, document alternatives without endorsing
one as politically "correct," and choose **neutral examples**; we take no position on script/naming
disputes.

## Quality, review & risk gates

**Risk tier: medium.** Per the good-deed definition, medium = needs **domain accuracy** and a
**reviewer with the relevant skill** — here a **qualified reviewer with native/near-native command of
the script and linguistic/transliteration competence** for the specific script pair. (This is *not*
high risk: transliteration is not medical/legal/safety advice. But a *misleading reversibility claim*
or a *mojibake'd* name is a real harm, so review and tests are mandatory.)

**Required review before a scheme is "done"**
1. **Automated harness green** — schema validation, **collision/bijection checker** consistent with
   the declared class, **golden vectors**, **property-based round-trip**, and **open-corpus
   round-trip** all passing in CI. A `strict` claim the checker cannot support **fails the build.**
2. **Agent self-check** with explicit uncertainty flags (below) — first pass, not sufficient alone.
3. **Qualified per-script reviewer sign-off** recorded in the PR (mapping correctness, edge cases,
   reversibility-class honesty, neutrality of examples, loss report accuracy).
4. **License/provenance verification** — every source recorded, open/PD/CC/OFL, no copyrighted-table
   copying; output license compatible.
5. **Maintainer approval** of the PR.

**Reviewer independence & second-reviewer rule.** The human who ran the drafting agent **may not** be
the sole sign-off reviewer for that scheme; each reviewer records a **conflict-of-interest
declaration**. A **second independent qualified reviewer is mandatory for any scheme claiming
`strict` reversibility** (because a false bijection claim silently corrupts data downstream) and for
the **first scheme in each new script family**. Disagreements: reconcile against open sources and the
checker → escalate to a third reviewer/maintainer → **the more conservative reversibility class wins**
(e.g., downgrade `strict`→`normalized`) and the result is recorded in `review.yaml`.

**Agent uncertainty self-check (operationalized).** The drafting agent must emit explicit flags —
`UNCERTAIN: <grapheme/rule> | <type: mapping|context|reversibility|normalization|provenance|neutrality>
| <note>` — for anything it is unsure about. These are copied into `review.yaml` as `agentFlags`. **No
sign-off may be recorded while any flag is unresolved**; each must be `resolved` (with adjudication) or
`accepted-as-is` with reasoning.

**Definition of Shipped (project-specific).** A scheme is *shipped* only when: acceptance criteria met
**and** the full automated harness is green **and** the declared reversibility class is honest and
checker-supported **and** qualified per-script reviewer sign-off is recorded **and**
license/provenance verified (no copyrighted-table copying) **and** docs + (where applicable) loss
report present **and** the dataset is **adopted/integrated by at least one real downstream project.**
Released-but-unadopted is **not** shipped (it is "released," a lesser state we track separately).

## Roadmap & milestones

**M0 — Foundation & cold-start: prove the harness on a strict bijection (no partner required).**
Goal: stand up the schema, engine, provenance model, and **round-trip test harness**, and prove them
on **one strictly-reversible scheme (Cyrillic ↔ Latin, ISO 9 System A–conformant)** end-to-end except
adoption.
Exit criteria: provenance allow-list with ≥ 4 verified open/PD sources (snapshot hash/archive + a
minimal source-change re-fetch check); `mappingScheme`/`mappingRule`/`provenance`/`review` JSON
Schemas merged and CI-validated; the deterministic engine (forward + inverse) merged; the
**collision/bijection checker** + **property-based round-trip** + **golden vectors** harness merged
and green; the **BLOCKING license/provenance prerequisite confirmed** for the first scheme **before
authoring**; **one `strict` Cyrillic ↔ Latin scheme passing 100% round-trip** on golden vectors +
property tests + an open corpus, with **qualified reviewer sign-off** and a passing license check.
`verifiedNeed` honestly `false` (no adopter). The "100% / 0-defect" headline metrics are **effective
from M0 for `strict` schemes** (the whole point of starting with a bijection is that they are provable
on day one).

**M1 — Repeatability & second script family (Devanagari ↔ Latin; reviewer network).**
Goal: generalize the engine to richer **context-sensitive rules, conjuncts, and the `normalized`
class**, add a second scheme, and build the reviewer network.
Exit criteria: engine handles Devanagari grapheme clusters (virama/conjuncts, anusvara/visarga,
nukta, schwa handling) and the `normalized` reversibility class with a **documented normalization
contract + loss report**; **one Devanagari ↔ Latin scheme (ISO 15919 / IAST–conformant)** released
with reviewer sign-off and license-clear provenance; **automated source-change watcher** operating;
documented **reviewer-qualification criteria + ≥ 2 named qualified reviewers** (per-script); license/
provenance check formalized as tooling in CI.
Dependency: reviewer sourcing.

**M2 — The hard case: Arabic ↔ Latin (honest reversibility).**
Goal: tackle the hardest pair, where unwritten short vowels make naive romanization `lossy`, and ship
it with **honest tagging**.
Exit criteria: an Arabic ↔ Latin scheme released that is **reversible for fully-vocalized input**
(declared `strict` or `normalized` on vocalized text) and **explicitly `lossy` with a published loss
report for unvocalized input**; sun/moon letters, hamza/madda, ta marbuta, and Arabic-Indic digits
handled and tested; second independent reviewer + a back-mapping QA pass on any `strict` claim.
Dependency: M0/M1 + Arabic-script reviewer.

**M3 — First adopter (needs partner).**
Goal: get the datasets **integrated and depended upon** by a real downstream project.
Exit criteria: a named adopter secured (`verifiedNeed = true`); ≥ 1 scheme **integrated and confirmed
in use** by the adopter; an adopter-facing **stable data contract/versioning policy** published. This
is the first true Definition-of-Shipped event.
Dependency: M0–M2 + adopter.

**M4 — Scale & sustain.**
Goal: broaden coverage with sustained quality and clear maintenance.
Exit criteria: ≥ 3 script pairs released and ≥ 1 adopted across them; clearly-tagged `lossy`
pronunciation schemes added where useful (never mislabeled); reviewer rotation; outcome tracking
(post-release defect + adopter-feedback log) operating; semantic-versioning + deprecation policy for
data; named sustainability owner.

## Work breakdown

The itemized, sized backlog lives in **[TASKS.md](./TASKS.md)**, organized by the milestones above
(M0–M4) plus a Backlog/future section. Each task maps to an Elyos Task JSON (see the schema in
`packages/schema/src/schemas.ts`) with id, type, lane, risk tier, deliverable, acceptance criteria,
and license fields. M0–M2 tasks are partner-independent foundations; M3+ tasks are gated on a secured
adopter and are marked accordingly (`verifiedNeed: false` until then).

## Governance, roles & stakeholders

- **Maintainer (Owner): J. Carter (acting)** — accepts/sequences tasks, approves PRs, owns provenance
  allow-list integrity and the license gate; acts as interim license/compliance reviewer until a
  dedicated reviewer is named.
- **Qualified per-script reviewers: TO BE SECURED** — native/near-native + linguistic competence per
  script (Cyrillic, Devanagari, Arabic); sign-off recorded in PRs; rotation defined in M1/M4.
- **License/compliance reviewer** — may be the maintainer initially; verifies open/PD provenance, no
  copyrighted-table copying, and output-license compatibility; escalates ambiguous IP questions.
- **Steward (last-mile owner): TBD — named by end of M1** (acting maintainer holds these duties
  interim). Owns the adopter relationship and confirms **integration/adoption** (without this, nothing
  reaches "shipped"). Naming a steward is a **prerequisite for entering M3**.
- **Adopter / requestor: TO BE SECURED** — downstream project/org that integrates and depends on the
  datasets and confirms adoption.
- **Expert reviewers** — for any drift toward high-stakes use (e.g., legal-document identity
  transliteration), require the relevant credentialed expert *or* reject as out of scope and rely on
  the "not certified/official" disclaimer.

## Dependencies & integrations

- **Elyos donated lane**: `packages/cli` (workspace prep + PR), `packages/core`, `packages/schema`
  (Task JSON). No funded-lane / API-key execution in this project.
- **Open/PD source references**: UNGEGN/UN romanization systems, BGN/PCGN & US-gov works, Unicode
  CLDR/ICU transform data, openly-licensed OSS transliteration projects (license-verified).
- **Open/PD test corpora**: UDHR translations, Wikisource PD texts, UNGEGN sample word lists.
- **Tooling**: TypeScript/ESM, pnpm; AJV + `ajv-formats`; `fast-check` (property testing); Unicode
  normalization + grapheme segmentation (e.g., `Intl.Segmenter` / a vetted grapheme library).
- **Qualified per-script reviewers** (individuals or a language/translation org) — external
  dependency, not yet secured.
- **Adopter** (Wikimedia tool, GLAM system, NLP group, language NGO) — not yet secured.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Copying a copyrighted standard's table (ISO/DIN/ALA-LC) | Medium | High | Build from open/PD sources only; re-derive + cite conformance; license gate refuses unclean provenance; explicit excluded-source list | License reviewer / Maintainer |
| **False reversibility claim** silently corrupts downstream data | Medium | High | Reversibility is typed + enforced by collision/bijection checker, property tests, open-corpus round-trip; mandatory 2nd reviewer for `strict`; conservative-class-wins on dispute | Reviewers / Maintainer |
| Mapping error (wrong grapheme) | Medium | Medium | Golden vectors from open authoritative examples; per-script qualified review; post-release defect log; conformance tests | Reviewers |
| EU sui generis database right over a substantial table | Low–Med | Medium | Use facts only, re-derive arrangement, prefer CC0 for raw tables; legal note in provenance docs | License reviewer |
| No qualified reviewer for a script | Medium | High | Don't ship unreviewed; partner with a language/translation org; only schedule pairs with reviewer coverage | Steward / Maintainer |
| No adopter secured → nothing reaches "shipped" | High | High | M0–M2 build adopter-independent value; concrete outreach plan (Open questions) with owner + timeline; `verifiedNeed=false` until secured; pause/decision point if none by end of M2 | Acting maintainer → Steward |
| Unicode normalization / grapheme-segmentation edge cases break round-trips | Medium | Medium | NFC contract baked into engine; explicit normalization/combining-mark tests; vetted segmentation library | Maintainer |
| Misuse for surveillance / name-deanonymization | Low | High | Refusal guardrail; no person-matching features; no real-name datasets; "not certified" disclaimer | Maintainer |
| Politically contested romanization drags project into dispute | Low–Med | Medium | Neutral, standards-conformant schemes; document alternatives; neutral examples; no political adjudication | Maintainer |
| Scope creep into transcription/MT/ML | Medium | Medium | Explicit non-goals; deterministic-core rule; reject such tasks | Maintainer |

## Security & privacy

- **Threat surface** is small: open-source ingestion of public reference material + text/data
  artifacts in a public repo. Main risks are **integrity** (wrong/tampered source, mapping error,
  false reversibility claim) and **license/compliance**, not data exfiltration.
- **No secrets** in normal flow (donated lane uses the human's own agent; no API keys, tokens, or
  escrow). Per CLAUDE.md, never write secrets/tokens into logs, receipts, or committed files.
- **PII**: none ingested; test corpora are PD/open and non-personal; **no real-name datasets**;
  output carries a "not a certified/official transliteration" disclaimer; **no person-matching/
  deanonymization features** (refusal guardrail).
- **Abuse/misuse prevention**: refuse tasks that would build surveillance/identity-matching tooling,
  inject misinformation, take a political side on contested romanization, primarily benefit a
  for-profit, or violate a source license/privacy.
- **Supply-chain**: pin source URLs + version/date; **hash/archive license + source snapshots**; run a
  source-change watcher (hash-diff) to detect later drift (M0 minimal, M1 automated). Pin tooling
  dependencies; the engine has no network calls at runtime.

## Sustainability & maintenance

- **After release**, the maintainer + steward keep the provenance allow-list and schemes current,
  re-verify sources on drift, and run the harness in CI on every change so reversibility cannot
  silently regress. **Semantic versioning + a deprecation policy** for the data give adopters a stable
  contract.
- **Outcome tracking** continues post-release: a per-scheme **defect/feedback log** (mapping errors,
  reversibility regressions) and periodic adopter check-ins to confirm continued use. Outcomes
  (adoption, defects, fidelity) — not merge counts — are the maintained metrics.
- **Decommissioning**: if a source's license changes to forbid reuse, affected schemes are flagged and,
  if required, rebuilt from alternative open sources or withdrawn; provenance makes impact assessment
  possible. Reviewer rotation (M1/M4) avoids single-person dependence.

## Open questions

1. **Adopter (blocks M3 and `verifiedNeed=true`).** Who is the first downstream project that will
   integrate and depend on these datasets? **Outreach plan (concrete):**
   - **Target types (named):** Wikimedia language tooling / Wikipedia transliteration gadgets,
     library/GLAM cataloguing systems and ALA/IFLA working groups, Unicode CLDR (as upstream/peer),
     Translators without Borders / CLEAR Global, academic NLP groups, language-revitalization NGOs.
   - **Owner:** acting maintainer until a Steward is named (end of M1), who then takes it over.
   - **Timeline:** begin outreach in parallel with M0; aim for **≥ 3 serious conversations by end of
     M2** and a **first adopter during M3 sourcing**.
   - **Pause/decision point:** if **no adopter by end of M2** (M0–M2 foundations complete), the
     maintainer makes an explicit **go / pivot / hold** decision (e.g., publish as a pure open data
     release and pause new scheme work) rather than building schemes no one has committed to use.
2. **Data license: CC-BY-4.0 vs CC0** for the raw factual mapping tables? CC0 maximizes reuse and
   sidesteps attribution-stacking and DB-right concerns; CC-BY keeps attribution to open sources
   legible. Maintainer decision needed (default in this draft: **CC-BY-4.0**, CC0 under consideration).
3. **Reviewer sourcing:** recruit individual per-script reviewers or partner with a language/
   translation org? Formal qualification criteria? (Drafted in M1.)
4. **Conformance citation vs. independence:** how precisely do we claim "ISO 9 System A–conformant"
   when re-deriving from open sources, to be accurate without implying endorsement or copying? (Legal/
   wording review.)
5. **Scheme selection per script:** which exact scheme(s) per pair (e.g., ISO 15919 vs IAST vs
   Hunterian for Devanagari; ISO 233 vs Buckwalter-style for Arabic) best balance reversibility,
   open provenance, and real-world demand?
6. **Funded lane?** Out of scope for v0.1 (donated only); revisit only if a large, well-scoped batch
   build is justified — would require `fundedBudgetUsd` and a hard cap.

## References

- `C:\code\elyos\CLAUDE.md` — Elyos work rules, lanes, quality bar, refusal guardrails.
- `C:\code\elyos\docs\good-deed-definition.md` — good-deed criteria and risk tiers.
- `C:\code\elyos\packages\schema\src\schemas.ts` — Task JSON schema.
- `C:\code\elyos\planning\ROADMAP.md` — portfolio roadmap (open-transliteration listed, Track 4).
- UNGEGN / UN romanization systems for geographical names (verify per-document reuse terms).
- BGN/PCGN romanization systems (US-gov / Crown; verify per document).
- Unicode CLDR / ICU transform data (Unicode License) — open prior art and comparison baseline.
- ISO 9 / ISO 233 / ISO 15919, DIN 31635, ALA-LC tables — **cited for conformance only; not copied.**
- IAST (scholarly Sanskrit convention), Buckwalter transliteration (published ASCII Arabic scheme).
- Open/PD corpora for test vectors: UDHR translations, Wikisource PD texts.

---

## Appendix A — Improvements applied

Twenty-five specific refinements applied to the first draft of this plan (and the companion TASKS.md).
Each is concrete and already reflected above.

1. **Typed reversibility (`strict | normalized | lossy`).** Replaced the vague word "reversible" with
   a three-tier, machine-checkable class enforced by the test harness — so the headline claim is
   honest per scheme.
2. **NFC normalization contract baked into the engine** and into every scheme definition, with
   `reverse(forward(s)) == NFC(s)` as the explicit `strict` test target (not byte-identity).
3. **Extended-grapheme-cluster segmentation** specified as the tokenization unit (not code points),
   so combining marks, conjuncts, and ligatures round-trip correctly.
4. **Collision/bijection static checker** added as a first-class gate that *fails the build* when a
   `strict` claim is not supported by the rule set.
5. **Re-derive, never copy** doctrine made explicit, with ISO/DIN/ALA-LC named as **excluded sources**
   for verbatim copying and "cite conformance, don't paste the table" as the rule.
6. **EU sui generis database-right** risk surfaced explicitly (not just US copyright), with CC0-for-
   raw-tables as a mitigation under consideration.
7. **Cold-start sequenced on the *provably-reversible* case (Cyrillic ↔ Latin, ISO 9 System A)** so the
   round-trip harness is trustworthy *before* the hard scripts — a deliberate ordering decision.
8. **Arabic reversibility handled honestly:** `strict/normalized` only on **vocalized** input, with an
   explicit **`lossy` + loss report** for unvocalized input — instead of an unqualified "reversible."
9. **Loss report artifact** mandated for every `normalized`/`lossy` scheme, enumerating exactly what
   is dropped.
10. **Property-based testing (`fast-check`) over the script's Unicode ranges** added alongside golden
    vectors and open-corpus round-trips — three independent proof methods, not one.
11. **Open/PD test corpora named** (UDHR, Wikisource PD, UNGEGN sample lists) and **real-name datasets
    explicitly banned** — closing a privacy hole specific to name transliteration.
12. **"Not a certified/official transliteration" disclaimer** added to every deliverable to prevent
    misuse as legal/identity documentation.
13. **Surveillance / name-deanonymization refusal** written in as a hard guardrail (no person-matching
    features) — addressing the dual-use risk of cross-script identity matching.
14. **Political-neutrality stance** added for contested scripts/place names, with neutral examples and
    no adjudication of romanization disputes.
15. **Determinism / no-ML constraint** elevated to a locked decision — reversibility cannot rest on a
    model's guess; this also keeps the core agent-neutral per CLAUDE.md.
16. **Provenance-as-data with SHA-256 snapshot hash + web-archive URL + source-change watcher**
    (minimal M0, automated M1) — mirroring the proven license-discipline pattern from
    `vital-info-translations`.
17. **BLOCKING license/provenance prerequisite per scheme** promoted from an open question to a hard
    pre-authoring gate (build-from-open-sources + output-license-compatible confirmed first).
18. **Second-reviewer rule scoped to the real risk:** mandatory for any `strict` claim and the first
    scheme in each new script family (a false bijection is the silent, high-impact failure).
19. **Conservative-class-wins** dispute rule (downgrade `strict`→`normalized` rather than over-claim)
    added to conflict resolution.
20. **Agent uncertainty self-check operationalized** with a defined `UNCERTAIN:` flag format and a
    rule that unresolved flags block sign-off — adapted to transliteration flag types
    (mapping/context/reversibility/normalization/provenance/neutrality).
21. **Two distinct success states — "released" vs "shipped (adopted)"** — so we never mistake a public
    data release for the actual outcome (downstream adoption).
22. **Interim foundation metrics** (adopter-independent) separated from outcome metrics so progress is
    visible before a partner exists, and "100%/0-defect" is honestly scoped to `strict` schemes from M0.
23. **ISO 15924 script codes** (`Cyrl`, `Latn`, `Deva`, `Arab`) adopted in the schema for unambiguous
    script identification and interoperability.
24. **Semantic versioning + deprecation policy for the data** added under sustainability, giving
    adopters a stable contract (data, not just code, is an API here).
25. **Explicit hard-case test inventory** encoded as required tests (Cyrillic case; Devanagari
    virama/conjuncts/anusvara/nukta/schwa; Arabic sun/moon letters, hamza/madda, ta marbuta;
    Arabic-Indic & Devanagari digits; punctuation/foreign-text passthrough; homoglyph output) — so
    edge cases are tested by contract, not discovered in production.

## Review sign-off

**Reviewer:** acting maintainer (self-review pass against PLAN_SPEC, CLAUDE.md, and the good-deed
definition). **Date:** 2026-06-28. **Verdict:** Approved as Draft v0.1.0.

**Completeness check (PLAN_SPEC 17 sections):** all 17 mandated H2 sections present and in order —
Executive summary; Problem & beneficiaries; Goals and non-goals; Success metrics; Scope; Solution
approach & architecture; Data, licensing & compliance; Quality, review & risk gates; Roadmap &
milestones; Work breakdown; Governance, roles & stakeholders; Dependencies & integrations; Risks &
mitigations (table present); Security & privacy; Sustainability & maintenance; Open questions;
References. Plus Appendix A (25 improvements) and this sign-off.

**Correctness fixes applied during review:**
- Verified every `riskTier` use is **medium** for schemes and **low** for pure tooling/process — and
  confirmed the project is *not* high risk (no medical/legal/safety advice), so no credentialed-expert
  gate is required, only qualified per-script reviewers. Stated explicitly in Quality gates.
- Confirmed **all licensing claims are conservative**: ISO/DIN/ALA-LC marked *cite-only, never
  copied*; data default **CC-BY-4.0** with CC0 flagged as an open decision; code **MIT**; every source
  class license named and "verify per document" attached where terms vary.
- Confirmed **`verifiedNeed=false`** is the honest default everywhere value depends on the unsecured
  adopter, and **partner/adopter marked TO BE SECURED** in Problem, Governance, Dependencies, and
  Open questions.
- Confirmed the **reversibility headline metric ("100%")** is scoped to `strict` schemes and defined
  precisely (Success metrics) so it is not over-claimed for `normalized`/`lossy` schemes.
- Confirmed alignment with CLAUDE.md: agent-neutral core (no ML/vendor logic in `packages/core`),
  donated lane only (no API-key execution), no secrets in artifacts, refusal guardrails encoded
  (surveillance, privacy, neutrality, for-profit, license).

**Residual risks accepted (tracked, not blocking the Draft):** no adopter yet (M0–M2 deliver
adopter-independent value; pause/decision point at end of M2); reviewer sourcing per script (M1);
CC-BY vs CC0 data-license decision (Open question 2). These are explicit and owned, not hidden.
