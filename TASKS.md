# TASKS — open-transliteration

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: J. Carter (acting maintainer) · Lane: donated

The backlog for the `open-transliteration` good-deed project. Read alongside [PLAN.md](./PLAN.md).
Milestones (M0–M4) match the roadmap there.

## How these tasks map to Elyos

Each task below becomes an **Elyos Task JSON** validated against `packages/schema/src/schemas.ts`.
Field mapping:

- **id** — stable slug id, e.g. `open-transliteration-schema-001` (table column `ID`).
- **title** — the task title.
- **project** — always `open-transliteration`.
- **type** — one of `code | research | writing | data | design-spec | maintenance` (table `Type`).
- **lane** — `donated` for all current tasks (no funded/API execution). Funded tasks would need
  `fundedBudgetUsd`; none here.
- **priority** — `high | medium | low`.
- **domain** — array, e.g. `["transliteration","linguistics","i18n"]`.
- **riskTier** — `low | medium | high`. **Scheme/mapping/review tasks are `medium`** (need a qualified
  per-script reviewer + domain accuracy); pure tooling/schema/process tasks are `low` (table `Risk`).
  No task is `high` — this project gives no medical/legal/safety advice.
- **urgent** — boolean (default `false`; no live emergency).
- **deliverable** — `pr | dataset | document | translation` (table `Deliverable`). Mapping data and
  test vectors are `dataset`; engine/schema/tooling are `pr`; docs/criteria are `document`. (Note:
  `translation` is the schema's term for transliteration output where a discrete artifact is produced.)
- **tokenEstimate** — `small | medium | large` (table `Size`).
- **status** — `open | in-progress | review | delivered | done` (all start `open`).
- **context / objective** — why + what.
- **acceptanceCriteria[]** — checkable bullets (listed below tables for key tasks).
- **resources[]** — links/paths (PLAN.md, provenance allow-list, open source refs, corpora).
- **output** — path/description of the produced artifact.
- **requestor** — adopter/requestor; `TO BE SECURED` until an adopter is confirmed.
- **verifiedNeed** — boolean; **`false`** wherever value depends on an unsecured adopter.
- **outputLicense** — **MIT** for code/tooling/schema; **CC-BY-4.0** for mapping data, vectors, and
  docs (CC0 under consideration for raw mapping tables — see PLAN Open question 2).

---

## Milestone M0 — Foundation & cold-start (strict bijection; no partner required)

Goal: stand up schema, engine, provenance model, and the round-trip test harness, and prove them on
one **strictly-reversible** Cyrillic ↔ Latin scheme end-to-end except adoption.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| open-transliteration-provenance-001 | Build open/PD source provenance allow-list (license + snapshot hash/archive) | data | small | low | dataset | — | Maintainer / License reviewer |
| open-transliteration-schema-001 | JSON Schemas: mappingScheme, mappingRule, provenance, review (+ CI validation) | code | small | low | pr | — | Maintainer |
| open-transliteration-license-000 | **BLOCKING:** confirm open/PD-only buildability + output-license compatibility for first scheme | research | small | low | document | provenance-001 | License reviewer / Maintainer |
| open-transliteration-engine-001 | Deterministic engine: NFC + grapheme-cluster segmentation + ordered rules (forward+inverse) | code | medium | low | pr | schema-001 | Maintainer |
| open-transliteration-checker-001 | Collision/bijection static checker (fails unsupported `strict` claims) | code | small | low | pr | schema-001, engine-001 | Maintainer |
| open-transliteration-harness-001 | Round-trip test harness: property tests + golden vectors + open-corpus | code | medium | low | pr | engine-001, checker-001 | Maintainer |
| open-transliteration-scheme-cyrl-001 | Cyrillic ↔ Latin scheme (ISO 9 System A–conformant, `strict`) | data | medium | medium | dataset | engine-001, harness-001, **license-000** | Qualified Cyrillic reviewer (+ 2nd reviewer, `strict`) |
| open-transliteration-vectors-cyrl-001 | Golden test vectors for Cyrillic ↔ Latin from open examples | data | small | medium | dataset | scheme-cyrl-001 | Qualified Cyrillic reviewer |
| open-transliteration-watcher-001 | Minimal source-change re-fetch check (hash-diff) [automated in M1] | code | small | low | pr | provenance-001, schema-001 | Maintainer |

**Acceptance criteria — key M0 tasks**

`provenance-001` (provenance allow-list)
- `data/provenance/allow-list.yaml` lists ≥ 4 sources with **verified open/PD/CC/OFL terms**
  (e.g., a UNGEGN system, a US-gov/BGN-PCGN work, Unicode CLDR/ICU data, an open corpus).
- Each entry records: id, name, canonical URL, version/date, retrieval date, license name + URL,
  **snapshot of license text**, derivatives-allowed flag, required attribution string, notes.
- Each entry stores **`snapshotHash` (SHA-256)** and, where available, **`snapshotArchiveUrl`**.
- **ISO/DIN/ALA-LC published tables are explicitly excluded** as copy sources (recorded as
  cite-only references, not allow-listed for copying).
- Each entry has `verifiedBy` + `verifiedDate`; unclear/incompatible sources are marked `excluded`
  with a reason. Validates against `provenanceSchema` in CI.

`license-000` (BLOCKING prerequisite of `scheme-cyrl-001`)
- For the **specific Cyrillic ↔ Latin scheme** chosen, confirm and record **in writing on the
  provenance entries**: (a) the mapping can be **built entirely from open/PD sources or re-derived**
  without copying any copyrighted table; (b) the **output license** (CC-BY-4.0 data / MIT code) is
  compatible with every source used.
- If either cannot be confirmed, the scheme is **deferred** and an alternative open-buildable scheme
  is selected. `scheme-cyrl-001` **must not start** until this passes.

`engine-001` (reference engine)
- Pipeline: **NFC-normalize → segment into extended grapheme clusters → apply ordered
  context-sensitive rules → emit**, with a symmetric inverse path; **deterministic, no ML, no network
  at runtime**.
- Lives in `packages/core` with no vendor/agent-specific logic (CLAUDE.md); loads schemes validated by
  `mappingSchemeSchema`.
- Unit tests cover normalization, combining marks, context rules, and passthrough of
  digits/punctuation/foreign text.

`scheme-cyrl-001` (first scheme, `strict`)
- `license-000` passed **before authoring**; provenance recorded.
- One Cyrillic ↔ Latin scheme authored as validated data, tagged `reversibility: strict`,
  `normalizationForm: NFC`, `conformsTo` citing ISO 9 System A **without copying its table**.
- **Collision/bijection checker confirms `strict` is achievable** (no ambiguous inverse).
- **100% round-trip** (`reverse(forward(s)) == NFC(s)`) on golden vectors + property-generated cases +
  an open Cyrillic corpus, green in CI.
- Cyrillic **case** and combining-mark edge cases tested; examples are neutral/PD.
- **Qualified Cyrillic reviewer sign-off recorded in the PR** (reviewer independent of drafting; COI
  declared) **and a second independent reviewer** signs off because the scheme claims `strict`.
- Agent `UNCERTAIN:` flags captured into `review.yaml`; **no sign-off while any flag unresolved**.

**M0 Definition of Done:** provenance allow-list (≥ 4 verified open/PD sources, snapshot hash/archive)
+ JSON Schemas + deterministic engine + collision/bijection checker + round-trip harness merged and
green in CI; **first scheme's open-buildability + license compatibility confirmed (`license-000`)
before authoring**; **one `strict` Cyrillic ↔ Latin scheme passing 100% round-trip** with qualified
(+ second) reviewer sign-off and a passing license/provenance check; minimal source-change re-fetch
check green. `verifiedNeed: false` on all M0 deliverables (no adopter; adoption deferred to M3).

---

## Milestone M1 — Repeatability & second script family (Devanagari ↔ Latin; reviewers)

Goal: generalize the engine to richer context rules + conjuncts + the `normalized` class, add a second
scheme, and build the reviewer network.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| open-transliteration-engine-002 | Engine: context-sensitive conjuncts/virama + `normalized` class + loss reports | code | medium | low | pr | engine-001 | Maintainer |
| open-transliteration-reviewers-001 | Per-script reviewer qualification criteria + onboarding | writing | small | low | document | — | Maintainer |
| open-transliteration-reviewers-002 | Recruit/engage ≥ 2 qualified per-script reviewers (or a language org) | research | medium | low | document | reviewers-001 | Maintainer / Steward |
| open-transliteration-scheme-deva-001 | Devanagari ↔ Latin scheme (ISO 15919 / IAST–conformant, `normalized`) | data | large | medium | dataset | engine-002, reviewers-002 | Qualified Devanagari reviewer (+ 2nd, first-in-family) |
| open-transliteration-vectors-deva-001 | Golden vectors + loss report for Devanagari ↔ Latin | data | medium | medium | dataset | scheme-deva-001 | Qualified Devanagari reviewer |
| open-transliteration-license-001 | License/provenance check tooling (lint schemes vs allow-list) in CI | code | small | low | pr | schema-001, provenance-001 | License reviewer / Maintainer |
| open-transliteration-watcher-002 | Automate source-change watcher (scheduled hash-diff) | code | small | low | pr | watcher-001 | Maintainer |

**Acceptance criteria — key M1 tasks**

`reviewers-001`
- Objective criteria for "qualified per-script reviewer" (native/near-native script command +
  linguistic/transliteration competence + COI declaration), plus onboarding/sign-off workflow.
- Defines **reviewer independence** (drafting human ≠ sole reviewer), the **mandatory second reviewer**
  for any `strict` claim and the first scheme in a new script family, **back-mapping QA** for `strict`
  claims, and the **disagreement rule** (reconcile against open sources/checker → escalate → **more
  conservative reversibility class wins**; recorded in `review.yaml`).

`scheme-deva-001`
- Engine handles Devanagari grapheme clusters: virama/conjuncts, anusvara/visarga, nukta, and
  documented **schwa handling**; Devanagari digits (०१२३) tested.
- Tagged `reversibility: normalized` with a **documented normalization contract** and a **loss report**
  enumerating exactly what (if anything) is dropped; round-trips losslessly **up to that normalization**
  on golden vectors + open corpus.
- `conformsTo` cites ISO 15919 / IAST **without copying any copyrighted table**; provenance recorded.
- Qualified Devanagari reviewer + **second reviewer** (first scheme in family) sign-off; `UNCERTAIN:`
  flags resolved.

`license-001`
- Tooling validates each scheme's `provenanceRefs` resolve to allow-listed open/PD sources and that
  `license` is compatible; **fails CI** on any scheme with missing/incompatible provenance or any
  reference to an excluded (copyrighted-table) source as a copy source.

**M1 Definition of Done:** engine supports context-sensitive conjuncts + the `normalized` class with
loss reports; **one Devanagari ↔ Latin scheme released** (reviewed, license-clear, tested);
qualification criteria published and **≥ 2 qualified per-script reviewers engaged**; license/provenance
check **enforced in CI**; **automated source-change watcher operating**. **Steward named** (governance
prerequisite for M3). `verifiedNeed: false` (no adopter yet).

---

## Milestone M2 — The hard case: Arabic ↔ Latin (honest reversibility)

Goal: tackle the hardest pair (unwritten short vowels) and ship it with **honest tagging** —
reversible on vocalized input, explicitly `lossy` (with a loss report) on unvocalized input.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| open-transliteration-scheme-arab-001 | Arabic ↔ Latin scheme: reversible (vocalized) / `lossy` (unvocalized) | data | large | medium | dataset | engine-002, reviewers-002 | Qualified Arabic reviewer (+ 2nd, first-in-family) |
| open-transliteration-vectors-arab-001 | Golden vectors + loss report for Arabic ↔ Latin | data | medium | medium | dataset | scheme-arab-001 | Qualified Arabic reviewer |
| open-transliteration-docs-001 | Per-scheme docs: conformance, reversibility class, loss report, "not certified" disclaimer | writing | small | low | document | scheme-cyrl-001, scheme-deva-001, scheme-arab-001 | Maintainer |

**Acceptance criteria — key M2 tasks**

`scheme-arab-001`
- Handles sun/moon letters, hamza/madda, ta marbuta, shadda, and **Arabic-Indic digits** (٠١٢٣);
  tested explicitly.
- Declares **two modes**: `strict`/`normalized` on **fully-vocalized** input (round-trip proven) and
  **`lossy` with a published loss report** on **unvocalized** input — **never** claimed reversible for
  unvocalized text.
- `conformsTo` cites the chosen scheme (e.g., ISO 233 / a Buckwalter-style ASCII scheme) **without
  copying a copyrighted table**; provenance recorded.
- Collision/bijection checker run for the `strict`/`normalized` mode; **second independent reviewer +
  back-mapping QA** on any `strict` claim.
- Qualified Arabic reviewer sign-off; `UNCERTAIN:` flags resolved; neutral/PD examples only.

`docs-001`
- Per-scheme README states: the standard it conforms to (cited, not copied), reversibility class +
  normalization contract, the loss report (if any), usage, and the **"not a certified/official
  transliteration"** disclaimer.

**M2 Definition of Done:** Arabic ↔ Latin scheme released with honest reversibility tagging and a
published loss report; three script pairs (Cyrillic, Devanagari, Arabic ↔ Latin) released, each
reviewed, tested, and license-clear; per-scheme docs with disclaimer merged. **Adopter outreach has
produced ≥ 3 serious conversations**; if none, the **go/pivot/hold decision** is recorded.
`verifiedNeed: false` until an adopter is secured.

---

## Milestone M3 — First adopter (needs partner)

Goal: get the datasets **integrated and depended upon** by a real downstream project. **All tasks
gated on a secured adopter** (`verifiedNeed` flips to `true` only when confirmed).

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| open-transliteration-adopter-001 | Secure first adopter; agree scheme(s) + integration needs | research | medium | low | document | reviewers-002 | Steward / Maintainer |
| open-transliteration-contract-001 | Stable data contract + semantic-versioning/deprecation policy | design-spec | small | low | document | adopter-001 | Maintainer |
| open-transliteration-integration-001 | Support adopter integration; confirm scheme in use | code | medium | medium | pr | adopter-001, contract-001 | Steward |

**Acceptance criteria — key M3 tasks**

`adopter-001`
- Outreach (acting maintainer → Steward) targets named candidate types (Wikimedia language tooling,
  GLAM/ALA/IFLA, Unicode CLDR, Translators without Borders/CLEAR Global, NLP groups, language NGOs).
- A named project/org confirmed **in writing** as adopter; agreed scheme(s) and integration needs
  recorded. On completion, related tasks update `requestor` and `verifiedNeed: true`.
- **Pause/decision point** from M2 honored: if no adopter materialized, the recorded go/pivot/hold
  decision governs whether M3 proceeds.

`integration-001`
- The adopter integrates ≥ 1 scheme and **confirms it is in use** (recorded in the PR/receipt) — the
  first true **Definition of Shipped** event.
- Adopter feedback captured into the outcome log; any mapping defect filed against the defect log.

**M3 Definition of Done:** adopter secured (`verifiedNeed=true`); stable data contract +
versioning/deprecation policy published; ≥ 1 scheme **integrated and confirmed in use** by the adopter.

---

## Milestone M4 — Scale & sustain

Goal: broaden coverage with sustained quality and clear maintenance.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| open-transliteration-scale-001 | Add further script pairs / additional schemes per pair | data | large | medium | dataset | integration-001 | Qualified per-script reviewers |
| open-transliteration-lossy-001 | Add clearly-tagged `lossy` pronunciation schemes (never mislabeled) | data | medium | medium | dataset | engine-002 | Qualified per-script reviewers |
| open-transliteration-rotation-001 | Reviewer rotation + load balancing | maintenance | small | low | document | reviewers-002 | Maintainer |
| open-transliteration-outcomes-001 | Outcome tracking: post-release defect + adopter-feedback log | data | small | low | dataset | integration-001 | Steward |
| open-transliteration-maint-001 | Source re-verification + scheme maintenance cadence | maintenance | small | low | document | provenance-001 | Maintainer |

**Acceptance criteria — key M4 tasks**

`lossy-001`
- Any pronunciation-oriented scheme is tagged `reversibility: lossy` with a loss report; the harness
  does **not** assert round-trip equality for it; docs make the non-reversibility unmistakable.

`outcomes-001`
- A maintained log capturing, per released/adopted scheme: adoption status, post-release critical
  defects (target 0), round-trip regressions, and adopter usefulness feedback; feeds PLAN.md metrics.

**M4 Definition of Done:** ≥ 3 script pairs released and ≥ 1 adopted; any `lossy` schemes correctly
tagged; reviewer rotation operating; outcome tracking live; source/scheme maintenance cadence in
effect; named sustainability owner.

---

## Backlog / future

Sized but unscheduled:

- **(large) Additional script families** — Greek, Hebrew, Georgian, Armenian, Thai, Bengali, Tamil ↔
  Latin (each its own provenance + reviewer + reversibility analysis).
- **(medium) Cross-script (non-Latin↔non-Latin) transforms** — e.g., Cyrillic ↔ Greek — composing
  via a pivot, with reversibility analysis across the composition.
- **(medium) WASM build of the engine** for browser/edge reuse by adopters (no behavior change).
- **(small) Conformance-comparison report** vs. ICU/CLDR transforms (open prior art) to document
  agreements/differences — references only, no copying.
- **(small) Reverse-only "search normalization" profiles** (fold to ASCII for indexing) — clearly
  tagged `lossy`, separate from reversible schemes.
- **(large, funded — needs escrow) Batch scheme build** under the funded lane for a well-scoped set,
  with a hard `fundedBudgetUsd` cap — out of scope for v0.1.

---

## Example task JSON

Schema-valid Task JSON for the first M0 task. `verifiedNeed` is **false** (no adopter secured);
`outputLicense` is **CC-BY-4.0** because the provenance allow-list is project data/documentation.

```json
{
  "id": "open-transliteration-provenance-001",
  "title": "Build open/PD source provenance allow-list with per-source license terms",
  "project": "open-transliteration",
  "type": "data",
  "lane": "donated",
  "priority": "high",
  "domain": ["transliteration", "linguistics", "i18n", "licensing"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "dataset",
  "tokenEstimate": "small",
  "status": "open",
  "context": "open-transliteration ships open, reversible, tested transliteration data between scripts. The authoritative scheme descriptions (ISO 9/233/15919, DIN 31635, ALA-LC) are copyrighted documents whose tables must NOT be copied; mapping facts are built instead from openly-licensed/public-domain sources (UNGEGN/UN romanization systems, BGN/PCGN and other US-gov works, Unicode CLDR/ICU transform data, open corpora). Nothing may be built into a scheme until its source is allow-listed with verified open/PD/CC/OFL terms.",
  "objective": "Create a structured, per-source provenance allow-list recording each approved open/PD source's license terms, derivatives permission, required attribution, and a hash/archive snapshot, so each scheme's provenance and the license gate can be checked consistently — and so copyrighted standards are recorded as cite-only, never as copy sources.",
  "acceptanceCriteria": [
    "data/provenance/allow-list.yaml lists >= 4 sources with verified open/PD/CC/OFL terms (e.g., a UNGEGN system, a US-gov/BGN-PCGN work, Unicode CLDR/ICU data, an open corpus)",
    "Each entry records: id, name, canonical URL, version/date, retrieval date, license name + URL, snapshot of license text, derivatives-allowed flag, required attribution string, notes",
    "Each entry stores snapshotHash (SHA-256) and, where available, snapshotArchiveUrl (web-archive copy)",
    "ISO/DIN/ALA-LC published tables are explicitly recorded as cite-only references and excluded as copy sources",
    "Each entry has verifiedBy and verifiedDate; any source with unclear or incompatible terms is marked excluded with a reason",
    "File validates against provenanceSchema and passes CI structural checks"
  ],
  "resources": [
    "C:/Users/jason/AppData/Local/Temp/claude/C--code-elyos/5eca0d44-6b8b-4c30-9696-37a524cb249a/scratchpad/plans/open-transliteration/PLAN.md",
    "UNGEGN / UN romanization systems for geographical names",
    "BGN/PCGN romanization systems (US-gov / Crown)",
    "Unicode CLDR / ICU transform data (Unicode License)",
    "Open/PD corpora: UDHR translations, Wikisource PD texts"
  ],
  "output": "data/provenance/allow-list.yaml plus a short README documenting the allow-list schema and verification process",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```
