# Must-mention list

Per-page messages that are expected to land. This file exists to catch **omission**, a failure
the other checks are blind to: a page can be completely accurate, on-brand, and free of banned
words while still failing because it never mentions the one thing that makes Aikido the answer.
The AutoFix page is the canonical example. It said nothing false. It just never said "Libraries."

**How to read an entry:** `CORE` items are the reason the page exists. A missing `CORE` item is a
high-severity finding. `EXPECTED` items should be present but a page can survive without one, so
those are medium. `AVOID` records things a page should specifically NOT feature, which is a
separate axis from being wrong: the capability is real, it's just the wrong context.

**How to maintain:** this is human-owned. Add a page section when a page's job becomes clear, and
delete entries that stop being true. An entry that no longer reflects current positioning is worse
than no entry, because the pipeline will keep flagging correct pages.

**Starter status:** the entries below were derived from the capability ledger's designated
differentiators and the corrections ledger's `expected` fields, not from an independent review of
each page's goals. Treat them as a first pass to correct, not as settled truth. Pages with no
section here get no omission check, which is the right default: silence means "nobody has said
what this page is for yet," not "this page needs nothing."

---

## /code/autofix
Source: capability ledger (Libraries designated core) + corrections `autofix-missing-libraries`

- `CORE` **Aikido Libraries**: patched drop-in version of the exact package in your lockfile, same
  APIs, vulnerability removed, no code rewrites. This is the designated core differentiator for CVE
  autofix. The page must contain the word "Libraries."
- `CORE` **The three-part fix story**, all three parts present: CVE autofix (fast, never breaks the
  build via Libraries, agentic, custom instructions); SAST (matches your code style, guided);
  container (reads your Dockerfile, offers multiple options, hardened images).
- `EXPECTED` **You review and merge, Aikido never commits directly** (read-only access). This is
  both a trust signal and a boundary that keeps the page honest.
- `EXPECTED` **AutoFix works in-IDE and in-PR**, not only in the dashboard.
- `AVOID` Anything implying Aikido adds or installs a package for you as a fix. Confirmed not a
  capability (2026-08-10).
- `AVOID` "data-backed fixes" or any "backed by data" phrasing with no named data or mechanism.

## /comparison/aikido-vs-coderabbit
Source: corrections `coderabbit-*` (CEO, 2026-08-13)

- `CORE` **Platform vs point tool**: the argument that Aikido covers whole categories CodeRabbit
  does not, rather than that Aikido is CodeRabbit with extras.
- `CORE` **A tagline that does not concede the category frame.** Leading with a variant of "AI code
  review that goes beyond code quality" positions Aikido as CodeRabbit-plus and is a high-severity
  positioning finding. The replacement should center developer experience. Owner: Madeline.
- `EXPECTED` **Pricing comparison** present, after the tagline and the platform argument.
- `AVOID` **Hardened libraries.** Real capability, too niche for this audience, wrong page
  (confirmed 2026-08-14). Valid on other pages.

## /protect/device-protection
Source: capability ledger

- `CORE` **macOS and Windows both supported.** Saying only Windows understates current support,
  which is an accuracy finding in the understating direction, and those are easy to miss because
  nothing on the page is technically false.
- `EXPECTED` **Blocks malicious packages before install**, across package registries, IDE
  extensions, and browser extensions.
- `EXPECTED` **MDM deployment** (Jamf, Fleet, Iru) or manual install.
- `AVOID` Any specific Linux availability date. Docs say "coming soon" with no committed date, and
  the page's "early Q3 2026" is overdue.

## /cloud/dspm
Source: capability ledger

- `CORE` **Code-first, never accesses your actual data or data stores.** This is the whole
  positioning and the reason the product is trustworthy to a security buyer.
- `CORE` **The CSPM boundary stated honestly**: DSPM does not see data with no code reference, such
  as an orphaned bucket. The page currently says this correctly. A rewrite that blurs it is a
  regression, so check for erosion, not just absence.
- `EXPECTED` **Every finding carries exact file and line plus an AutoFix PR.**

## /cloud/cloud-posture-management-cspm
Source: capability ledger (freshness flag)

- `CORE` **DSPM appears in the platform coverage list.** DSPM shipped 2026-08-04 and this page
  predates it, so the coverage list is stale. Stale coverage lists on platform pages are a
  recurring failure worth checking on any page that enumerates products.
- `EXPECTED` **Agentless, read-only cloud connection.**
- `EXPECTED` **Guided fixes and PRs for review, not direct auto-remediation of cloud infra.**
