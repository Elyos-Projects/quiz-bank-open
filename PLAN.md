# PLAN — quiz-bank-open

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## Executive summary

Open Educational Resources (OER) — openly licensed textbooks, modules, and courseware — give the
world free *content*, but they are chronically short of the thing that turns reading into learning:
**good assessment**. Teachers and self-learners can find an openly licensed chapter on photosynthesis
or linear equations, but rarely a matching set of **auto-gradable practice questions with correct
answer keys and explanations** they can actually trust and drop into a learning platform. The result
is that the most resource-constrained learners — the ones OER is meant to serve — get content without
practice, feedback, or a way to check understanding.

`quiz-bank-open` produces **open, auto-gradable question banks aligned to specific OER units**, each
item shipped with (a) a deterministic, machine-checkable **answer key**, (b) a **rationale** that
explains why the key is correct and why each distractor is wrong, and (c) **provenance + alignment
metadata** tying it to the OER learning objective (and, where licensing permits, a curriculum
standard) it assesses. Banks are authored once in a **canonical item model** and exported to the
interoperable formats real platforms consume — **1EdTech QTI 3.0, Moodle GIFT, Moodle XML**, and a
plain CSV/JSON — so a teacher can import them into Moodle, Canvas, or an OER platform without
rekeying.

The **deliverable is assessment content + light tooling** (a schema, validators, format exporters,
and an auto-grade self-test harness). The central design principle — and the project's hardest
constraint — is **answer-key correctness**: a question bank with a wrong key is not a neutral failure,
it actively *teaches the wrong thing*. So every item is gated on a machine-checkable self-test (the
published key must score full marks through the grader; a known-wrong answer must score zero) **and** a
human subject-matter sign-off. The second hard constraint is the **OER license/provenance gate**: any
source passage, figure, or dataset reused in an item must be openly licensed in a way that permits
derivatives (CC-BY, CC-BY-SA, CC0, or public domain); No-Derivatives (ND) and unverified sources are
excluded, not guessed at.

Risk tier is **low** overall, with two elevated paths handled explicitly: (1) any item whose *subject
matter* touches health, legal, safety, or financial facts is escalated to **medium/high**, framed as
"educational, not advice," and routed to a credentialed expert; (2) bias/fairness and accessibility
are treated as first-class review gates, not afterthoughts, because assessment content can encode
stereotypes or exclude disabled learners if authored carelessly.

## Problem & beneficiaries

**Who is helped.**
- **Self-directed learners** (including unschooled adults, learners in low-income regions, and exam
  preppers) who have free OER content but no trustworthy way to practice and check understanding.
- **Teachers in under-resourced schools** who lack time to author and validate question banks and
  cannot afford commercial item banks locked behind paywalls.
- **OER platforms and authors** (OpenStax, libretexts, Oppia, Kolibri/Learning Equality, CK-12,
  Saylor, and similar) whose excellent content lacks an open assessment layer.
- **Translators and accessibility communities** downstream: a clean canonical item model with
  separated stem/choices/rationale is far cheaper to translate and to render accessibly than prose.

**The verified need.** The *general* need is well established: the absence of openly licensed,
quality-assured, auto-gradable assessment aligned to OER is a widely cited gap, and "content without
assessment" is a recognized barrier to OER adoption in formal instruction. We treat the general need
as real. However, the **per-bank, per-partner need is TO BE SECURED**: we have **not** yet confirmed a
named OER platform, school, or teacher who has agreed to adopt contributed banks. Until a specific
beneficiary confirms they will review and use a bank, individual tasks carry `verifiedNeed: false`.
This honesty is required because Elyos's bar is "delivered, not merged" — a bank that is produced but
never adopted is not a good deed delivered.

**Partner org.** TO BE SECURED. Candidate channels include OER content publishers (OpenStax,
LibreTexts, CK-12, Saylor Academy), open learning platforms (Kolibri / Learning Equality, Oppia,
Moodle community), and individual teachers/teacher networks. M0 includes explicit partner-outreach
work; no partner is assumed, and no bank is authored against a unit until a realistic adoption path
exists for it (see cold-start de-risking in the roadmap).

## Goals and non-goals

**Goals**
- Define a **canonical item model + JSON schema** that cleanly separates stem, choices, answer key,
  scoring rule, rationale, alignment, provenance, and accessibility metadata — the single source of
  truth every export is a projection of.
- Specify and implement **deterministic auto-grading** for a defined set of objective item types, so
  "auto-gradable" is a tested property, not a claim.
- For each in-scope OER unit, deliver a complete, **answer-key-correct**, accessibility-clean,
  bias-reviewed question bank that measurably covers the unit's learning objectives.
- Make **answer-key correctness** a non-skippable gate: a machine-checkable self-test *plus* a human
  subject-matter sign-off before any bank ships.
- Make **OER license/provenance verification** a non-skippable, auditable gate before any item is
  authored from a source.
- Export banks to the interoperable formats real platforms consume (QTI 3.0, Moodle GIFT, Moodle XML,
  CSV/JSON) so adoption requires no rekeying.
- Provide alignment metadata to OER learning objectives and (where the standard's license permits) to
  curriculum standards via 1EdTech **CASE**.

**Non-goals**
- We do **not** build a quiz-delivery platform, LMS, proctoring system, or runtime test engine. We
  produce banks; existing platforms deliver them.
- We do **not** create or grade **subjective / free-response / essay** items, and we do **not** use an
  LLM as a runtime grader. "Auto-gradable" means deterministic, rule-based scoring only.
- We do **not** produce **high-stakes certification or admissions exams**, and we do not claim
  psychometric calibration (IRT/equating) for high-stakes use.
- We do **not** republish, mirror, or re-host closed or non-derivative source content; items reuse
  source material only when its license permits derivatives, with attribution.
- We do **not** author items from sources with unclear, No-Derivatives (ND), or all-rights-reserved
  licenses — these are flagged and excluded, never best-guessed.
- We do **not** collect, store, or include learner/student personal data; word problems use synthetic,
  bias-screened names and contexts.
- We do **not** auto-publish to any platform; a human contributes after review.
- We do **not** give medical/legal/financial/safety **advice**; items in those subject areas are
  educational only, "not advice," and require expert sign-off (medium/high tier).

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics ("items written") are explicitly excluded; an
item that is authored but never adopted, or that ships with a wrong key, counts against us.

| Metric | Baseline | Target (first 6 months) |
| --- | --- | --- |
| Question banks **adopted by a beneficiary** (imported into a platform / used by a confirmed teacher or OER partner, last-mile delivered) | 0 | ≥ 6 banks adopted |
| **Answer-key defect rate in delivered banks** (wrong key or wrong rationale found after delivery, per 100 items) | n/a | **0** key defects; < 1 rationale clarification per 100 items |
| Auto-grade self-test pass rate (published key scores 100%, a known-wrong answer scores 0) across delivered items | n/a | **100%** of delivered items pass the self-test |
| Learning-objective coverage per delivered unit (objectives with ≥ 1 valid item / total stated objectives) | n/a | ≥ 90% of each unit's stated objectives covered |
| Accessibility conformance (items passing the a11y checklist: alt text, no color-only cues, screen-reader-clean math) | n/a | 100% of delivered items pass |
| Reuse signal: delivered bank imported/forked/cited by a third party (verifiable) | 0 | ≥ 3 verifiable external reuse events |
| Confirmed adoption partners (platforms/teachers committed to using banks) | 0 | ≥ 2 secured |
| Bias/sensitivity review escapes (stereotyped/exclusionary content found post-delivery) | n/a | 0 |

Notes on attribution of outcomes: an "adoption" or "reuse event" must be externally verifiable (a
merged PR into an OER repo, a platform import record/permalink, a teacher's written confirmation of
classroom use, a citation/fork). Self-reported use does not count. **"Coverage" is measured against
the OER unit's *stated* learning objectives**, recorded at authoring time in the bank's blueprint, so
the denominator is fixed and auditable rather than retrofitted.

## Scope

**In scope**
- A canonical item model + JSON schema and a **bank/blueprint model** (the set of items mapped to a
  unit's learning objectives, with target counts and difficulty spread).
- Objective, auto-gradable item types only: **single-select MCQ, multiple-select MCQ (with defined
  partial-credit rules), true/false, numeric (with tolerance + units), short-text (normalized /
  accepted-answers match), fill-in-the-blank / cloze, matching, and ordering.**
- For every item: a deterministic scoring rule, a correct answer key, an overall rationale, and
  per-distractor explanations.
- Format exporters: **QTI 3.0**, **Moodle GIFT**, **Moodle XML**, and canonical **CSV/JSON**.
- Validators: schema validation, **auto-grade self-test harness**, item-quality lint (e.g.
  duplicate/implausible distractors, "all/none of the above" hygiene, answer-position balance),
  reading-level check, and an accessibility lint.
- Alignment metadata to OER learning objectives and, where the framework's license permits, to
  curriculum standards via 1EdTech **CASE**; plus Bloom's-level and difficulty-estimate tags.
- License/provenance/PII triage and recording for every source used.

**Out of scope**
- Any test-delivery, proctoring, adaptive-engine, or LMS runtime (we emit content, not a player).
- Subjective/essay items and any AI-as-grader scoring.
- High-stakes certification/admissions exam production; psychometric calibration claims for
  high-stakes use.
- Hosting, mirroring, or republishing source content; deriving items from ND / unclear / closed
  sources.
- Collecting or embedding real student/personal data.
- Automated, unattended publishing to any platform.
- Subject-matter **advice** in regulated domains (handled as educational-only, expert-gated content).
- Authoring banks for a unit before a realistic adoption path exists (avoids producing unused content).

## Solution approach & architecture

This is a **content + light-software project** (a schema, validators, exporters, and a self-test
harness), not a runtime service and not a data pipeline that moves data.

**Authoring pipeline (per bank)**
1. **Source & gate** — identify the OER unit, its source content, publisher, license, and any reused
   asset's license. PASS only if every reused source permits derivatives (CC-BY/CC-BY-SA/CC0/PD) and
   contains no PII. Record the decision as a committed gate artifact.
2. **Blueprint** — extract the unit's stated learning objectives; define target item count, item-type
   mix, difficulty spread, and Bloom-level coverage per objective. The blueprint fixes the coverage
   denominator.
3. **Item authoring** — author items in the canonical model: stem, choices, answer key, scoring rule,
   rationale (overall + per-distractor), alignment (objective + optional CASE standard + Bloom),
   difficulty estimate, reading level, provenance, accessibility metadata.
4. **Auto-grade self-test** — run each item through the grader: the published key must score 100%, and
   at least one deliberately-wrong response must score 0 (and partial-credit items must score the
   defined partial value). This is machine-checkable and runs in CI.
5. **Quality + accessibility + bias lint** — automated checks (distractor hygiene, position balance,
   reading level, alt-text presence, no color-only cues) plus the bias/sensitivity checklist.
6. **Review** — subject-matter reviewer signs off key + rationale correctness; accessibility reviewer;
   bias/sensitivity reviewer; license reviewer. Regulated-domain items add a credentialed expert.
7. **Export** — project the canonical bank to QTI 3.0, Moodle GIFT, Moodle XML, and CSV/JSON.
8. **Contribute** — a human submits the bank to the platform/partner; the Steward records adoption.

**Canonical item model (source of truth; all exports are projections).** Per item:
`id`, `type`, `stem` (Markdown + MathML for math), `assets[] {ref, altText, license, attribution,
sourceUrl}`, `choices[] {id, text, correct}`, `answerKey`, `scoring {method, tolerance, units,
partialCredit, caseSensitive, acceptedAnswers[]}`, `rationale {overall, perChoice{}}`,
`alignment {oerSourceId, learningObjectiveId, standardId (CASE, optional), bloomLevel}`,
`difficulty {estimate: easy|medium|hard, basis}`, `readingLevel`, `provenance {sourceUrl, license,
retrievedAt, attribution}`, `license`, `accessibility {altTextPresent, plainLanguage, noColorOnlyCue,
mathFormat}`, `locale`. A **bank** wraps `items[]` with `bankMetadata {unit, oerSource, blueprint,
license, coverage{before,after}}`.

**Auto-grading model.** Each item type has one deterministic scoring function: single-select
(exact id), multiple-select (defined partial-credit, e.g. proportion correct minus proportion
incorrect, floored at 0), true/false (exact), numeric (value within `tolerance`, optional unit
normalization), short-text (Unicode-normalized, trimmed, case-folded match against `acceptedAnswers`),
cloze (per-blank scoring), matching/ordering (per-pair / Kendall-style partial credit). The scoring
spec is versioned; exporters must preserve equivalent scoring semantics in QTI/Moodle where the target
format supports them, and **flag** (not silently drop) any scoring rule a target format cannot express.

**Tech stack.** TypeScript, ESM, pnpm workspaces (per Elyos conventions). Validators, the self-test
harness, and exporters are small Node packages with minimal dependencies. Items authored as
JSON/Markdown; math as MathML (with LaTeX source retained for authoring). No runtime services;
everything runs locally or in CI. Code is MIT; content is CC-BY-4.0 (or CC-BY-SA-4.0 where derived
from a share-alike source).

**Pinned target spec versions** (so format/compat mitigations are real; recorded in `specVersions`,
bumped only via a deliberate task):
- **QTI** — 1EdTech **QTI 3.0** (validate exported items against the QTI 3.0 information model /
  examples; assessmentItem + responseDeclaration + outcomeDeclaration shape).
- **Moodle GIFT** — current GIFT format (Moodle 4.x import).
- **Moodle XML** — current Moodle question XML (Moodle 4.x).
- **CASE** — 1EdTech **CASE 1.1** for standards/competency alignment (used only for openly licensed
  frameworks; see licensing).
- **Bloom** — Bloom's revised taxonomy (controlled vocabulary for `bloomLevel`).

**Key decisions.**
- Canonical-model-first: never hand-maintain four parallel export formats.
- Answer-key correctness is enforced *both* by a machine self-test (CI) *and* a human SME sign-off —
  neither alone is sufficient.
- The license/PII gate is a *blocking* committed checklist artifact, not an informal judgement.
- Exporters are output-only; a human performs the actual platform contribution.
- Regulated-domain subject matter is escalated by tier and framed "not advice," never excluded
  silently nor shipped without an expert.

## Data, licensing & compliance

**This is a critical section.** Our deliverable is derivative educational content built *from* OER, so
licensing and provenance must be handled conservatively and per-source.

**Source licenses we accept (must permit derivative works):**
- **Public domain / CC0 1.0** — accepted; no attribution legally required (we still record provenance
  and attribute as courtesy).
- **CC-BY 4.0** (and compatible CC-BY versions) — accepted; attribution required; our derived bank is
  licensed CC-BY-4.0.
- **CC-BY-SA 4.0** — accepted **with share-alike propagation**: any bank derived from a CC-BY-SA
  source is itself licensed **CC-BY-SA-4.0**, recorded per bank. Mixing SA and non-SA sources in one
  item/bank is governed by the compatibility policy (`policy-004`).
- Government/open-government-licensed content that permits derivatives (e.g. OGL) — accepted with
  attribution.

**Excluded (flagged, never guessed):**
- **No-Derivatives (CC-BY-ND, CC-BY-NC-ND)** — questions are derivatives; ND forbids this → **excluded**.
- **NonCommercial (CC-BY-NC, CC-BY-NC-SA)** — deferred to the **NC/share-alike/compatibility policy
  (`policy-004`)**, which must be decided and merged before any authoring; default is exclude/escalate
  because downstream commercial reuse (e.g. a teacher at a fee-charging school) can be ambiguous.
- **All-rights-reserved, unclear, or unverifiable** licenses → flagged and excluded.

**Standards-alignment licensing (a real trap, handled explicitly).** Curriculum-standards frameworks
are **not** uniformly open. We align via CASE only to frameworks whose license permits redistribution
of identifiers/derivatives:
- **Common Core State Standards** — published under a CC-BY-style license (with attribution/branding
  conditions); usable with attribution.
- **Many national curricula / government standards** — often crown-copyright/OGL or public-domain;
  verified per framework.
- **NGSS and some proprietary frameworks** — usage terms restrict reproduction; we record only a
  *reference* where the license forbids embedding text, or omit alignment rather than reproduce
  restricted standard text. Standards alignment is **optional** metadata; absence never blocks a bank.

**Objective "permits derivatives" acceptance criterion.** A source PASSes only if its license is on
the accepted list (or accepted by `policy-004`) **and** `license.permitsDerivatives: true` is recorded
with a cited clause/URL. Missing evidence, an unparseable license, or ND terms = FLAG/EXCLUDE — never
default-allow.

**Provenance model.** Every item records: source OER unit URL, publisher, retrieval timestamp, source
version/edition, license id + URL + a captured license-text snapshot (committed copy + SHA-256 +
Wayback save URL), and the required attribution string. Reused **assets** (figures/images/datasets)
each carry their own license + attribution + alt text. Provenance is part of the committed deliverable.

**Privacy / PII stance.** Assessment content must contain **no real personal data**. Word problems and
scenarios use **synthetic, bias-screened** names, places, and contexts. If a source passage or dataset
reused in an item contains personal/identifying data, the item is reworked to remove it or the source
is excluded — we never embed real PII in a question. Any dataset reused in a numeric/data item must be
aggregate/de-identified and openly licensed (this project does not handle controlled-access data).

**Bias, fairness & sensitivity.** Items are screened for stereotypes, exclusionary assumptions
(culture/region/gender/disability/socioeconomic), trick wording, and construct-irrelevant difficulty
(e.g. unnecessarily complex English in a math item). This is a mandatory review gate, not optional.

**Attribution requirements.** Every bank attributes the source OER per its license, links to the
source, and states clearly that the *items* (not the source) are our contribution. **Output license:
CC-BY-4.0** for items/banks (or **CC-BY-SA-4.0** where derived from a share-alike source); **MIT** for
code (schema, validators, exporters, self-test harness).

## Quality, review & risk gates

**Risk tier: low** for general academic subjects (math, science, language arts, history, CS).
**Escalated** as follows:
- **medium** — items requiring domain accuracy where an error has real consequences, or any reuse of a
  sensitive/aggregate dataset, or NC/ambiguous-license judgement.
- **high** — items whose subject matter is health, legal, safety, or financial *fact* (even when framed
  educationally): require **credentialed expert sign-off** before merge and a visible "educational, not
  advice" framing. (Such content is in scope only with an expert; otherwise excluded.)

**Required review before a bank is "done":**
- **Subject-matter reviewer** (mandatory, every bank): confirms each item's **answer key and rationale
  are correct** and the item validly assesses its objective. This is the central accuracy gate.
- **License reviewer** (mandatory): confirms every source permits derivatives and provenance is
  recorded.
- **Accessibility reviewer** (mandatory): confirms alt text, no color-only cues, screen-reader-clean
  math, plain language.
- **Bias/sensitivity reviewer** (mandatory): confirms no stereotyped/exclusionary/trick content.
- **Credentialed expert** (for any health/legal/safety/finance subject matter): required sign-off;
  escalates the task to `riskTier: high`.

**Machine gate (CI) — what "auto-gradable" and "key-correct" actually mean.** Each bank ships with the
self-test harness exercised in CI: (1) the published answer key scores **100%** through the grader for
every item; (2) at least one enumerated **wrong** response scores **0**; (3) partial-credit items
score the **defined** partial value for defined partial responses; (4) every item validates against the
JSON schema; (5) quality lint (distractor hygiene, position balance, no orphan "all/none of the
above"), reading-level, and accessibility lint all pass; (6) every exporter round-trips the bank to
QTI 3.0 / Moodle GIFT / Moodle XML and the result validates against the pinned spec (with any
unexpressible scoring rule explicitly flagged, never silently dropped). Golden fixtures (synthetic
items, never restricted source text) back every validator and exporter.

**Definition of Shipped.** A question bank **adopted by a beneficiary** — imported into a platform
(import record/permalink), merged into an OER repo (merge commit), or confirmed in classroom use by a
named teacher (written confirmation) — with: verified source license(s) + recorded provenance; the
machine gate green; the human SME, accessibility, bias, and license sign-offs recorded; coverage ≥
90% of the unit's stated objectives; and (for regulated domains) expert sign-off. Producing the bank is
**not** shipped; recorded adoption by the beneficiary is. The Steward records a canonical adoption
evidence artifact (`outcomes/<bank-id>.json`: channel, URL/permalink, timestamp, coverage before/after).

## Roadmap & milestones

**M0 — Foundation & cold-start (thin)**
- Goal: build the canonical model, the grader spec, the gates, and prove the end-to-end flow on one
  pilot bank; secure the mandatory reviewer roles; begin partner outreach.
- **Cold-start de-risking (pilot selection).** To avoid authoring content nothing adopts, the pilot is
  gated on a realistic adoption path *before* authoring, in priority order: (a) an **informal channel**
  — a teacher or OER maintainer who has agreed to review/use a bank; failing that, (b) a **self-serve
  fallback** — a **GitHub PR to the OER source's own repo** or a self-published, openly licensed bank
  release we can ship ourselves (e.g. a tagged repo release with QTI/Moodle exports). The pilot must
  use one of these so M0 yields a real *adopted* outcome, not a "produced, unused" one. The pilot OER
  unit is chosen for a clearly-permissive license (CC-BY/CC0/PD) and a non-regulated subject (e.g. a
  middle-school math or intro-science unit) to keep the first run low risk.
- Exit criteria: (1) canonical item model + JSON schema + bank/blueprint model published; (2)
  auto-grader scoring spec for the core item types published; (3) the license/PII gate checklist and
  the NC/SA/compatibility policy (`policy-004`) exist and are applied to the pilot; (4) item validator
  + auto-grade self-test harness working in CI with golden fixtures; (5) mandatory reviewer roles
  (SME, accessibility, bias, license) named; (6) **one pilot bank** authored end-to-end for one OER
  unit, machine gate green, all human sign-offs recorded, and **adopted** via an informal channel or
  self-serve fallback (adoption evidence recorded) — or, if no channel materializes, **submitted** with
  the blocker surfaced; (7) ≥ 1 partner-outreach thread opened.

**M1 — Quality gate hardened + exporters + first adoptions**
- Goal: make the gates rigorous, ship the interoperable exporters, and get real banks adopted.
- Exit criteria: (1) QTI 3.0 and Moodle (GIFT + XML) exporters working, validated against pinned
  specs with golden fixtures and at least one real Moodle import round-trip; (2) license/PII gate +
  bias + accessibility lints codified as reviewable artifacts and applied to ≥ 3 banks; (3) ≥ 2 banks
  **adopted** by a beneficiary (adoption evidence recorded); (4) ≥ 1 confirmed adoption partner;
  (5) answer-key self-test = 100% pass on all delivered items, with zero key defects found in review.

**M2 — Scale across OER units + standards alignment**
- Goal: increase throughput across OER units and add (optional) curriculum-standards alignment.
- Exit criteria: (1) CASE-based standards alignment implemented for ≥ 1 openly licensed framework
  (e.g. Common Core), with the per-framework license verified; (2) additional item types (matching,
  ordering, cloze, multiple-select partial credit) authored and self-tested in delivered banks;
  (3) ≥ 6 banks adopted cumulatively across ≥ 2 subjects; (4) median per-bank authoring+review effort
  measurably reduced vs. the recorded M0/M1 baseline (logged in the outcome ledger).

**M3 — Reuse outcomes & sustainability**
- Goal: demonstrate real downstream reuse and a maintenance model that survives OER source updates.
- Exit criteria: (1) ≥ 3 verifiable external reuse events; (2) ≥ 10 banks adopted cumulatively across
  ≥ 3 subjects; (3) a documented **source-drift / refresh process** (detect when an OER source changes
  and re-validate affected items) and a named steward for ongoing partner liaison and outcome tracking.

Dependencies: M1 exporters depend on M0's canonical model + grader spec; M2 scale depends on M1's
hardened gates and exporters; M3 depends on a body of adopted banks from M1–M2.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above. Each
milestone has a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer`),
acceptance criteria for the most important tasks, and a milestone Definition of Done. A backlog of
sized-but-unscheduled tasks and one complete, schema-valid example Task JSON are included there.
Per-unit authoring tasks are instantiated from a shared bank template once an OER unit passes the
license/PII gate; `TASKS.md` includes concrete example per-unit tasks drawn from real OER sources.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the canonical model, grader spec, tooling, and backlog.
- **Subject-matter reviewer(s):** TBD (TO BE SECURED) — **mandatory, non-skippable** gate for
  answer-key + rationale correctness. Must be filled (≥ 1 qualified reviewer per subject in play)
  **before the M0 pilot bank is reviewed**; may rotate among ≥ 2 reviewers to avoid a bottleneck.
  Until named for a subject, banks in that subject stay `verifiedNeed:false` and cannot ship.
- **License reviewer:** TBD — confirms every source permits derivatives + provenance recorded.
- **Accessibility reviewer:** TBD — confirms a11y conformance per the checklist.
- **Bias/sensitivity reviewer:** TBD — confirms no stereotyped/exclusionary/trick content.
- **Credentialed expert reviewer(s):** engaged per-item for health/legal/safety/finance subject
  matter (high tier); without one, such items are excluded.
- **Steward (last-mile owner):** TBD — owns relationships with platforms/teachers, confirms adoption
  (the "delivered" signal), and records the outcome ledger. Critical because Shipped = adoption.
- **Partner / requestor:** TO BE SECURED — named OER platform(s), school(s), or teacher(s).

## Dependencies & integrations

- **External standards/specs (pinned — see Tech stack):** 1EdTech QTI 3.0, Moodle GIFT, Moodle XML
  (4.x), 1EdTech CASE 1.1, MathML, Bloom's revised taxonomy, SPDX license identifiers. Versions
  recorded in `specVersions` and bumped only via a deliberate task.
- **OER sources (TO BE SELECTED via the gate; none assumed in scope yet):** candidate publishers
  include OpenStax, LibreTexts, CK-12, Saylor, Siyavula, and openly licensed government curricula.
- **Platforms (output-only; no automated upload):** Moodle, Canvas (QTI import), Kolibri/Learning
  Equality, Oppia, and OER repos on GitHub/Zenodo.
- **Elyos pieces:** Task JSON schema (`packages/schema`), the donated-lane CLI workspace/PR flow
  (`packages/cli`), good-deed definition + refusal guardrails. No funded-lane/runner dependency (this
  project is donated lane). If a funded lane is ever used for bulk authoring, tasks add
  `fundedBudgetUsd` with a hard per-task cap.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| **Wrong answer key ships** (teaches the wrong thing) | Medium | High | Machine self-test (key scores 100%, wrong scores 0) **and** mandatory SME sign-off; both required before ship | Subject-matter reviewer |
| Rationale plausible but subtly wrong (misexplains why) | Medium | High | SME reviews rationale, not just key; per-distractor explanations required and checked | Subject-matter reviewer |
| Deriving items from ND / unclear / closed source | Medium | High | Mandatory license gate; `permitsDerivatives:true` with cited clause; ND = hard exclude | License reviewer |
| Reproducing restricted curriculum-standard text | Medium | Medium | CASE alignment only for open frameworks; reference-not-reproduce for restricted; alignment optional | License reviewer |
| Bias/stereotype or exclusionary content in items | Medium | High | Mandatory bias/sensitivity gate; synthetic bias-screened names; checklist artifact per bank | Bias reviewer |
| Inaccessible items (color-only cues, image-only math) | Medium | Medium | Accessibility lint + reviewer; MathML + alt text required; no color-only answers | Accessibility reviewer |
| Banks produced but never adopted (fails "delivered") | Medium | High | Cold-start adoption-path gate before authoring; partner outreach; Steward; `verifiedNeed:false` until secured | Steward |
| Auto-grade semantics lost on export (QTI/Moodle can't express a scoring rule) | Medium | Medium | Exporter flags (not drops) unexpressible rules; round-trip validation in CI; pin spec versions | Maintainer |
| OER source updated → items become stale/incorrect | Medium | Medium | Record source version; drift-detection/refresh process (M3); stale items become `maintenance` tasks | Maintainer |
| Regulated-domain item slips through without expert | Low | High | Subject-area triage routes health/legal/safety/finance to high tier + expert; "not advice" framing | Maintainer |
| PII embedded in a reused passage/dataset | Low | High | PII screen in gate; synthetic contexts only; rework or exclude on any signal | License reviewer |
| Scope creep into building a delivery/grading runtime | Medium | Low | Explicit non-goal; reviewers reject runtime/LMS work | Maintainer |

## Security & privacy

- **Threat surface is small** (no runtime service, no user data, no hosting). Main surfaces are CI and
  the published content/code.
- **Secrets handling:** validators/exporters need no credentials by default. If a platform API token is
  ever needed for a contribution, it is supplied by the human submitting and must never be written into
  logs, receipts, or committed files (per Elyos rules).
- **PII:** the project collects **no** learner data and embeds **no** real personal data in items;
  scenarios are synthetic and bias-screened. Reused sources are PII-screened in the gate; any signal
  triggers rework or exclusion.
- **Abuse/misuse prevention:** refuse and flag tasks that steer the bank toward producing exam-leak /
  cheating material for a specific live high-stakes exam, disinformation framed as quiz "facts",
  laundering closed content as open, or items targeting/demeaning a group. Content must remain
  educational, source-verified, and openly licensed. Regulated-domain content is educational-only,
  expert-gated, and never framed as advice.

## Sustainability & maintenance

- **Post-delivery ownership:** the Steward maintains platform/teacher relationships and the outcome
  ledger; the Maintainer keeps the canonical model, grader spec, validators, and exporters current with
  spec changes (QTI/Moodle/CASE version bumps via deliberate tasks).
- **Source drift / refresh:** each bank records its source version; a refresh process (M3) detects when
  an OER source changes and re-validates affected items; stale items become `maintenance` tasks.
- **Outcome tracking:** the Steward records adoption and external reuse events against the success
  metrics; reviewed each milestone. Answer-key defect reports are tracked as high-priority maintenance
  and feed back into the review checklist.

## Open questions

- Which specific OER platform(s)/teacher(s) will be the first confirmed adoption partner(s)?
- How do we accept NC and share-alike sources for derivative assessment content — blanket policy or
  per-source? (Default in `policy-004`: exclude/escalate; SA propagates to CC-BY-SA output.)
- For multiple-select and matching/ordering, which exact **partial-credit formula** do we standardize,
  and how do we map it onto QTI 3.0 and Moodle scoring where their native models differ?
- Which curriculum-standards frameworks are openly licensed enough to embed via CASE vs. reference-only?
- What is the minimum SME credential per subject, and how do we cover subjects where no reviewer is yet
  secured (block vs. defer the subject)?
- What counts as a sufficiently "verifiable" adoption/reuse event for the outcome metric (platform
  import record? merged PR? teacher attestation)?
- Do we localize the canonical model for translation (handing banks to translation-track projects), and
  if so, what is the i18n key strategy for stems/choices/rationales?

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap — `C:\code\elyos\planning\ROADMAP.md`
- House-style exemplar plan — `C:\code\elyos\planning\projects\open-data-datasheets\PLAN.md`
- 1EdTech QTI 3.0 (Question & Test Interoperability) specification
- Moodle GIFT and Moodle XML question import formats (Moodle 4.x docs)
- 1EdTech CASE 1.1 (Competencies & Academic Standards Exchange)
- Bloom's revised taxonomy; MathML; SPDX license list
- Creative Commons CC0 / CC-BY 4.0 / CC-BY-SA 4.0; Open Government Licence (OGL)
- OpenStax, LibreTexts, CK-12, Saylor Academy (candidate OER sources)

---

## Appendix A — Improvements applied

The following 25 improvements were identified against the first draft of this plan and **applied** to
the text above (they are not deferred suggestions).

1. **Dual answer-key gate.** Made key correctness enforced by *both* a machine self-test *and* a human
   SME sign-off, since neither alone catches all errors (machine can't catch a "correct-but-misaligned"
   item; humans miss edge cases). Reflected in Quality gates, metrics, and risks.
2. **Rationale-correctness as its own risk.** Added the distinct failure mode "key right but rationale
   subtly wrong" to the risk table and made SME review the rationale explicitly, not just the key.
3. **Coverage denominator fixed at authoring time.** Defined coverage against the unit's *stated*
   objectives recorded in the blueprint, so the metric can't be retrofitted to look complete.
4. **ND is a hard exclude, stated plainly.** Called out that questions are derivative works, so
   No-Derivatives sources are categorically excluded — a subtle license trap many overlook.
5. **Standards-licensing trap surfaced.** Added explicit handling that curriculum standards (NGSS,
   etc.) are not all open; CASE alignment is optional and reference-only where reproduction is barred.
6. **Share-alike propagation made concrete.** Specified that CC-BY-SA sources force CC-BY-SA output and
   that mixing is governed by `policy-004`, rather than hand-waving "CC content."
7. **"Auto-gradable" defined as a tested property.** Specified the self-test (key→100%, wrong→0,
   partial→defined value) so the headline claim is machine-checkable, not marketing.
8. **Partial-credit formula flagged as an open decision.** Added an open question and exporter caveat
   because QTI and Moodle express partial credit differently and silent mismatch corrupts scoring.
9. **Exporters must flag, not drop, unexpressible scoring.** Added the rule + a risk row so semantic
   loss on export is caught rather than silently shipping a mis-scoring bank.
10. **Real Moodle import round-trip required.** M1 exit criteria require at least one actual platform
    import, so "exporter works" isn't proven on golden files alone.
11. **Cold-start adoption-path gate.** Mirrored the exemplar's anti-"produced-but-unused" gate: no bank
    is authored for a unit before an informal channel or self-serve fallback exists.
12. **`verifiedNeed:false` until a partner is secured.** Stated honestly throughout that no adoption
    partner exists yet, per Elyos's "delivered, not merged" bar.
13. **Bias/sensitivity elevated to a mandatory gate.** Assessment content can encode stereotypes;
    added a dedicated reviewer role, checklist artifact, and metric (zero escapes).
14. **Accessibility elevated to a mandatory gate.** Added a11y lint + reviewer (alt text, no color-only
    cues, MathML for screen-readable math) with a 100% conformance target.
15. **No real PII in items.** Specified synthetic, bias-screened names/contexts and a PII screen on
    reused passages/datasets — closing a privacy gap unique to word problems.
16. **Regulated-domain escalation path.** Routed health/legal/safety/finance subject matter to
    high tier with expert sign-off and "not advice" framing, rather than treating all content as low.
17. **Subjective/essay + LLM-grader explicitly out of scope.** Prevented scope creep into
    non-deterministic grading that can't be auto-graded honestly.
18. **No-runtime/no-LMS non-goal.** Drew a hard line: we ship banks, not a delivery/proctoring engine,
    keeping the project sized for donated sessions.
19. **Canonical-model-first architecture.** Made one item model the source of truth with all four
    export formats as projections, so we never hand-maintain parallel formats.
20. **Pinned spec versions.** Pinned QTI 3.0, Moodle 4.x, CASE 1.1, MathML, Bloom so "compat" is real
    and drift is handled by a deliberate version-bump task.
21. **License-text snapshot format reused from house style.** Adopted committed copy + SHA-256 +
    Wayback for provenance, matching the org's decided format rather than linking a bare URL.
22. **SME role is a blocking prerequisite, per subject.** Required ≥ 1 qualified reviewer per subject
    *before* that subject's banks can ship, with rotation to avoid a bottleneck.
23. **Steward + adoption evidence artifact.** Defined Shipped as recorded adoption (`outcomes/<bank-id>.json`)
    and named a last-mile owner, so outcomes are tracked, not assumed.
24. **Source-drift/refresh maintenance.** Added an M3 process to detect OER source changes and
    re-validate affected items, preventing silent staleness.
25. **Outcome-based metrics with verifiability rules.** Replaced vanity counts with adoption,
    key-defect rate, coverage, a11y conformance, and externally verifiable reuse events.

---

## Review sign-off

**Reviewer:** senior staff engineer + TPM (self-review against PLAN_SPEC.md, CLAUDE.md, the
good-deed definition, and the Task JSON schema).

**Completeness check.** All 17 required H2 sections present and in order; metadata header matches the
global convention (Status/Version/Date/Owner/Lane). Success metrics are outcome-based with baselines +
targets. Scope states explicit out-of-scope items. The roadmap has M0–M3 with measurable exit
criteria, M0 thin/cold-start. The risk table uses the required columns. Appendix A applies 25 concrete
improvements. `TASKS.md` carries the schema-mapped backlog, milestone tables, acceptance criteria,
DoD, a backlog, and a schema-valid example Task JSON.

**Correctness / guardrail check.**
- *License/provenance:* open/PD/CC-with-derivatives only; ND hard-excluded; NC/SA via `policy-004`;
  share-alike propagation specified; standards-licensing trap handled; per-source snapshot + provenance.
  ✔
- *Privacy/PII:* no learner data collected; no real PII embedded; synthetic bias-screened contexts;
  reused-source PII screen. ✔
- *Non-partisan/sensitivity:* bias/sensitivity is a mandatory gate; refusal guardrails cover
  cheating/disinformation/closed-content laundering/targeting. ✔
- *Expert review for high-stakes:* health/legal/safety/finance subject matter escalated to high tier
  with credentialed expert + "not advice" framing. ✔
- *Schema validity:* the example Task JSON in `TASKS.md` uses only enum-valid values and all required
  fields; funded-lane note included though all tasks are donated. ✔
- *"Delivered, not merged":* Shipped = adoption with evidence artifact; `verifiedNeed:false` until a
  partner is secured; partner/requestor marked TO BE SECURED. ✔

**Fixes applied during review.** (1) Added the exporter "flag-not-drop" rule to both the architecture
and the risk table after noticing the metric claimed auto-grading but export could silently lose
scoring. (2) Made coverage's denominator explicitly the *stated* objectives to prevent retrofitting.
(3) Clarified SME role as per-subject blocking prerequisite (not a single global reviewer) so
multi-subject scaling doesn't stall on one person. (4) Tied the regulated-domain path to a `riskTier`
escalation in both Quality gates and the example/typed tasks for consistency.

**Residual items requiring a human decision:** (a) confirm the first adoption partner; (b) ratify the
NC/SA/compatibility policy (`policy-004`) and the standardized partial-credit formula; (c) confirm the
minimum SME credential per subject. These are tracked in Open questions and as M0/M1 tasks.

**Status:** Approved as a Draft (v0.1.0) for circulation; not yet ratified (pending the human
decisions above).
