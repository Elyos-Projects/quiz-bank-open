# TASKS — quiz-bank-open

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug ID from the tables (e.g. `quiz-bank-open-model-001`).
- `title` — the table's Title.
- `project` — `quiz-bank-open`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table). Schema
  models/specs/checklists → `design-spec`; tooling → `code`; authored banks → `writing`; outreach /
  triage → `research`; refresh/staleness → `maintenance`.
- `lane` — `donated` for all tasks here (no funded escrow). A funded task would add `fundedBudgetUsd`
  with a hard per-task cap.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["education","oer","assessment"]`.
- `riskTier` — `low | medium | high`. License/PII and bias judgement → `medium`; any health/legal/
  safety/finance subject matter → `high` (credentialed expert sign-off required).
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. Code → `pr`; authored question banks and
  specs/checklists → `document` (banks are authored CC-BY content, not harvested data, so we use
  `document` not `dataset`); localized banks → `translation`.
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until an adoption partner is confirmed.
- `verifiedNeed` — **`false`** until a named OER platform/teacher agrees to adopt a bank (general
  need is real; per-task delivery need is unproven).
- `outputLicense` — `CC-BY-4.0` for items/banks/docs (`CC-BY-SA-4.0` where derived from a
  share-alike source); `MIT` for code (schema, validators, exporters, self-test harness).

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| quiz-bank-open-model-001 | Canonical item model + JSON schema + bank/blueprint model | design-spec | medium | low | document | — | Maintainer, Technical |
| quiz-bank-open-grader-002 | Auto-grader scoring spec for core item types (incl. partial credit) | design-spec | medium | low | document | model-001 | Technical |
| quiz-bank-open-gate-003 | OER license + provenance + PII triage checklist (blocking gate) | design-spec | small | medium | document | — | License |
| quiz-bank-open-policy-004 | NC / share-alike / license-compatibility acceptance policy | design-spec | small | medium | document | gate-003 | License |
| quiz-bank-open-a11ybias-005 | Accessibility + bias/sensitivity review checklist | design-spec | small | low | document | — | Accessibility, Bias |
| quiz-bank-open-validator-006 | Item validator + auto-grade self-test harness (CI) | code | medium | low | pr | model-001, grader-002 | Technical |
| quiz-bank-open-reviewers-007 | Name/secure mandatory reviewers (SME per subject, a11y, bias, license) | research | small | low | document | — | Maintainer |
| quiz-bank-open-outreach-008 | OER partner/platform outreach + pilot unit shortlist | research | small | low | document | — | Maintainer |
| quiz-bank-open-pilot-009 | Pilot question bank end-to-end for one OER unit | writing | medium | low | document | model-001, grader-002, gate-003, policy-004, a11ybias-005, validator-006, reviewers-007, outreach-008 | SME, License, Accessibility, Bias |

**Acceptance criteria — key tasks**

- **model-001 (canonical item model + schema)**
  - [ ] JSON schema defines every item field from PLAN: `id, type, stem, assets[]{ref,altText,license,
        attribution,sourceUrl}, choices[]{id,text,correct}, answerKey, scoring{method,tolerance,units,
        partialCredit,caseSensitive,acceptedAnswers[]}, rationale{overall,perChoice}, alignment{
        oerSourceId,learningObjectiveId,standardId,bloomLevel}, difficulty{estimate,basis}, readingLevel,
        provenance{sourceUrl,license,retrievedAt,attribution}, license, accessibility{...}, locale`.
  - [ ] Bank/blueprint model defined: `items[]` + `bankMetadata{unit,oerSource,blueprint,license,
        coverage{before,after}}`, where the blueprint fixes per-objective target counts/difficulty.
  - [ ] Enumerates the supported item types (single/multi MCQ, true/false, numeric, short-text, cloze,
        matching, ordering) and which `scoring.method` applies to each.
  - [ ] States deliverable is assessment content (not a runtime), output licensed CC-BY-4.0
        (CC-BY-SA-4.0 when derived from SA), code MIT.
  - [ ] At least one filled worked-example item (per type) committed as a fixture skeleton.

- **grader-002 (auto-grader scoring spec)**
  - [ ] One deterministic scoring function specified per item type; numeric uses `tolerance` + optional
        unit normalization; short-text uses Unicode-normalized/trimmed/case-folded match vs `acceptedAnswers`.
  - [ ] **Partial-credit formula standardized** for multiple-select and matching/ordering, with the
        chosen formula documented and a mapping note for QTI 3.0 and Moodle (flag where native models differ).
  - [ ] Spec is versioned (`scoringSpecVersion`) and recorded in `specVersions`.
  - [ ] Defines the self-test contract: published key → 100%; ≥ 1 enumerated wrong answer → 0;
        partial responses → defined partial value.

- **gate-003 (license + provenance + PII gate)**
  - [ ] Enumerates accepted licenses (PD/CC0, CC-BY, CC-BY-SA, OGL) and excluded ones (ND categorically;
        NC deferred to `policy-004`; all-rights-reserved/unclear excluded).
  - [ ] Objective criterion: PASS only if `license.permitsDerivatives:true` is set from a cited clause/URL;
        missing/unparseable/ND = FLAG/EXCLUDE (no default-allow).
  - [ ] Records license id + URL + text snapshot (committed copy + SHA-256 + Wayback URL) per source and
        per reused asset.
  - [ ] PII screen: no real personal data may be embedded; reused passages/datasets screened; any signal
        → rework or exclude; scenarios must use synthetic, bias-screened contexts.
  - [ ] Produces a committed PASS/FLAG/EXCLUDE artifact per OER unit recording which checks ran.

- **validator-006 (validator + auto-grade self-test harness)**
  - [ ] Validates every item against the JSON schema and runs the grader self-test (key→100%, wrong→0,
        partial→defined value) in CI.
  - [ ] Quality lint: duplicate/implausible distractors, answer-position balance, "all/none of the above"
        hygiene; reading-level check; accessibility lint (alt-text presence, no color-only cue, MathML math).
  - [ ] Ships golden fixtures (synthetic items only — never restricted source text): valid items that must
        pass and malformed/wrong-key items that must fail.
  - [ ] Code MIT-licensed; `pnpm build && pnpm test && pnpm lint` green; commit DCO signed-off.

- **pilot-009 (pilot bank, end-to-end)**
  - [ ] Pilot OER unit chosen on a realistic adoption path first (informal channel, or self-serve
        fallback = GitHub PR to the OER repo / self-published openly licensed release) and a clearly
        permissive license + non-regulated subject.
  - [ ] Source passed gate-003 (permits derivatives; no PII) with the artifact committed.
  - [ ] Items authored in the canonical model with correct keys + rationales (overall + per-distractor);
        coverage ≥ 90% of the unit's stated objectives (recorded before/after in the blueprint).
  - [ ] Machine gate green (schema valid + self-test 100% + lints pass) in CI.
  - [ ] Human sign-offs recorded: SME (key+rationale), license, accessibility, bias.
  - [ ] Bank **adopted** via the informal channel or self-serve fallback with the Steward's adoption
        evidence artifact (`outcomes/<bank-id>.json`) — or **submitted** with the blocker surfaced.

**M0 Definition of Done:** canonical item model + schema + bank/blueprint model published; auto-grader
scoring spec (incl. standardized partial-credit) published; license/PII gate + NC/SA policy published
(policy merged before any authoring); validator + self-test harness green in CI with golden fixtures;
mandatory reviewers named (≥ 1 SME for the pilot's subject, plus a11y/bias/license); one pilot bank
authored end-to-end, machine gate green, all human sign-offs recorded, and **adopted** via an informal
channel or self-serve fallback (evidence recorded) — or submitted with the blocker surfaced; ≥ 1
partner-outreach thread opened.

---

## Milestone M1 — Quality gate hardened + exporters + first adoptions

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| quiz-bank-open-qti-010 | QTI 3.0 exporter (canonical → QTI 3.0) | code | medium | low | pr | model-001, grader-002 | Technical |
| quiz-bank-open-moodle-011 | Moodle GIFT + Moodle XML exporters | code | medium | low | pr | model-001, grader-002 | Technical |
| quiz-bank-open-snapshot-012 | License-text snapshot capture in provenance flow | code | small | low | pr | gate-003 | Technical |
| quiz-bank-open-partner-013 | Secure first confirmed adoption partner | research | small | low | document | outreach-008 | Steward |
| quiz-bank-open-bank-014 | Question bank #2 end-to-end (math, low risk) | writing | medium | low | document | pilot-009, qti-010, moodle-011 | SME, License, Accessibility, Bias |
| quiz-bank-open-bank-015 | Question bank #3 end-to-end (intro science, low risk) | writing | medium | low | document | pilot-009, qti-010, moodle-011 | SME, License, Accessibility, Bias |

**Acceptance criteria — key tasks**

- **qti-010 (QTI 3.0 exporter)** *(pattern also applies to moodle-011)*
  - [ ] Projects a canonical bank to valid **QTI 3.0** (assessmentItem + responseDeclaration +
        outcomeDeclaration), validated against the pinned spec; moodle-011 emits GIFT + Moodle 4.x XML.
  - [ ] Preserves scoring semantics where the target supports them; any scoring rule the format cannot
        express is **explicitly flagged**, never silently dropped.
  - [ ] Ships golden input→output fixtures diffed in CI; output schema/format validated.
  - [ ] At least one **real platform import round-trip** verified (Moodle import of GIFT/XML; QTI import
        into a QTI-consuming platform or the QTI reference validator).
  - [ ] Code MIT-licensed; tests + CI green; no credentials embedded.

- **partner-013 (first confirmed adoption partner)**
  - [ ] A named OER platform/teacher confirms they will review and adopt contributed banks.
  - [ ] Contribution mechanism documented (platform import vs. repo PR vs. release).
  - [ ] Tasks for that partner updated to `verifiedNeed: true` with `requestor` set.

- **bank-014 / bank-015 (banks #2 and #3)**
  - [ ] Source passed gate-003; coverage ≥ 90% of stated objectives; machine gate green.
  - [ ] All human sign-offs recorded (SME key+rationale, license, accessibility, bias).
  - [ ] Exported to QTI 3.0 + Moodle (GIFT/XML) and **adopted** by a beneficiary, with the Steward's
        adoption evidence artifact recorded.
  - [ ] Per-bank authoring+review effort logged in the outcome ledger (baseline for M2 reduction).

**M1 Definition of Done:** QTI 3.0 + Moodle (GIFT/XML) exporters working, pinned-spec-validated with
golden fixtures and ≥ 1 real import round-trip; license/PII gate + bias + a11y lints applied to ≥ 3
banks; ≥ 2 banks adopted (evidence recorded); ≥ 1 confirmed partner; self-test 100% pass and **zero
answer-key defects** found in review across delivered items.

---

## Milestone M2 — Scale across OER units + standards alignment

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| quiz-bank-open-case-016 | CASE 1.1 standards-alignment for ≥ 1 open framework | code | medium | medium | pr | model-001 | License, Technical |
| quiz-bank-open-itemtypes-017 | Author + self-test advanced item types (matching/ordering/cloze/multi-select) | writing | medium | low | document | grader-002, validator-006 | SME, Technical |
| quiz-bank-open-banks-018 | Question banks #4–#6 across ≥ 2 subjects | writing | large | low | document | bank-014, bank-015, qti-010, moodle-011, partner-013 | SME, License, Accessibility, Bias |
| quiz-bank-open-effort-019 | Throughput/effort tracking + per-bank baseline reduction | research | small | low | document | bank-014, bank-015 | Maintainer |

**Acceptance criteria — key tasks**

- **case-016 (CASE standards alignment)**
  - [ ] The chosen framework's **license is verified** to permit redistribution of identifiers/derivatives
        (e.g. Common Core CC-BY-style); restricted frameworks are reference-only or omitted.
  - [ ] Implements CASE 1.1 alignment in `alignment.standardId`; alignment is optional metadata and its
        absence never blocks a bank.
  - [ ] Golden fixtures + CI; no restricted standard text reproduced where the license forbids it.

- **banks-018 (banks #4–#6)**
  - [ ] Each source passes gate-003; coverage ≥ 90% of stated objectives; machine gate green; all human
        sign-offs recorded.
  - [ ] Banks span ≥ 2 subjects; each adopted with evidence artifact recorded.
  - [ ] Any regulated-domain subject matter is escalated to high tier with expert sign-off + "not advice"
        framing, or excluded.

**M2 Definition of Done:** CASE alignment implemented for ≥ 1 license-verified open framework; advanced
item types authored + self-tested in delivered banks; ≥ 6 banks adopted cumulatively across ≥ 2
subjects; measurable median per-bank effort reduction vs. the recorded M0/M1 baseline.

---

## Milestone M3 — Reuse outcomes & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| quiz-bank-open-reuse-020 | Track + verify external reuse events | research | small | low | document | bank-014, bank-015, banks-018 | Steward |
| quiz-bank-open-refresh-021 | OER source-drift detection + refresh process | maintenance | small | low | document | validator-006 | Maintainer |
| quiz-bank-open-banks-022 | Question banks #7–#10 across ≥ 3 subjects | writing | large | low | document | banks-018, refresh-021 | SME, License, Accessibility, Bias |

**Acceptance criteria — key tasks**

- **reuse-020 (reuse tracking)**
  - [ ] ≥ 3 verifiable external reuse events recorded (platform import record / merged PR / teacher
        attestation / fork-citation), each with externally verifiable evidence (no self-reported reuse).

- **refresh-021 (source-drift + refresh)**
  - [ ] Documented process to detect when a bank's OER source version has changed.
  - [ ] Re-validation step flags affected items; stale items become `maintenance` tasks; answer-key
        defect reports routed as high-priority maintenance.

**M3 Definition of Done:** ≥ 3 verifiable reuse events; ≥ 10 banks adopted cumulatively across ≥ 3
subjects; documented source-drift/refresh process and a named steward for ongoing partner liaison and
outcome tracking.

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| quiz-bank-open-h5p-023 | H5P question-type exporter | code | medium | low | pr | If H5P-based OER platforms enter scope |
| quiz-bank-open-anki-024 | Anki / spaced-repetition deck exporter | code | small | low | pr | Overlaps `open-flashcards`; coordinate to avoid duplication |
| quiz-bank-open-i18n-025 | Localize a delivered bank (translation track) | translation | medium | medium | translation | Hands canonical bank to a translation-track project; needs language + SME reviewer |
| quiz-bank-open-expert-026 | Expert-reviewed bank for a regulated subject (health/safety facts) | writing | large | high | document | "Educational, not advice"; credentialed expert sign-off required |
| quiz-bank-open-dash-027 | Outcome dashboard (adoptions + reuse + key-defect rate) | code | medium | low | pr | Reads the outcome ledger; supports success-metric tracking |
| quiz-bank-open-difficulty-028 | Lightweight difficulty/discrimination calibration from voluntary aggregate response data | research | medium | medium | document | Aggregate/de-identified only; never high-stakes calibration claims |

---

## Example task JSON

```json
{
  "id": "quiz-bank-open-model-001",
  "title": "Canonical item model + JSON schema + bank/blueprint model",
  "project": "quiz-bank-open",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["education", "oer", "assessment", "open-content"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "medium",
  "status": "open",
  "context": "Open Educational Resources provide free learning content but rarely ship with trustworthy, auto-gradable question banks. Before authoring any bank, the project needs one canonical item model and JSON schema that all export formats (QTI 3.0, Moodle GIFT, Moodle XML, CSV/JSON) are projections of, plus a bank/blueprint model that fixes per-objective coverage. The deliverable is assessment content and light tooling, never a delivery runtime, and never items derived from non-open or No-Derivatives sources.",
  "objective": "Define the canonical item model, its JSON schema, and the bank/blueprint model that every per-unit authoring task and every exporter will use.",
  "acceptanceCriteria": [
    "JSON schema defines all item fields: id, type, stem (Markdown+MathML), assets[]{ref,altText,license,attribution,sourceUrl}, choices[]{id,text,correct}, answerKey, scoring{method,tolerance,units,partialCredit,caseSensitive,acceptedAnswers[]}, rationale{overall,perChoice}, alignment{oerSourceId,learningObjectiveId,standardId,bloomLevel}, difficulty{estimate,basis}, readingLevel, provenance{sourceUrl,license,retrievedAt,attribution}, license, accessibility{altTextPresent,plainLanguage,noColorOnlyCue,mathFormat}, locale.",
    "Bank/blueprint model defines items[] plus bankMetadata{unit,oerSource,blueprint,license,coverage{before,after}}, where the blueprint records each stated learning objective with target item count and difficulty spread (fixing the coverage denominator).",
    "Supported item types enumerated (single-select MCQ, multiple-select MCQ, true/false, numeric, short-text, fill-in-the-blank/cloze, matching, ordering) with the scoring.method applicable to each.",
    "Document states the deliverable is assessment content (not a runtime/LMS), item/bank output is CC-BY-4.0 (CC-BY-SA-4.0 when derived from a share-alike source), and code is MIT.",
    "At least one filled worked-example item per type is committed as a fixture skeleton.",
    "pnpm build && pnpm test && pnpm lint pass for any committed schema/tooling; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\elyos\\planning\\projects\\quiz-bank-open\\PLAN.md",
    "C:\\code\\elyos\\docs\\good-deed-definition.md",
    "1EdTech QTI 3.0 specification",
    "Moodle GIFT and Moodle XML import format docs",
    "1EdTech CASE 1.1 specification",
    "Bloom's revised taxonomy"
  ],
  "output": "A canonical item model definition plus its JSON schema and the bank/blueprint model, committed to the project repo and ready for reuse by per-unit authoring tasks and all format exporters.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```

---

## Notes on the example banks

- The pilot and M1/M2 banks deliberately target **non-regulated, clearly-permissive** OER units
  (middle-school math, intro science) to keep the first runs `riskTier: low` and prove the pipeline.
- A **regulated-subject** bank (health/safety facts) is intentionally held in the backlog
  (`expert-026`) at `riskTier: high` until a credentialed expert reviewer is secured; it is framed
  "educational, not advice."
- All authoring tasks remain `verifiedNeed: false` and `requestor: "TO BE SECURED"` until a named OER
  platform/teacher confirms adoption (`partner-013` flips the relevant tasks to `verifiedNeed: true`).
- Banks derived from a **CC-BY-SA** source carry `outputLicense: "CC-BY-SA-4.0"`; all other banks are
  `CC-BY-4.0`; all code tasks are `MIT`.

---

## Generated task index

> Auto-generated by Elyos task-decomposition agent on 2026-06-29. 28 schema-valid task files in `tasks/`.

| File | Title | Type | Deliverable | Priority | Risk |
| --- | --- | --- | --- | --- | --- |
| [quiz-bank-open-model-001.json](tasks/quiz-bank-open-model-001.json) | Canonical item model + JSON schema + bank/blueprint model | design-spec | document | high | low |
| [quiz-bank-open-grader-002.json](tasks/quiz-bank-open-grader-002.json) | Auto-grader scoring spec for core item types (incl. partial credit) | design-spec | document | high | low |
| [quiz-bank-open-gate-003.json](tasks/quiz-bank-open-gate-003.json) | OER license + provenance + PII triage checklist (blocking gate) | design-spec | document | high | medium |
| [quiz-bank-open-policy-004.json](tasks/quiz-bank-open-policy-004.json) | NC / share-alike / license-compatibility acceptance policy | design-spec | document | high | medium |
| [quiz-bank-open-a11ybias-005.json](tasks/quiz-bank-open-a11ybias-005.json) | Accessibility + bias/sensitivity review checklist | design-spec | document | high | low |
| [quiz-bank-open-validator-006.json](tasks/quiz-bank-open-validator-006.json) | Item validator + auto-grade self-test harness (CI) | code | pr | high | low |
| [quiz-bank-open-reviewers-007.json](tasks/quiz-bank-open-reviewers-007.json) | Name/secure mandatory reviewers (SME per subject, a11y, bias, license) | research | document | high | low |
| [quiz-bank-open-outreach-008.json](tasks/quiz-bank-open-outreach-008.json) | OER partner/platform outreach + pilot unit shortlist | research | document | medium | low |
| [quiz-bank-open-pilot-009.json](tasks/quiz-bank-open-pilot-009.json) | Pilot question bank end-to-end for one OER unit | writing | document | high | low |
| [quiz-bank-open-qti-010.json](tasks/quiz-bank-open-qti-010.json) | QTI 3.0 exporter (canonical → QTI 3.0) | code | pr | high | low |
| [quiz-bank-open-moodle-011.json](tasks/quiz-bank-open-moodle-011.json) | Moodle GIFT + Moodle XML exporters | code | pr | high | low |
| [quiz-bank-open-snapshot-012.json](tasks/quiz-bank-open-snapshot-012.json) | License-text snapshot capture in provenance flow | code | pr | medium | low |
| [quiz-bank-open-partner-013.json](tasks/quiz-bank-open-partner-013.json) | Secure first confirmed adoption partner | research | document | high | low |
| [quiz-bank-open-bank-014.json](tasks/quiz-bank-open-bank-014.json) | Question bank #2 end-to-end (math, low risk) | writing | document | medium | low |
| [quiz-bank-open-bank-015.json](tasks/quiz-bank-open-bank-015.json) | Question bank #3 end-to-end (intro science, low risk) | writing | document | medium | low |
| [quiz-bank-open-case-016.json](tasks/quiz-bank-open-case-016.json) | CASE 1.1 standards-alignment for >= 1 open framework | code | pr | medium | medium |
| [quiz-bank-open-itemtypes-017.json](tasks/quiz-bank-open-itemtypes-017.json) | Author + self-test advanced item types (matching/ordering/cloze/multi-select) | writing | document | medium | low |
| [quiz-bank-open-banks-018.json](tasks/quiz-bank-open-banks-018.json) | Question banks #4–#6 across >= 2 subjects | writing | document | medium | low |
| [quiz-bank-open-effort-019.json](tasks/quiz-bank-open-effort-019.json) | Throughput/effort tracking + per-bank baseline reduction | research | document | low | low |
| [quiz-bank-open-reuse-020.json](tasks/quiz-bank-open-reuse-020.json) | Track + verify external reuse events | research | document | medium | low |
| [quiz-bank-open-refresh-021.json](tasks/quiz-bank-open-refresh-021.json) | OER source-drift detection + refresh process | maintenance | document | medium | low |
| [quiz-bank-open-banks-022.json](tasks/quiz-bank-open-banks-022.json) | Question banks #7–#10 across >= 3 subjects | writing | document | medium | low |
| [quiz-bank-open-h5p-023.json](tasks/quiz-bank-open-h5p-023.json) | H5P question-type exporter | code | pr | low | low |
| [quiz-bank-open-anki-024.json](tasks/quiz-bank-open-anki-024.json) | Anki / spaced-repetition deck exporter | code | pr | low | low |
| [quiz-bank-open-i18n-025.json](tasks/quiz-bank-open-i18n-025.json) | Localize a delivered bank (translation track) | writing | translation | low | medium |
| [quiz-bank-open-expert-026.json](tasks/quiz-bank-open-expert-026.json) | Expert-reviewed bank for a regulated subject (health/safety facts) | writing | document | low | high |
| [quiz-bank-open-dash-027.json](tasks/quiz-bank-open-dash-027.json) | Outcome dashboard (adoptions + reuse + key-defect rate) | code | pr | low | low |
| [quiz-bank-open-difficulty-028.json](tasks/quiz-bank-open-difficulty-028.json) | Lightweight difficulty/discrimination calibration from voluntary aggregate response data | research | document | low | medium |
