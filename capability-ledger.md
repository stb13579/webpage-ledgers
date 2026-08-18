# Aikido capability ledger

Human-owned source of truth for what each product area **does** and **does not** do. The health-check pipeline reads this on every run to catch fabricated or overstated capability claims — the failure mode where a marketing page describes something the product doesn't actually do, which docs-comparison alone misses because undocumented is not the same as false.

**How to maintain:** when a product owner confirms a capability (or confirms the product does NOT do something), add it here. This file, not the model, is authoritative. Keep entries concrete and checkable. Owners listed per section are the people to route "confirm this claim" questions to.

**Status legend:** `SHIPPED` (live today) · `BETA` · `COMING SOON` (announced, not yet available — do not describe as present-tense capability) · `NOT DONE` (explicitly not a capability — flag any page that claims it).

---

## AutoFix / remediation
Owner: _TBD (product) — Leadership feedback flagged this page_

**Does:**
- `SHIPPED` One-click AI-generated fix PRs for SAST, IaC, SCA, and container issues.
- `SHIPPED` Preview proposed fix before generating the PR; user reviews and merges (Aikido never commits directly — read-only access).
- `SHIPPED` AutoFix for container images, including an indication of how many issues a fix resolves and whether it would introduce new ones.
- `SHIPPED` AutoFix for pentest and code-audit findings (review-ready patch).
- `SHIPPED` AutoFix available in-IDE and in-PR (inline comments, severity/type gating).
- `SHIPPED` "Ask Aikido" for validating/adapting a generated fix without leaving the platform.
- `SHIPPED` Aikido Libraries: patched drop-in version of the exact package in your lockfile — same APIs, vulnerability removed, no breaking changes / no code rewrites. **This is the designated core differentiator for CVE autofix and must appear on the autofix page.**

**Does NOT do (flag if claimed):**
- Does **not** automatically add a new package to your project as a "fix." The line "Seamlessly adding a package to your project? We got you covered" was confirmed by the product owner as describing something Aikido does not do (2026-08-10). Any variant of "we add/install packages for you" should be flagged.

**Watch phrases (empty/unsupported jargon — flag for rewrite, not necessarily false):**
- "data-backed fixes" — confirmed nonsense by product owner; no clear mechanism referent. Flag any claim of the form "backed by data" that doesn't specify what data or how.

---

## SCA (open-source dependency scanning)
Owner: _TBD (SAST/SCA service owner cited in messaging)_

**Does:**
- `SHIPPED` Scans open-source dependencies for CVEs, malware, license issues, EOL.
- `SHIPPED` Reachability analysis: dependency-level + function-level + contextual/runtime, to determine whether a vulnerable dependency is actually reachable/exploitable.
- `SHIPPED` Exploitability analysis (agent-based): assesses each CVE against how the package is used in the repo, with written reasoning logged in run history.
- `SHIPPED` One-click AutoFix and Aikido Libraries patched packages.
- `SHIPPED` SBOM export (CycloneDX, SPDX, CSV); import external SBOMs.
- `SHIPPED` Aikido Intel pre-CVE detection cross-referenced with NVD, GitHub Advisory, and 10+ feeds.

**Claims to handle carefully (not false, but unsupported as stated):**
- "100% reduction in false positives" / "cuts false-positives entirely" — the docs describe the same mechanism in **conditional, continuously re-evaluated** terms (findings downgraded/suppressed *only when provably unreachable*; some kept high when usage is unclear). An unqualified absolute claim of zero false positives is not corroborated by any doc page. Flag for review — needs either a named internal benchmark to back it or softening.

---

## Device Protection
Owner: _TBD (Aikido Intel / research team)_

**Does:**
- `SHIPPED` Blocks known-malicious packages/extensions before install (npm, PyPI, Maven, NuGet, Go, Ruby, Rust, PHP, and more; VS Code / OpenVSX / Visual Studio / JetBrains extensions; Chrome & Firefox extensions; AI tool ecosystems).
- `SHIPPED` Deploys via MDM (Jamf, Fleet, Iru) or manual install.
- `SHIPPED` Minimum package age (default 48h), allow/block lists, group-based policies, request-and-approval workflows, audit trail.
- `SHIPPED` Typosquatting detection on first request, no config needed.
- `SHIPPED` Available on **macOS and Windows** (both — per current docs).
- `COMING SOON` Linux support (docs say "coming soon," no committed date as of the last docs update).
- `COMING SOON` Shadow AI detection (early access).

**Does NOT do / flag if claimed:**
- Marketing currently says "available on Windows" and omits macOS — this **understates** current support. The correct present-tense claim is macOS + Windows.
- Do **not** state a specific Linux availability date. The "early Q3 2026" date on the marketing page is now overdue and no longer matches the docs' vaguer "coming soon."

---

## DSPM (data security posture management)
Owner: _TBD_

**Does:**
- `SHIPPED` (launched 2026-08-04) Code-first DSPM: analyzes code, schemas, ORM models, API definitions, IaC, CI/CD — **never accesses your actual data / data stores**.
- `SHIPPED` Traces sensitive data flow from entry to every destination; finds sensitive data in logs, API responses, third-party shares, data sent to AI, missing encryption, prod data in test envs, incomplete deletion, exposed secrets.
- `SHIPPED` Every finding includes exact file/line and an AutoFix PR.

**Boundary (accurately stated on current page, keep it that way):**
- Does **not** see data with no code reference (e.g. an orphaned bucket) — that's CSPM's job, not DSPM's. The page correctly says so; don't let a rewrite blur this.

---

## CSPM (cloud security posture management)
Owner: _TBD_

**Does:**
- `SHIPPED` Agentless, read-only API connection to AWS / Azure / GCP (+ DigitalOcean and others).
- `SHIPPED` Detects misconfigurations, over-permissive IAM, exposures; context-aware severity (prod > staging).
- `SHIPPED` Cloud Search (query cloud like a database) + real-time alerts on asset changes.
- `SHIPPED` Compliance mapping (SOC 2 / ISO 27001), syncs to Vanta, Drata.
- `SHIPPED` Container image, VM, IaC scanning; outdated-runtime detection; AI AutoFix for cloud/IaC/container issues.
- `SHIPPED` Custom CSPM rules.

**Does NOT do / flag if claimed:**
- Does **not** auto-remediate cloud infra directly — it produces guided fixes / Terraform / one-click PRs for review. (Page states this correctly.)

**Freshness flag:**
- The "Full Coverage in One Platform" list on this page **omits DSPM** (shipped 2026-08-04). Page predates that launch and needs updating.

---

## PR Review (AI PR Review / Deep PR Review)
Owner: _TBD_

**Does:**
- `SHIPPED` Reviews every PR with full codebase context (multi-repo, static results, PR comments); flags bugs, security issues, side effects; can block risky PRs.

**Naming note:** appears as both "AI PR Review" and "Deep PR Review" across nav/pages (the CodeRabbit comparison uses "Deep PR Review"). Confirm the canonical name.

**Open question (not a defect, needs owner input):**
- No dedicated help.aikido.dev page found under this feature name. Confirm whether it's documented elsewhere or is a genuine docs gap.

---

# Competitor capabilities (for comparison pages)

**Why this section exists:** comparison pages make claims about what *competitors* do and don't do. The framework previously only knew Aikido's own capabilities, so a table row asserting "CodeRabbit doesn't do X" had no ground truth to check against — and "not in our notes" silently became "they don't do it." That's how a false comparative claim ships, and comparative claims carry legal/reputational risk. Every "competitor does NOT do X" mark on a live comparison page must trace to an entry here, or be flagged as unverified before publishing.

**How to maintain:** record competitor capabilities only from the competitor's own current docs/site (dated), not memory or inference. Competitors ship fast — mark a `checked` date and re-verify. When unsure, the entry is `UNVERIFIED`, and any table row depending on it must be flagged, not published.

## CodeRabbit
Comparison page: /comparison/aikido-vs-coderabbit · Last verified: _TBD (needs a pass against coderabbit.ai)_

**Executive feedback flat wins (Aikido does, CodeRabbit does NOT) — VERIFY each against CodeRabbit's site before publishing:**
- Malware protection — _UNVERIFIED_
- CVE detection — _UNVERIFIED_
- Container image scanning — _UNVERIFIED_
- Cloud misconfiguration / CSPM — _UNVERIFIED_
- Pentesting — _UNVERIFIED_
- CVE exploitability analysis — _UNVERIFIED_
- Local security scanning without sharing code — _UNVERIFIED_

(Source: Executive leadership Slack, 2026-08-13. These are the capabilities executive leadership wants the comparison table built around. They are asserted, not yet verified against CodeRabbit's documentation — do not publish a "CodeRabbit doesn't do this" mark until each is confirmed.)

**Scope guidance from leadership:**
- Do NOT feature "hardened libraries" on the CodeRabbit comparison (too niche for that audience — interpretation unconfirmed, see corrections-ledger `coderabbit-hardened-libraries-scope`).

---

# Ledger scope note

This ledger now covers four claim types the pipeline should check:
1. **Aikido does** — capability claims (catch understatement/inaccuracy).
2. **Aikido does NOT do** — catch fabricated own-capabilities (e.g. AutoFix "we got you covered").
3. **Competitor does / does NOT** — catch false comparative claims on comparison pages (this section).
4. **Watch phrases / empty jargon** — per-product, catch meaningless copy (e.g. "data-backed fixes").
