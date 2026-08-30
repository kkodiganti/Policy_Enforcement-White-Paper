# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a content repository, not a software project — there is no build, lint, or test tooling. It holds a single white paper and its supporting research, authored by Krishna Murthy Kodiganti (Senior Lead Software Engineer, Capital One), written as an independent thought-leadership piece — not an official Capital One publication.

## Status

**First full draft complete.** `Policy-Enforcement-White-Paper.md` has real prose in every section (Abstract through Conclusion, plus Contact Information and a populated 26-source three-tier `# Sources` list). The Capital One conflict-of-interest question (see prior status note, now resolved) was decided as: omit Capital One entirely from the paper. Section 2.2 (Timing Gaps) was written without a named-breach example as a result (Section 4's Target/T-Mobile vignettes carry that weight instead), the Section 7 OPA-adopter mention excludes Capital One from its named-organization list, and the Section 4 Banking vignette uses Citibank's Oct. 2020 OCC consent order instead — framed around Section 2.6 (Governance & Accountability Gaps) rather than access-control architecture, since that consent order's verified findings are about data-governance ownership gaps, not access control specifically. `pdftotext` (via Homebrew's `poppler`) was installed locally partway through research/drafting because WebFetch could not render several regulator/academic PDFs as text; most of the sources flagged as unverified in the research file's original "Gaps" section were subsequently confirmed by downloading the PDF directly and running it through `pdftotext -layout` — reuse that method for any remaining unverified PDF rather than re-attempting WebFetch on it.

**First editorial pass complete.** The draft has had one editorial/tone pass: removed a bracketed process note that had leaked into the reader-facing title block (title-finalization status belongs only here in CLAUDE.md), cut the near-verbatim repetition of the six-item failure-mode list across the Abstract/Problem Statement/Conclusion (Conclusion now recaps by category name instead of re-deriving each example), fixed a "rather than" construction that had drifted into a repeated rhetorical cadence (was hitting 2–3 uses per sentence in places, notably Section 3.1's three framework parts), and rewrote several templated "flagged honestly" disclaimer sentences that had all converged on the same shape ("X, not Y — stated plainly rather than glossed over") into more varied, plainly-stated prose. Not yet done: a pass for overall length and whether the six-category taxonomy (Section 2) is introduced in the clearest order for a first-time reader. Treat further citation work the same way as before: research a real source or leave a gap explicitly flagged, never invent a statistic or case study to fill one.

**Visual assets complete.** All 9 planned SVG charts/diagrams are built, validated, and embedded in the paper — see "Visual assets" below for the inventory and the `sips` rendering techniques learned while building them.

## Scope

This paper's subject is **technical/architectural policy enforcement inside an enterprise**: policy-as-code, Policy Decision Point / Policy Enforcement Point (PDP/PEP) architecture, RBAC/ABAC/ReBAC authorization models, and centralized policy engines, contrasted with fragmented, per-service, hardcoded enforcement. It is explicitly *not* a broad people/process/compliance-training paper — HR conduct policy, legal/regulatory policy text itself, and audit-only GRC tooling are out of scope except where they motivate a technical control. Enforcement *timing* (preventive vs. detective vs. corrective) and enforcement *placement* (gateway, service mesh, application code, database, human review) are treated as first-class architectural variables, not implementation detail.

## Files

- `Policy-Enforcement-White-Paper.md` — the white paper itself. Working title: *Policy Enforcement in the Enterprise: From Fragmented Controls to a Unified Enforcement Framework*. First full draft complete, one editorial/tone pass done (see Status below). Thesis: most policy enforcement failures are not rule-design failures or training failures but infrastructure failures — policy logic is duplicated and drifts across services because there is no single enforcement point, no consistent timing model, and no named ownership connecting the written policy to the code that enforces it.
- `Policy-Enforcement-White-Paper-Research.md` — the sourcing backbone: compiled links and figures, organized by taxonomy category (mirrors the white paper's Section 2 structure, the same way the Data-Quality paper's research file mirrors its taxonomy rather than organizing by industry).
- `assets/` — 9 hand-authored SVG charts/diagrams, all embedded in the white paper and validated via `sips` (see inventory below), plus `assets/export-png/` holding the corresponding rendered PNGs per Chart conventions below.
- `Policy-Enforcement-White-Paper.pdf` — the SSRN-ready PDF export of the white paper. A build artifact of the markdown, not hand-edited — regenerate via the recipe in "PDF export" below any time `Policy-Enforcement-White-Paper.md` or the `assets/*.svg` files change.
- `pdf-template.html` — the reusable HTML/CSS shell the PDF export wraps the paper's content in (Letter page size, Georgia serif, 1in margins, centered byline styling). Edit this if the PDF's typography/layout needs to change.
- `SSRN-SUBMISSION.md` — prep sheet for the SSRN submission form: keywords, verified SSRN classifications, and draft Declaration of Interest / Funder / Ethics Approval statements. Not part of the paper itself. The Declaration of Interest has two undecided variants (name Capital One as employer, or keep it anonymized to match the byline) — resolve before actually submitting.

## PDF export

The PDF is built by converting the markdown to an HTML fragment with `pandoc`, merging it into `pdf-template.html`, fixing image paths to absolute (pandoc/browser-relative paths don't resolve once the merged file moves), then printing to PDF with headless Chrome (no LaTeX engine is installed on this machine, so pandoc's default PDF path won't work). Full recipe, run from the repo root:

```bash
mkdir -p .build
pandoc Policy-Enforcement-White-Paper.md -f markdown -t html --wrap=none -o .build/body.html
python3 -c "
import re
tpl = open('pdf-template.html', encoding='utf-8').read()
body = open('.build/body.html', encoding='utf-8').read()
body = body.replace('src=\"assets/', 'src=\"file://$(pwd)/assets/')
open('.build/paper.html', 'w', encoding='utf-8').write(tpl.replace('PAPER_BODY_PLACEHOLDER', body))
"
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-sandbox \
  --print-to-pdf="$(pwd)/Policy-Enforcement-White-Paper.pdf" \
  --print-to-pdf-no-header --no-pdf-header-footer --virtual-time-budget=10000 \
  "file://$(pwd)/.build/paper.html"
```

Then validate before trusting it: `pdfinfo Policy-Enforcement-White-Paper.pdf` for page count/size, and `pdftoppm -png -r 100 Policy-Enforcement-White-Paper.pdf .build/preview/page` + reading a few preview PNGs (title page, a page with a chart, the Sources page) — don't just trust that the command exited 0, since a broken image path fails silently as a blank/broken-image box in the PDF, not a shell error. `.build/` is gitignored; only the final PDF and `pdf-template.html` are tracked.

**Two CSS gotchas already hit and fixed, worth knowing before touching `pdf-template.html`:** (1) don't use a broad selector like `p > em:only-child` to style a standalone-italic paragraph (e.g. the byline) — CSS `:only-child` ignores text-node siblings, so it also matches ordinary inline `*emphasis*` words in the middle of a sentence and centers them incorrectly; use a precise structural selector instead (this file uses `h1:first-of-type + p` and `#contact-information + p`, targeting the exact two byline paragraphs by position). (2) Image `src="assets/..."` paths in the pandoc output are relative to wherever the final merged HTML file sits, not to the repo root — since the merge happens in `.build/`, they must be rewritten to absolute `file://` paths or the images fail silently.

## Visual assets

All 9 are hand-authored SVG (not generated from a script or Mermaid — a deliberate choice, matching this repo's convention over the ease of diagram-as-code), embedded in `Policy-Enforcement-White-Paper.md` via plain `![alt](assets/...)` links, and validated by rendering to `assets/export-png/*.png` with `sips` and visually checking the PNG. If a figure's underlying numbers or argument change, edit the SVG directly and re-validate — don't regenerate from scratch.

| File | Used in | Content |
|---|---|---|
| `chart-taxonomy-overview.svg` | Section 2 intro | The six failure-mode categories, each tagged with which Section 3 framework part addresses it |
| `chart-fragmentation-patterns.svg` | Section 2.1 / 3.2 | The three Barabanov & Makrushin (2020) patterns: decentralized, centralized-single-PDP, centralized-embedded-PDP |
| `chart-credential-breach-stats.svg` | Section 2.4 | Verizon DBIR: 28% / 39% credential figures |
| `chart-entitlement-sprawl.svg` | Section 2.4 | Veza entitlement/dormant/orphaned-account figures — dashed border + "VENDOR-REPORTED, ILLUSTRATIVE" badge, deliberately distinct styling from the primary-sourced charts |
| `chart-gao-recommendations.svg` | Section 2.6 | GAO's 1,610 issued vs. 567 still-open recommendations, as a segmented progress bar |
| `chart-framework-overview.svg` | Section 3.1 | The three framework parts + coverage as a cross-cutting discipline, at a glance |
| `chart-pdp-pep-architecture.svg` | Section 3.2 | XACML PDP/PEP/PAP/PIP reference architecture |
| `chart-rbac-abac-rebac.svg` | Section 3.2 | RBAC vs. ABAC vs. ReBAC comparison |
| `chart-zanzibar-scale.svg` | Section 3.3 | Zanzibar's production figures (trillions of ACLs, sub-10ms p95, 99.999% availability) |

**Two `sips` rendering techniques learned building these, beyond the `tspan`/`dy` caveat already in Chart conventions below:**
- Arrowheads must be manually-drawn `<polygon>` triangles at each line's destination, not `<marker>`/`marker-end` — `sips` doesn't reliably rasterize SVG markers, so a diagram can look correct in the raw SVG and have invisible arrows in the validated PNG. First diagram draft (`chart-pdp-pep-architecture.svg`) hit this; all 9 files use the polygon approach.
- Rounded-end segmented/progress bars (e.g. `chart-gao-recommendations.svg`) need a `<clipPath>` with a single rounded `<rect>`, then flat-edged colored segments drawn inside a `<g clip-path="...">` — two separately-rounded overlapping rects leave a visible notch where they meet. `clipPath` itself renders correctly in `sips` (unlike `marker`), confirmed working.

## Document conventions

- **Required section order** (mirrors the other two papers in this author's series): Title → Abstract → 1. Introduction (Background, Problem Statement) → 2. A Taxonomy of [topic] → 3. The Proposed Framework/Solution → 4. Cross-Industry Evidence → 5. Conclusion → Contact Information → Sources.
- **Section 2 is taxonomy-first; Section 4 is supporting evidence, not the spine.** Section 2 builds a taxonomy of policy-enforcement failure modes (see taxonomy below); Section 4 shows that taxonomy holding across industries as evidence. Don't let Section 4 become the main organizing structure of the argument.
- **Numbered citations must stay in sync.** The `# Sources` section will be a numbered list split into three tiers, in this order: Primary & Regulatory/Standards, Academic & Peer-Reviewed Literature, Industry Research & Vendor Sources. When adding or removing a source, update both this file and the research file, re-verify numbering/links match, and place new sources in the correct tier.
- **Sourcing quality: aim for primary and peer-reviewed where possible.** Prefer, in order: standards bodies (NIST SP 800-162 ABAC, NIST SP 800-207 Zero Trust, OASIS XACML, IETF), peer-reviewed literature, named public incident/breach post-mortems with primary reporting, and only then vendor-blog restatements of the same figures. Vendor sources are fine as a last resort but must be flagged in the Sources section's closing note, not presented as equivalent-weight evidence.
- **Byline is name + professional-society credentials only — no employer, no disclaimer.** As of 2026-08-30, the author deliberately removed the "Senior Lead Software Engineer, Capital One" line and the Capital One disclaimer from both the title block and Contact Information section, so the published paper doesn't name an employer at all. This was a considered reversal of the paper's earlier convention (an employer-independence disclaimer only makes sense if an employer is actually named), most likely driven by preparing the paper for an SSRN submission where "Independent Researcher" is the intended affiliation. The byline is now just: *Krishna Murthy Kodiganti, Fellow IETE, Fellow IAENG, Fellow SCRS, Senior Member IEEE, Member ACM* (ordered Fellow → Senior Member → Member), in the title block and the Contact Information section. Don't reintroduce the Capital One line or disclaimer without the author explicitly asking for it back. If a new credential is earned or a grade changes, update both places.
- **No embedded images.** Real chart/diagram assets live in `assets/` as standalone SVG files, linked via plain `![alt text](assets/...)`. Never inline base64 image data in the markdown.
- **Tone: no "agentic"/AI-essay register.** No meta-referential framing ("this paper argues/shows"), no "not X, it's Y" rhetorical contrast constructions, no repeated-word rhetorical cadences, no heavy em-dash-aside overuse. State claims directly (declarative, third-person); prefer commas/colons/separate sentences to stacked em-dashes.
- **Governing terminology**, to be defined precisely on first use and not blurred together: *policy enforcement* (the umbrella term — actually applying a rule at runtime), *authorization* (the decision of whether a specific request is allowed), *PDP/PEP/PAP/PIP* (Policy Decision Point, Policy Enforcement Point, Policy Administration Point, Policy Information Point — standard XACML-derived architecture terms), *RBAC/ABAC/ReBAC* (role-, attribute-, and relationship-based access control), *preventive / detective / corrective control* (when in the timeline a control acts relative to the action it governs). Don't introduce a synonym for any of these without adding it to the same definition sentence.

## Working taxonomy for Section 2 (subject to revision once research lands)

1. **Fragmentation & Duplication** — policy logic reimplemented per service/team instead of enforced from one place; drift between the "same" rule as implemented in different systems.
2. **Timing Gaps** — enforcement that happens too late (detective/audit-only, after the action already occurred) when it needed to be preventive, or blocks legitimate action because it's evaluated too early/without full context.
3. **Semantic Translation Gap** — the written/legal/business policy and the code that supposedly enforces it diverge, because translating natural-language policy into rules is manual and unverified.
4. **Contextual & Identity Staleness** — enforcement decisions made on stale inputs: entitlement creep, orphaned accounts, over-provisioned roles, attributes (risk score, employment status, device posture) not refreshed at decision time.
5. **Coverage Gaps** — shadow IT, legacy systems, and break-glass paths that bypass the central policy engine entirely, so the taxonomy's other failure modes are moot for the traffic that never reaches it.
6. **Governance & Accountability Gaps** — no named owner connecting a written policy to its enforced implementation; no versioned audit trail of who approved or changed a rule.

## Chart conventions (for when `assets/` is created)

Categorical palette blue `#2a78d6` / orange `#eb6834` / aqua `#1baf7a` in fixed order, chart surface `#fcfcfb`, primary ink `#0b0b0b`, secondary ink `#52514e`, muted `#898781`, gridline `#e1e0d9`, light-mode only (static document asset, not an interactive artifact). Multi-line SVG text must use **separate `<text>` elements with explicit `y` per line**, not `<tspan dy="...">` — some lightweight SVG rasterizers (e.g. macOS `sips`) don't honor `tspan`/`dy` line breaks and silently overlap lines. **Arrowheads must be manually-drawn `<polygon>` triangles at the line's destination point, not `<marker>`/`marker-end`** — `sips` doesn't reliably render SVG markers either, and a diagram with invisible arrows looks fine in the raw SVG but broken in the validated PNG; this is the same class of issue as the `tspan`/`dy` caveat, so treat `sips` as supporting only basic shapes and text, not the fuller SVG feature set. Validate with `sips -s format png in.svg --out out.png` and read back the PNG (preserves true `viewBox` pixel dimensions); don't trust `qlmanage -t` for overflow checks, it renders into a fixed square thumbnail and can crop content that's actually within the `viewBox`.

## Open TODOs / gaps

- **This is a first draft, not a polished one.** No editorial pass yet for overall length, section-to-section flow, or whether the six-category taxonomy (Section 2) is introduced in the clearest order.
- **Section 4 industries: Banking (Citibank/OCC), Healthcare (Yakima Valley/HHS OCR), Government (GAO), Retail/Telecom (Target/T-Mobile).** Cloud/SaaS was evaluated and dropped as redundant with what would otherwise have been the Banking vignette (Capital One, since excluded — see Status above).
- **A few substantive research gaps still have no adequate source**, flagged in the prose itself rather than filled: shadow IT prevalence has no primary Gartner/Everest figure (Section 2.5), stale-contextual-attribute-at-decision-time has no dedicated source distinct from entitlement creep (Section 2.4), and no independent study compares breach cost/incident rate for centralized vs. fragmented enforcement (Section 3.3 falls back to IBM/Ponemon, flagged as vendor-tier).
- **A few sources are still verified only via secondary reporting**, not the primary document directly: the Yakima Valley HHS OCR settlement (source 11) and ISO/IEC 27001:2022 Clause 5.2 (source 15, paywalled standard). GAO-17-549 (source 8) is cited only for its general finding, not the specific weakness counts, which remain unconfirmed against the primary PDF (GAO's site blocks non-browser fetches; not yet worked around).
- **Title not finalized.** "Policy Enforcement in the Enterprise: From Fragmented Controls to a Unified Enforcement Framework" is a working title.
