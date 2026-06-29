# Competitive & Improvement Analysis — quiz-bank-open

Project: open, quality-reviewed, standards-aligned question/quiz banks (formative assessment items)
with verified answer keys, rationales, and interoperable exports.
Sources reviewed: PLAN.md (v0.1.0) + TASKS.md (M0 tables/acceptance criteria). Research current to June 2026.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually rigorous and already internalizes most assessment-quality literature. The
dual answer-key gate (machine self-test **and** human SME sign-off), the "auto-gradable = tested
property" framing, the ND-is-a-hard-exclude license stance, and the standards-licensing trap are
all genuinely strong and correctly prioritized. Findings below are gaps/risks, ordered by severity.

**Cardinal risk — answer-key/key-correctness (well handled, two residual holes).**
- The self-test contract (`key→100%, wrong→0, partial→defined`) only proves the grader agrees with
  the *declared* key. It does **not** detect a key that is internally consistent but factually wrong
  (e.g. a chemistry item keyed to an outdated value). The plan leans on SME review for this, which is
  correct — but there is no **second-independent-reviewer / blind re-key** step for the highest-value
  items. Industry item banks (NWEA) re-review large portions of the bank continuously; a single SME
  per subject is a single point of failure for the cardinal risk. Recommend: independent blind re-key
  for a sampled %, or two-SME sign-off for medium+ items.
- "At least one enumerated wrong answer scores 0" is a weak floor. For a 4-option MCQ it should assert
  **all** declared distractors score 0 and only the key scores full — otherwise a typo making two
  options "correct" can pass. Tighten the contract to exhaustively score every choice.

**Item-writing quality / distractor quality — partially specified, under-grounded.**
- The lint mentions "duplicate/implausible distractors, all/none-of-the-above hygiene, answer-position
  balance" but does not anchor to the established **Haladyna/Rodriguez item-writing guidelines** (22
  condensed rules across content/format/stem/choices). Distractor *plausibility* and *homogeneity of
  length/content* are the most cited flaw classes in the literature and are largely un-lintable —
  they need a rubric for human reviewers, not just regex lints. Recommend an explicit item-writing
  rubric citing Haladyna as the review standard, plus a "nonfunctional distractor" guideline (every
  distractor should be selected by some real misconception).
- **Missing: distractor-as-misconception design.** The biggest pedagogical differentiator in the
  market (Eedi/Diagnostic Questions) is that each distractor maps to a *named misconception*. The
  canonical model has `rationale.perChoice` but no first-class `misconceptionTag`/diagnostic field.
  This is a cheap, high-value schema addition (see optimizations).

**Standards alignment — correct stance, one operational gap.** Optional CASE alignment with
reference-not-reproduce for restricted frameworks is right. But the plan never says **who verifies an
alignment is correct** — mis-tagging an item to the wrong standard is a quieter version of the cardinal
risk (teachers filter by standard and get the wrong item). Alignment should be an SME-checked field,
not author-asserted metadata.

**Interoperability (QTI/GIFT/CSV) — the round-trip claim is optimistic.**
- QTI 3.0 and Moodle express partial credit and response processing very differently; the plan
  correctly flags "flag-not-drop unexpressible rules," but several listed item types (multiple-select
  partial credit, matching/ordering Kendall-style, cloze per-blank) have **no clean GIFT representation
  at all** — GIFT is deliberately simple. The plan should state up front which (type × format) cells
  are unsupported and downgrade gracefully, rather than implying full matrix coverage.
- "Round-trip validation" is asserted but the *direction* matters: exporting to QTI and re-importing
  is not the same as a real LMS rendering/scoring it correctly. M1's "one real Moodle import" is good;
  add at least one real **QTI 3.0** consumer (e.g. TAO or a QTI validator) since QTI is the headline
  format and has the lowest real-world import success rate.

**Accessibility — strong, but MathML reality check needed.** MathML for screen-readable math is
correct in principle, but native browser/LMS MathML support is uneven; retaining LaTeX is good, but the
plan should specify an accessible math fallback (e.g. MathJax-rendered with alt text / `aria` math) and
test with at least one real screen reader, not just lint for "altTextPresent."

**Avoiding low-quality AI-generated items — named but not operationalized.** The refusal/abuse section
covers disinformation but there is no explicit gate against the dominant 2026 failure mode: plausible-
but-wrong **AI-drafted items** that pass schema + self-test (because the key was set by the same model)
yet are subtly wrong or hallucinated. The dual gate mostly covers this, but the plan should state
explicitly that **Claude-drafted keys are never self-certified** and that provenance records when an
item was AI-drafted so reviewers calibrate scrutiny.

**Bias review — good, add a fairness-statistics note.** Bias review is a mandatory gate (excellent).
Since the project disclaims psychometrics, it cannot do DIF (differential item functioning) analysis —
fine — but it should say so explicitly and note that bias review is therefore *content-judgment only*,
a known limitation vs. calibrated banks.

**Scope — appropriately tight.** No-runtime, no-essay, no-LLM-grader, no-high-stakes are all correct
guardrails. One scope ambiguity: "short-text normalized match" quietly reintroduces grading
subjectivity (synonyms, spelling, language variants). Recommend bounding short-text to closed numeric/
single-token answers in M0–M1 and deferring open short-text, which is a perennial auto-grade trap.

---

## 2. Competitive landscape (researched, cited)

**OpenStax (assessment).** CC-BY open textbooks with section-level formative questions; assessments
exportable in **QTI**, but the richer instructor **quiz/test banks are gatekept behind faculty
accounts**, not openly licensed/openly downloadable. Strength: trusted, peer-reviewed content, real
QTI export, huge reach. Weakness: assessment layer is partly closed, college-level skew, not a reusable
open item *bank* you can fork. https://help.openstax.org/s/article/OpenStax-Assignable-assessment-questions
· https://openstax.org/books/algebra-1/pages/11-assessment-types-and-resources

**Khan Academy.** Massive, high-quality exercises with mastery feedback. **Licensing is the killer:
CC BY-NC-SA** (NonCommercial + ShareAlike) — by the plan's own (correct) policy this is largely
*excluded* as a source for openly reusable derivatives, and Khan content is not exportable as portable
items. Strength: scale, pedagogy. Weakness: NC license + walled garden = not open/interoperable.
https://support.khanacademy.org/hc/en-us/articles/202262954

**CK-12.** Free, Creative-Commons, **standards-aligned to Common Core + NGSS**, SME-reviewed, with
adaptive practice (Braingenie). Strength: closest "open + standards-aligned + assessment" incumbent.
Weakness: content lives inside the CK-12 platform; no canonical, interoperable item model you can
export to QTI/Moodle and self-host; CC variant/derivative terms are platform-bound.
https://help.ck12.org/hc/en-us/articles/200344205 · https://en.wikipedia.org/wiki/CK-12_Foundation

**Quizlet / Kahoot.** Enormous **crowdsourced** libraries; the defining weakness is **no accuracy
QA** — user-created sets are frequently wrong, and AI-generation features explicitly warn users to
verify outputs. No standards alignment, no answer-key guarantee, no open license, no interoperable
export. This is precisely the "wrong key teaches the wrong thing" failure the plan is built to avoid.
https://www.edweek.org/technology/4-things-educators-need-to-know-about-kahoot/2023/08 ·
https://support.kahoot.com/hc/en-us/articles/17152945038355

**Eedi / Diagnostic Questions.** The pedagogical gold standard for MCQ design: 1 correct + 3
distractors, **each distractor engineered to reveal a specific misconception**, validated on
219,826 responses showing strong alignment between student reasoning and writer intent. Strength:
diagnostic distractor design + research base + ML misconception mapping (NeurIPS 2020 challenge,
Kaggle competition). Weakness: proprietary, UK-curriculum/maths-centric, not openly licensed or
interoperable. *This is the quality bar to emulate, not a license competitor.*
https://help.eedi.co.uk/en/articles/7234087-the-research-behind-eedi ·
https://link.springer.com/article/10.1007/s10649-021-10084-7

**1EdTech QTI 3.0.** The interoperability standard itself (the rails, not a competitor): packages
items/tests portably across systems; HTML5, Portable Custom Interactions, certification + validators.
Strength: the right target. Weakness: heavyweight, real-world import fidelity is uneven, partial-credit
semantics vary by consumer — exactly the export risk above. https://www.imsglobal.org/spec/qti/v3p0/oview
· https://www.1edtech.org/standards/qti

**NWEA MAP (item bank exemplar).** 42,000+ items **IRT-calibrated to a vertical RIT scale**, continuous
item-quality review, **DIF analysis** for fairness. Strength: the psychometric quality ceiling.
Weakness: fully proprietary, adaptive-test-locked, not open. Shows what "quality" means at scale — the
plan deliberately (and correctly) does *not* compete on calibration, but can borrow the review
discipline. https://charts.intensiveintervention.org/screening/tool/?id=576cd73956493b98

**Learnosity.** Commercial assessment **API + item bank + open-source QTI 2.1 conversion library**;
rich question types, programmatic item authoring. Strength: best-in-class item-bank tooling and QTI
bridge (a model for our exporters). Weakness: proprietary SaaS, paid, not content. Their
open-source `learnosity-qti` converter is a useful reference implementation.
https://github.com/Learnosity/learnosity-qti · https://learnosity.com/products/questions/

**H5P.** **MIT-licensed** open-source interactive content (Multichoice, fill-blank, drag-drop, Question
Set); content default **CC-BY-4.0**; huge Moodle/WordPress install base. Strength: open, ubiquitous,
authoring-friendly. Weakness: H5P is a *content-type/runtime*, not a curated, standards-aligned,
key-verified item bank, and `.h5p` is not QTI — interoperability is platform-plugin-bound. Strong
**distribution partner**, not a content competitor. https://h5p.org/ · https://h5p.org/question-set

---

## 3. Gaps we can fill

No incumbent occupies the intersection the plan targets:
1. **Open + interoperable + key-verified.** CK-12/Khan are open-ish but platform-locked and (Khan)
   NC-licensed; Quizlet/Kahoot are open-access but unverified; OpenStax banks are partly gated.
   Nobody ships a *forkable, CC-BY, QTI/Moodle-portable, SME-verified* item bank.
2. **Assessment layer for OER content** (the plan's exact thesis). OpenStax/LibreTexts/Kolibri have
   content; a matching open auto-gradable practice layer is genuinely missing.
3. **Diagnostic distractors at open-license scale.** Eedi proves the value of misconception-mapped
   distractors but keeps them proprietary; an *open* misconception-tagged bank is unfilled.
4. **A reusable, open item-quality/answer-key linter.** Learnosity/QTI validators check *format*;
   nobody ships an open *pedagogical-quality + key-self-test* linter as standalone tooling.
5. **Auditable provenance + license verification** baked into the item model — uniquely valuable given
   the OER licensing minefield (Khan NC, NGSS reproduction limits) that the plan already maps.

---

## 4. Differentiators to win

1. **Verified-correct, openly-licensed, portable items** — the trust + freedom + interoperability
   triple that no incumbent offers together. This is the single strongest wedge.
2. **Misconception-mapped distractors** (Eedi-grade pedagogy, open license) — borrow the research,
   open the output.
3. **Canonical-model-first** with QTI 3.0 + Moodle + CSV projections — adopt-without-rekeying.
4. **Provenance/license as first-class, machine-checkable artifacts** — defensible OER derivation.
5. **Quality as tooling, not just content** — the open linter/self-test harness is reusable by the
   whole OER ecosystem (and seeds spin-offs).
6. **Outcome-honest delivery** ("adopted, not produced") differentiates from vanity OER dumps.

---

## 5. Claude API leverage (and where Claude must NOT decide)

**High-value Claude uses (draft/assist only):**
- **Item + distractor + rationale drafting from a gated source passage** — fast first drafts of stems,
  plausible distractors, overall + per-distractor explanations, given an openly-licensed source. Use
  structured tool-use/JSON output to emit directly into the canonical item model.
- **Misconception-driven distractor generation** — prompt Claude to propose, for each item, distractors
  tied to *named* common misconceptions (Eedi-style), with the misconception text as a draft field.
- **Item-quality linting / Haladyna review assistant** — Claude as a *reviewer-augmenting* linter:
  flag implausible/non-homogeneous distractors, cluing, double-barreled stems, reading-level spikes,
  all/none-of-the-above misuse — producing reviewer-facing findings, not pass/fail verdicts.
- **Alignment-tag drafting** — suggest learning-objective / CASE-standard / Bloom-level tags for SME
  confirmation.
- **Accessibility + plain-language rewriting** — draft alt text, simplify construct-irrelevant language,
  generate MathML/aria-friendly math from LaTeX.
- **Translation/localization drafting** of stems/choices/rationales (ties to translation spin-off).

**Where Claude must NOT decide (hard human gates — keep as in plan):**
- **Answer-key correctness** — never self-certified; a human SME (ideally independent of the drafter)
  verifies the key. Claude-set keys are *especially* suspect and must be blind-checked.
- **Item quality + bias/fairness** sign-off — human reviewer judgment.
- **No fabricated facts** — every factual claim traces to the gated source; no model-introduced data.
- **Standards-alignment correctness** — SME-verified, not author/AI-asserted.
- **License/derivative permission** — verified from cited clause/URL by the license reviewer; never
  inferred by the model.
- **Regulated-domain (health/legal/safety/finance) content** — credentialed expert, "not advice."

A safe pattern: Claude drafts → CI self-test + lint → human SME/license/a11y/bias sign-off → ship.
Record `aiDrafted: true` + model id in provenance so reviewers calibrate scrutiny.

---

## 6. Ten concrete optimizations

1. **Add a first-class `misconceptionTag` (+ optional id) per distractor** to the canonical model —
   turns the bank into a diagnostic instrument (Eedi-grade) at near-zero cost; powers analytics later.
2. **Tighten the self-test contract to score every choice** (all distractors→0, only key→full;
   multi-select asserts the exact partial vector) instead of "≥1 wrong answer→0."
3. **Adopt Haladyna/Rodriguez as the named item-writing rubric** and ship it as the reviewer checklist;
   map each lint rule to a guideline number for traceability.
4. **Independent blind re-key on a sampled %** (or two-SME for medium+ items) to close the
   single-reviewer single-point-of-failure on the cardinal risk.
5. **Publish a (type × export-format) support matrix** declaring unsupported cells (esp. GIFT for
   partial-credit/cloze/matching) and define graceful downgrade rather than silent loss.
6. **Add a real QTI 3.0 consumer to CI** (e.g. TAO / official QTI validator), not only Moodle, since
   QTI is the headline format with the weakest real-world import fidelity.
7. **Make alignment a verified field** — SME confirms learning-objective and CASE-standard tags; record
   verifier + date, same as key sign-off.
8. **Ship the item-quality + self-test harness as a standalone open package** (`item-lint`) usable by
   any OER project — reuse + adoption flywheel + spin-off seed.
9. **Bound short-text/auto-grade scope** to closed numeric/single-token answers in M0–M1; defer
   open/synonym short-text to avoid reintroducing grading subjectivity.
10. **Stamp provenance with `aiDrafted` + model/version and a math-accessibility fallback path**
    (MathJax/aria) tested with one real screen reader, so a11y is verified behavior not a lint pass.

---

## 7. Parallel & perpendicular spin-offs

- **open-flashcards** (parallel): the canonical item model's stem/choice/rationale separation projects
  cleanly to spaced-repetition cards; a single source-of-truth feeds both quiz banks and flashcards.
- **teacher-lesson-plans** (perpendicular): banks attach as the formative-assessment component of open
  lesson plans, aligned to the same learning objectives/standards.
- **oer-everything** (parallel): quiz-bank-open is the *assessment layer* of a broader open-OER push;
  shares the license/provenance gate and CASE alignment tooling.
- **numeracy-from-zero** (perpendicular): a direct consumer of **misconception-tagged** numeric/cloze
  items — the diagnostic distractors become the misconception engine for adaptive numeracy practice.
- **Reusable item-quality linter** (`item-lint`): standalone MIT package (Haladyna rules + key
  self-test + a11y + reading level) usable across the OER ecosystem and by Quizlet/Kahoot-style
  authors who currently have *no* QA.
- **MCP server** (`quiz-bank-mcp`): expose the bank as tools — `search_items(standard, objective)`,
  `validate_item`, `export(qti|gift|moodle)`, `lint_item`, `self_test` — so any agent/LMS can query
  and validate verified items; also lets Claude draft *into* the schema via tool-use.

---

## 8. Open questions

- Independent re-key sampling rate and two-SME threshold — what % / which risk tiers justify the cost?
- Misconception taxonomy: adopt/extend an existing open one (e.g. Eedi/Diagnostic-Questions-derived,
  if licensable) or grow our own controlled vocabulary?
- Exact partial-credit formula and its **defensible** mapping onto QTI 3.0 vs Moodle response
  processing (the plan's own open question — pin before authoring multi-select/matching banks).
- Which CASE-eligible standards frameworks are open enough to *embed* vs reference-only (Common Core
  yes; NGSS reproduction-limited) — per-framework license register.
- Short-text auto-grade boundary: how far (if at all) into synonym/spelling tolerance without an LLM
  grader (explicitly out of scope)?
- Is `aiDrafted` provenance disclosure published to beneficiaries, and does it affect adoption trust?
- First confirmed adoption partner (still TO BE SECURED) — the plan's own #1 risk to "delivered."
