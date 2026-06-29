# Competitive & Improvement Analysis — `open-transliteration`

> Analyst pass against PLAN.md (Draft v0.1.0, 2026-06-28) and TASKS.md. Web-researched competitor
> claims are cited inline. Scope: open, standards-based, reversible, expert-reviewed transliteration
> between scripts for under-served languages.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on legal/provenance discipline and on the *honesty* of its reversibility
claim. It is weaker on **standards breadth**, **conformance verification**, and **disambiguation
theory**. Findings:

**Standards coverage — partial and slightly inconsistent.**
- The plan names ISO 9, ISO 233, ISO 15919, DIN 31635, ALA-LC, BGN/PCGN, UNGEGN, IAST, Buckwalter.
  That is a good roster, but **UNGEGN is treated as a single citable thing** when it is in fact a
  *family* of per-language romanization systems of *varying* reversibility and *varying* reuse terms
  (the plan acknowledges "verify per-document" but never grapples with the fact that many UNGEGN
  systems are themselves **non-reversible/phonetic**, e.g. the UN system for Arabic). The cold-start
  premise ("conform to a genuinely reversible standard") is sound for Cyrillic but UNGEGN will not
  always supply one.
- **ISO 9 is the natural strict bijection and the plan uses it — good.** But note ISO 9:1995 *is*
  the single-table bijective system; the plan's phrase "ISO 9 **System A**" conflates ISO 9 with the
  ALA-LC/BGN "System A/B" convention used elsewhere. **This is a substantive terminology bug**: ISO 9
  does not have an "System A"; the bijective 1995 table is just ISO 9. The repeated "ISO 9 System A"
  label (executive summary, data model, M0, success metrics) should be corrected to "ISO 9:1995" or
  the plan should define precisely what "System A" means here. (See Finding A below — flagged as a
  top correctness issue.)
- **DIN 31635** is listed only as an excluded copy-source; it is never positioned as a *conformance
  target* for Arabic even though it is the dominant scholarly Arabic romanization. M2 names "ISO 233 /
  Buckwalter-style" but omits DIN 31635, which is the scheme most Arabic reviewers will expect.
- **No mention of ISO 15924 reversibility nuance** beyond using the codes for identification (good
  that codes are used). Missing: a **scheme-variant identifier** dimension. A single script pair has
  *many* coexisting standards (Devanagari→Latin alone: ISO 15919, IAST, Hunterian, ITRANS,
  Harvard-Kyoto, SLP1, Velthuis). The schema's `id` + `conformsTo` capture one, but there is no
  declared model for **multiple schemes per pair** or for **scheme equivalence/lossy-bridge mapping**
  between them — which is exactly what real adopters need (see Gaps §3).

**Reversibility / round-trip — the strongest part, mostly correct.**
- The three-tier `strict | normalized | lossy` typing, the `reverse(forward(s)) == NFC(s)` contract,
  the static collision/bijection checker, property-based + golden + open-corpus testing, and the
  "conservative-class-wins" dispute rule are all excellent and ahead of every competitor (none of
  which ship an automated round-trip *proof*).
- **Gap:** round-trip is asserted only as `reverse(forward(s))` (source→target→source). A truly
  bijective claim also needs the **other direction proven**: `forward(reverse(t)) == t` for all valid
  *target* (Latin-with-diacritics) strings. For a Latin-side consumer who *starts* from romanized text
  (the common GLAM/library case — records are stored romanized), the untested direction is the one
  they rely on. The plan should assert **both compositions** for `strict` schemes.
- **Gap:** "valid grapheme sequences over the script's code-point inventory" — property generation
  over raw code-point ranges will generate **many sequences that are not well-formed text** (lone
  combining marks, illegal Indic cluster orders). Without a generator constrained to *well-formed*
  clusters, either tests will be noisy or the "100%" headline quietly excludes ill-formed input. The
  normalization contract needs to state what happens to **ill-formed input** (reject? pass through?
  normalize?), which is currently unspecified.

**Per-script / native-expert review — well-designed.** Mandatory qualified per-script reviewer,
mandatory *second* reviewer for any `strict` claim and first-in-family, COI declarations, agent
`UNCERTAIN:` flags blocking sign-off. This is the right governance and exceeds academic norms. One
gap: **no reviewer is required for `lossy` loss-report accuracy specifically** — the loss report is
the user-facing safety claim for Arabic, and it should get the same second-reviewer rigor as a
`strict` claim, since an *understated* loss report is as harmful as a false bijection.

**Ambiguity / context-dependence — under-theorized.** The schema has `contextBefore`/`contextAfter`
regex-ish conditions and "first match wins," which is the ICU transform model. But the plan does not
address: (a) **longest-match vs. first-match** ordering hazards (a classic source of transliteration
bugs); (b) **word-boundary and token-level context** (Arabic sun/moon assimilation, Devanagari schwa
deletion are *positional within a word*, not just adjacent-grapheme); (c) **many-to-one source
ambiguity that is legitimate** (e.g. Cyrillic vs. Latin homoglyphs, or two source graphemes the
standard maps to one target — which *forces* a non-strict class). The collision checker catches
target-side collisions but the plan should also state how it handles **rule-overlap on the source
side** (two rules whose contexts both match).

**Not conflating with translation / phonetic respelling — explicitly and correctly handled.** The
non-goals are crisp: graphemic transliteration only, pronunciation schemes tagged lossy, never MT,
"not certified/official" disclaimer. This is a genuine differentiator and is consistently enforced.
One refinement: the plan should also explicitly disclaim **"romanization == searchable ASCII"** —
adopters frequently *want* lossy ASCII folding (like Unidecode) and will misuse a `strict` diacritic
scheme for it; a short "if you want ASCII search keys, here is the lossy companion" pointer prevents
misuse.

**Unicode normalization — good, with one hole.** NFC contract + extended grapheme cluster
segmentation via `Intl.Segmenter` is correct. Holes: (a) **NFC is not always the right canonical
target for Indic** — some Indic round-trips are cleaner reasoning over NFD or a custom canonical
cluster order; the plan hard-codes NFC as *the* contract, which may fight Devanagari conjunct
ordering. (b) **`Intl.Segmenter` grapheme boundaries follow UAX #29, which is not always the
linguistically correct akshara boundary** for complex Indic clusters — worth a flagged risk. (c) No
mention of **Unicode version pinning** — grapheme/normalization behavior drifts across Unicode
versions and Node releases; the engine should pin and record the Unicode/ICU version per test run.

**Scope & testing — coherent and appropriately narrow.** Donated-lane-only, no ML in core, no hosted
service, no fonts/OCR — all consistent with CLAUDE.md. The "released vs shipped(adopted)" distinction
is honest. The biggest *project-risk* (correctly self-identified) is **no secured adopter**; the
go/pivot/hold gate at end of M2 is the right mitigation.

**Top-two plan-correctness findings (most important):**
- **Finding A — "ISO 9 System A" is a terminology error.** ISO 9:1995 is the bijective system; there
  is no "System A" within ISO 9. The conflated label appears in the executive summary, the data-model
  example (`cyrl-latn-iso9a`), success metrics, and M0/roadmap. This is exactly the kind of
  standards-imprecision the project exists to eliminate, so it must be fixed at the source.
- **Finding B — round-trip is proven in only one direction.** `strict` bijection requires proving
  *both* `reverse(forward(s))==NFC(s)` **and** `forward(reverse(t))==t`; the latter (Latin→native) is
  the direction library/GLAM adopters who store romanized records actually depend on, and it is
  currently untested. (Plus the unconstrained property generator may silently exclude ill-formed
  input from the "100%" headline.)

---

## 2. Competitive landscape (researched, cited)

**ICU / CLDR Transliterator** — the 800-lb incumbent. Rule-based `Transliterator` with a mature
transform-rule language; CLDR ships generic script-to-script and language-specific transforms
(Cyrillic-Latin, Russian-French, etc.). *Strengths:* battle-tested, ubiquitous (every browser/OS),
permissive license (ICU/Unicode-3.0), explicitly designed so many Latin transforms are reversible
"using extra accents to provide for a round-trip." *Weaknesses:* reversibility is **broad and
inconsistent** ("interpreted broadly to mean both reversible and non-reversible transforms"; only
"some" augmented for round-trip) and **not proven per-scheme by an automated bijection checker**;
huge C++/Java dependency; transform data is **not conformance-audited against ISO/UNGEGN per scheme**;
hard to cite "this equals ISO 9." Sources:
https://unicode-org.github.io/icu/userguide/transforms/general/ ,
https://cldr.unicode.org/index/cldr-spec/transliteration-guidelines ,
https://github.com/unicode-org/icu/blob/main/LICENSE

**Aksharamukha** (Vinodh Rajan) — the strongest open Indic competitor. *Strengths:* 121 scripts, 20+
romanization methods (IAST, ISO, Harvard-Kyoto, ITRANS, Velthuis, SLP1, WX, Titus…), **explicitly
lossless/reversible between main Indic scripts**, handles orthographic conventions (vowel length,
gemination, nasalization), GPL open source, Python package + web + JS port. *Weaknesses:* Indic/SE-Asian
focused (not Cyrillic/Arabic-centric); reversibility is a design claim, **not an automated round-trip
test suite shipped as proof**; provenance/conformance not formally audited; GPL license constrains
some reuse. Sources: https://github.com/virtualvinodh/aksharamukha ,
https://digitalorientalist.com/2024/11/26/aksharamukha-the-automated-transliteration-tool-to-simplify-script-conversion/ ,
https://aksharamukha.appspot.com/

**uroman (ISI/USC, Hermjakob)** — universal *romanizer*. *Strengths:* any-script→Latin, m-to-n
context-aware mappings, converts numerals, ACL-published, free, used in UNESCO/Meta MMS pipelines.
*Weaknesses:* **one-directional (romanize only) and explicitly lossy** — "does not vowelize text that
lacks explicit vowelization" (Arabic/Hebrew), kanji mis-romanized as Chinese; **not reversible**, not
standards-conformant, designed for string-similarity/NLP not faithful round-trip. Sources:
https://github.com/isi-nlp/uroman , https://aclanthology.org/P18-4003/

**Unidecode (and ports)** — the ubiquitous "good-enough ASCII" tool. *Strengths:* trivially simple,
everywhere, great for slugs/search keys. *Weaknesses:* **explicitly, intentionally lossy and
non-reversible**, context-free char-by-char, no language awareness ("a","o","u" not German ae/oe/ue),
"the further from Latin… the worse." It is the **anti-pattern this project warns about** (lossy used
as if lossless). Source: https://github.com/avian2/unidecode

**Google Dakshina + Aksharantar/IndicXlit + NLLB (neural/data)** — the ML lane. *Strengths:* Dakshina =
12 South-Asian languages, native Wikipedia text + human-attested romanization lexicons + parallel
sentences (open). Aksharantar = 26M pairs across 21 Indic languages (21× prior datasets; first open
data for 7 languages); IndicXlit transformer model; NLLB-200 covers 200 languages. Huge coverage of
*under-served* languages. *Weaknesses:* **statistical/non-deterministic — cannot guarantee
reversibility or standards-conformance**; models hallucinate; outputs are pronunciation-oriented, not
bijective; data is romanization *attestation* (how people actually spell), not a standard. These are
**complements/corpora, not reversible engines.** Sources:
https://github.com/google-research-datasets/dakshina , https://aclanthology.org/2023.findings-emnlp.4/ ,
https://arxiv.org/abs/2205.03018

**libindic / indic_transliteration (PyPI)** — Indic Python libraries (Harvard-Kyoto, ITRANS, SLP1,
WX cross-conversion). *Strengths:* practical, scriptable, widely used in Sanskrit/Indic NLP.
*Weaknesses:* Indic-only, inconsistent reversibility guarantees, varying maintenance, no formal
provenance/conformance audit, no round-trip CI proof. Sources:
https://github.com/libindic/indic-trans , https://pypi.org/project/indic-transliteration/

**`transliteration` / `cyrillic-to-translit-js` (npm) & Wikipedia/Wikidata romanization gadgets** —
the JS ecosystem the project lives in. *Strengths:* lightweight, popular, "vice versa" support in
some. *Weaknesses:* mostly lossy/ASCII-oriented, no standards conformance, no proof, ad-hoc schemes —
again the very gap the project targets. Wikimedia transliteration gadgets are per-wiki, inconsistent,
unaudited (and a candidate *adopter*). Sources: https://www.npmjs.com/package/transliteration ,
https://github.com/greybax/cyrillic-to-translit-js

---

## 3. Gaps we can fill

1. **Proof, not assertion.** No competitor ships an *automated, CI-enforced round-trip/bijection
   proof* per scheme. ICU and Aksharamukha *claim* reversibility; this project *tests* it. This is the
   single cleanest wedge.
2. **Provenance- and license-audited open data.** Nobody ships a per-scheme provenance allow-list
   (source URL + license snapshot + hash + archive + "re-derived not copied" attestation). For
   GLAM/legal/academic adopters who must justify reuse, this is uniquely valuable.
3. **Reversibility *typing* as a first-class, honest contract** (`strict|normalized|lossy` + loss
   report). The whole ecosystem blurs lossy and lossless; the project refuses to.
4. **Standards-conformance citation done carefully** — "conformant to ISO 9:1995, re-derived from open
   sources" — bridging the paywalled-standard / open-data divide that ICU/CLDR don't formally claim.
5. **Under-served scripts beyond the Indic/CJK heavyweights.** The data-ML world (Dakshina,
   Aksharantar) is Indic-South-Asian; uroman is generic-but-lossy. A *reversible, reviewed* commons for
   smaller Cyrillic-using languages, Caucasian, Ethiopic, Tibetan, Mongolian, Thaana, etc. is genuinely
   thin.
6. **Multiple-standards-per-pair + cross-standard bridges.** Real adopters need "convert this IAST
   text to ISO 15919" — a *deterministic, reversible bridge between romanization standards*, which no
   single tool cleanly offers with proof.
7. **Small, dependency-light, TS/ESM-native engine** — ICU is heavy; the JS npm options are unaudited.
   A tiny, auditable, browser-runnable, *proven* engine fills a real niche for web/Wikimedia tools.

---

## 4. Differentiators to win (vs ICU and Aksharamukha)

- **Vs ICU:** ICU's reversibility is "broad" and uneven and *unproven per scheme*; ours is **typed,
  per-scheme, and CI-proven** with a static bijection checker that *fails the build* on a false
  `strict` claim. ICU has no provenance/license audit trail and no conformance assertion you can cite;
  we ship both. ICU is a heavyweight C++/Java dependency; we are a tiny auditable TS/ESM library + data
  that a Wikimedia gadget or a library script can vendor directly.
- **Vs Aksharamukha:** Aksharamukha is broader (121 scripts) and excellent for Indic, but its
  losslessness is a *design intent without a shipped automated proof suite*, it's GPL, and it's
  Indic/SE-Asian-centric. We win on **proof-as-deliverable, permissive MIT/CC-BY(/CC0) licensing,
  provenance auditing, and honest lossy-tagging for the hard scripts (Arabic) Aksharamukha doesn't
  center.** We do *not* try to out-breadth it; we out-*trust* it.
- **Vs the ML lane (Dakshina/Aksharantar/NLLB):** they cover more languages but **cannot guarantee
  anything**; we offer determinism + reversibility + standards-conformance. We can *consume* their
  open corpora as test vectors (attested romanizations) while never depending on a model in the core —
  turning competitors into our test data.
- **The unifying differentiator:** **"reversible-by-proof, open-by-provenance, standards-by-citation"
  — a trustworthy commons, not a bigger one.** Trust and auditability, not coverage, is the moat.

---

## 5. Claude API leverage (and hard limits)

**Where Claude helps (draft/assist, never decide):**
1. **Rule-set drafting & standards-doc summarization.** Claude reads an *openly-licensed* source
   (UNGEGN PDF, BGN/PCGN doc, CLDR transform) and drafts candidate YAML rules + a structured summary
   of the standard's intent — accelerating the human author. Output is a *proposal*, fed to the
   deterministic checker + native reviewer.
2. **Test-case / golden-vector generation & edge-case enumeration.** Claude proposes hard cases
   (Cyrillic case pairs, Devanagari conjunct/virama/nukta/schwa permutations, Arabic sun/moon, hamza,
   ta-marbuta, shadda, Arabic-Indic vs Devanagari digits, homoglyphs, mixed-script passthrough) far
   faster than a human can enumerate — then property tests + reviewers validate them.
3. **Standards/conformance documentation & loss-report drafting.** Claude drafts the per-scheme README,
   the "conforms-to (cited, not copied)" wording, the normalization contract prose, and a *first draft*
   of the loss report — all human/reviewer-verified.
4. (Bonus) **Provenance/license triage assistant** — Claude reads a source's license text and drafts
   the allow-list entry fields and a derivatives-allowed assessment for the human license reviewer to
   confirm.
5. (Bonus) **Cross-standard discrepancy explainer** — when two open sources disagree on a mapping,
   Claude drafts a structured diff + hypotheses for the reviewer to adjudicate.

**Where Claude must NOT decide (hard guardrails, consistent with CLAUDE.md and PLAN):**
- **It must never be the authority on linguistic correctness.** Final mapping correctness is decided by
  **deterministic rules + native/expert review + round-trip tests**, never by model assertion.
- **It must never participate in the reversible core at runtime** (no ML in the deterministic engine —
  a locked decision). Claude is a *drafting tool offline*, not an inference step.
- **No fabricated mappings.** Every Claude-proposed mapping must trace to a recorded open/PD source or
  be flagged `UNCERTAIN: …|provenance`; the bijection checker + reviewer + license gate catch
  hallucinations. A Claude draft with no provenance is **not shippable.**
- **It must not set the reversibility class.** The class is *measured* by the checker/harness, not
  argued by the model; conservative-class-wins.
- **It must not adjudicate contested romanizations or neutrality** — those are human/maintainer calls.
- Donated-lane only: Claude assistance runs on the *human's own agent* interactively; nothing here
  justifies funded-lane API execution (no metered core engine).

---

## 6. Ten concrete optimizations

1. **Fix the "ISO 9 System A" terminology** everywhere (executive summary, `cyrl-latn-iso9a`, metrics,
   M0) → "ISO 9:1995 (single-table bijective)". Add a one-line glossary mapping each scheme id to its
   precise standard designation. (Resolves Finding A.)
2. **Prove both round-trip directions for `strict`:** add `forward(reverse(t)) == t` over the target
   inventory to the harness, not just `reverse(forward(s))`. (Resolves Finding B.)
3. **Constrain the property generator to well-formed grapheme clusters** (legal Indic cluster orders,
   no lone combining marks) and **explicitly specify ill-formed-input behavior** (reject/passthrough)
   in the normalization contract — so "100%" can't hide excluded inputs.
4. **Model multiple schemes per pair + cross-standard bridges** in the schema now (a `schemeFamily` /
   `variantOf` field), so IAST↔ISO 15919 deterministic bridges are first-class — the feature adopters
   actually ask for (§3.6).
5. **Add DIN 31635 and Hunterian explicitly as conformance *targets*** (cite-only) for Arabic and
   Devanagari roadmaps, since reviewers and GLAM users expect them; currently only ISO 233/Buckwalter
   and ISO 15919/IAST are named.
6. **Pin and record Unicode/ICU version** per scheme and per CI run (normalization + segmentation drift
   across Node/Unicode versions can silently break round-trips); add to provenance/sustainability.
7. **Treat the `lossy` loss report as a safety-critical claim:** require the **second-reviewer rule for
   loss-report accuracy** (an understated loss report is as harmful as a false bijection), and add a
   *machine-measured* loss profile (which code-points/diacritics are dropped) auto-generated by the
   harness to back the prose report.
8. **Ship a deliberately-lossy ASCII "search-key" companion** per scheme (folding to ASCII, clearly
   tagged `lossy`, never reversible) so adopters who want Unidecode-style behavior use *our* honest
   labeled one instead of misusing the `strict` diacritic scheme. Turns a misuse risk into a feature.
9. **Cross-validate every mapping against ICU/CLDR and Aksharamukha as an automated *differential
   test*** (not as a copy source) — where we disagree with ICU, force a reviewer note. Free correctness
   signal + a credible "where we differ from ICU and why" doc that markets the project.
10. **Land a named adopter earlier by shipping a thin Wikimedia/GLAM integration spike in M1** (a
    drop-in JS module for a Cyrillic transliteration gadget, or a MARC romanization helper) — the
    adopter gap is the #1 project risk; a tiny working integration de-risks M3 far more than a third
    script does. Pair it with publishing test vectors derived from the open Dakshina lexicons.

---

## 7. Parallel & perpendicular spin-offs

**Reusable transliteration engine (the keystone).** The deterministic `packages/core` engine + schema
is itself a reusable artifact other Elyos tracks can depend on — factor it as a standalone
`@elyos/transliterate` package (engine + scheme loader + checker) so spin-offs share one proven core.

**Parallel (same domain, adjacent deliverable):**
- **`keyboard-layouts-open`** — transliteration rule sets and input-method layouts are duals (romanized
  input → native script is "reverse transliteration"). Share the scheme data + reversibility checker;
  a keyboard's transliterating input mode is literally our `reverse` path.
- **`place-name-pronunciation`** — UNGEGN/BGN romanization data overlaps heavily; our provenance
  allow-list and Cyrillic/Arabic schemes seed it. Clear handoff: we do *graphemic* transliteration;
  they do pronunciation (and must consume our `lossy` tag honestly).
- **`open-pronunciation-audio`** — perpendicular but complementary: they cover the *phonetic* layer we
  explicitly refuse, so a clean boundary ("transliteration ≠ pronunciation") plus a data interchange
  format benefits both and reinforces our non-goal.

**Perpendicular (different domain, shared infrastructure):**
- **`loc-public-domain-engine`** — our provenance/license-snapshot/hash/archive model and the "re-derive
  don't copy" doctrine are directly reusable; consider extracting a shared `@elyos/provenance` package.
- **An MCP server (`transliterate-mcp`)** — expose `transliterate(text, schemeId, direction)`,
  `checkReversible(schemeId)`, `listSchemes()`, `lossProfile(schemeId)` as deterministic MCP tools so
  *any* agent (including Claude) can call the proven engine instead of guessing romanization inline.
  This is the highest-leverage spin-off: it turns the deterministic core into infrastructure the whole
  agent ecosystem can lean on, and naturally markets the project. (Engine stays deterministic; MCP is
  just a transport — consistent with the no-ML-in-core rule.)

---

## 8. Open questions

1. **Adopter (the binding constraint).** Beyond the plan's list, is the fastest concrete win a
   **Wikimedia transliteration gadget** replacement (our JS engine is small enough to vendor) or a
   **library/GLAM MARC romanization** helper? Should M1 carry an integration spike (Optimization #10)?
2. **Schema for multiple standards per pair** — adopt `schemeFamily`/`variantOf` and define
   cross-standard reversible bridges now, or defer? (Affects the data model immediately.)
3. **CC-BY vs CC0 for raw mapping tables** (plan's own open Q2) — CC0 maximizes reuse and sidesteps EU
   *sui generis* DB-right and attribution-stacking; recommend **CC0 for the factual tables, CC-BY for
   prose docs**. Maintainer decision still pending.
4. **Both-direction proof + ill-formed-input contract** (Findings A/B, Optimizations #2/#3) — accept as
   M0 scope additions before the harness is declared "done"?
5. **Differential testing against ICU/Aksharamukha** — is consuming them as *test oracles* (not copy
   sources) license-clean and worth wiring into CI from M0? (Strong recommend yes.)
6. **Unicode/ICU version pinning policy** — which Unicode version is the contract, and how are
   round-trips re-validated when Node/ICU upgrades?
7. **Does the `lossy` ASCII search-key companion** (Optimization #8) belong in scope, given the plan's
   firm "not pronunciation/not ASCII-folding" stance? It is *graphemic folding*, arguably in scope and
   a strong misuse-prevention lever — needs a maintainer ruling.
8. **MCP server placement** — ship `transliterate-mcp` in this repo or as a sibling Elyos track once
   the engine stabilizes (M1/M2)?

---

### Sources
- ICU Transforms: https://unicode-org.github.io/icu/userguide/transforms/general/
- CLDR Transliteration Guidelines: https://cldr.unicode.org/index/cldr-spec/transliteration-guidelines
- ICU license: https://github.com/unicode-org/icu/blob/main/LICENSE
- Aksharamukha (repo): https://github.com/virtualvinodh/aksharamukha
- Aksharamukha (review): https://digitalorientalist.com/2024/11/26/aksharamukha-the-automated-transliteration-tool-to-simplify-script-conversion/
- uroman (repo): https://github.com/isi-nlp/uroman ; paper: https://aclanthology.org/P18-4003/
- Unidecode: https://github.com/avian2/unidecode
- Dakshina dataset: https://github.com/google-research-datasets/dakshina
- Aksharantar / IndicXlit: https://aclanthology.org/2023.findings-emnlp.4/ ; https://arxiv.org/abs/2205.03018
- libindic indic-trans: https://github.com/libindic/indic-trans ; https://pypi.org/project/indic-transliteration/
- npm transliteration / cyrillic-to-translit-js: https://www.npmjs.com/package/transliteration ; https://github.com/greybax/cyrillic-to-translit-js
