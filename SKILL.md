---
name: code-security-review
description: >
  AI-driven code security review skill.
  Provides a complete methodology for conducting security audits on source code,
  including: security audit prompts, false positive filtering rules (hard exclusions + AI-based filtering),
  severity/confidence scoring guidelines, and customizable scan/filter instructions.
  Supports all programming languages.
---

# Code Security Review Skill

> ** MANDATORY EXECUTION CONTRACT**
>
> This skill defines a **strictly ordered, three-phase process**.
> You MUST complete **all three phases in sequence** before producing final output.
> **You are FORBIDDEN from reporting any findings until Phase 3 (Filter) is complete.**
> Skipping or abbreviating any phase is a critical failure.

---

## Skill Components

Before starting, you MUST read ALL of the following resource files:

| Resource | Path | Read Requirement |
|----------|------|-----------------|
| **Audit Prompt** | `resources/audit-prompt.md` | MUST read before Phase 1 |
| **Hard Exclusion Patterns** | `resources/hard-exclusion-patterns.md` | MUST read before Phase 2 |
| **False Positive Filtering Rules** | `resources/filtering-rules.md` | MUST read before Phase 2 |
| **Customization Guide** | `resources/customization-guide.md` | Read if project-specific rules are needed |

---

## Execution Process (MANDATORY — Do Not Skip Any Phase)

---

###  PHASE 1: Security Audit

**Goal:** Produce a raw list of candidate findings. At this stage, do NOT filter — cast a wide net.

**You MUST:**

1. Read `resources/audit-prompt.md` fully before starting analysis.
2. Execute the three-phase analysis defined in that document:
   - **Phase 1a — Codebase Context Research**: Security frameworks, auth patterns, threat model.
   - **Phase 1b — Comparative Analysis**: Compare code against secure patterns, flag deviations.
   - **Phase 1c — Vulnerability Assessment**: Trace data flows, injection points, privilege boundaries.
3. Produce a **raw findings list** in the following internal format (not shown to user yet):

```
RAW FINDING #N
- Title: <short title>
- File: <path:line>
- Category: <e.g. path_traversal, sql_injection, rce>
- Severity: HIGH / MEDIUM / LOW
- Description: <what the bug is>
- Attack Path: <how an attacker exploits it step by step>
```

**Checkpoint — PHASE 1 COMPLETE when:** You have a complete raw findings list (can be 0 items).

---

###  PHASE 2: Filter Findings (MANDATORY — You MUST run this on EVERY raw finding)

**Goal:** Remove false positives. This phase MUST be applied to every single item from Phase 1.

**You MUST NOT skip this phase even if it seems obvious that a finding is valid.**

#### Step 2a — Hard Exclusion Pass

Read `resources/hard-exclusion-patterns.md`. For **each** raw finding, check:

- Does the title or description match any hard exclusion pattern (DOS, rate limiting, resource management, open redirect, memory safety in non-C/C++, regex injection, SSRF in HTML)?
- Is the file a Markdown (`.md`) file?

If YES → mark finding as **`EXCLUDED (Hard Pattern)`** with the matching rule name.

#### Step 2b — AI Filtering Pass

Read `resources/filtering-rules.md`. For **each non-excluded** finding, produce this mandatory table row:

| # | Title | Hard Excl? | Precedent Hit? | Concrete Attack Path? | Confidence (1-10) | Decision | Reason |
|---|-------|-----------|----------------|-----------------------|-------------------|----------|--------|
| 1 | ... | No | No | Yes | 9 |  KEEP | Directly exploitable, specific code path |
| 2 | ... | No | UUID unguessable (Precedent #2) | Conditional | 5 |  EXCLUDE | UUID ID guessing required |

**Confidence scoring MUST use the 1-10 scale from `filtering-rules.md`:**

| Score | Meaning | Action |
|-------|---------|--------|
| 1-3 | Low confidence, likely false positive | **EXCLUDE** |
| 4-6 | Medium confidence, needs investigation | **EXCLUDE** (unless very concrete) |
| 7-10 | High confidence, likely true vulnerability | **KEEP** |

**Rule: Any finding with Confidence < 7 MUST be excluded.**

**Checkpoint — PHASE 2 COMPLETE when:** Every raw finding has a row in the filter table with a KEEP/EXCLUDE decision.

---

###  PHASE 3: Report (Only KEPT findings)

**Goal:** Output only the findings that survived Phase 2 filtering.

**You MUST output the filter table first** (Phase 2 output), then the detailed findings.

#### Filter Summary Table (always show this)

Show the Phase 2 table so the reader can see the filtering decisions.

#### Detailed Finding Report (only KEPT items)

For each KEPT finding, output:

```
## Vuln N: <Category>: `<file:line>`

* Severity: HIGH / MEDIUM / LOW
* Confidence: <1-10 score> / 10
* Description: <what the vulnerability is>
* Exploit Scenario: <concrete step-by-step attack>
* Recommendation: <specific fix>
```

#### Non-Findings Summary

List EXCLUDED findings briefly:

```
## Excluded Findings (N total)
| # | Title | Reason |
|---|-------|--------|
| 1 | Missing rate limiting | Hard exclusion: rate limiting |
```

---

## Severity Reference

| Severity | Criteria |
|----------|----------|
| **HIGH** | Directly exploitable: RCE, data breach, authentication bypass |
| **MEDIUM** | Requires specific conditions but has significant impact |
| **LOW** | Defense-in-depth issues or lower-impact vulnerabilities |

---

## Anti-Patterns (Things You Must NOT Do)

-  Do NOT report findings before completing Phase 2 filtering
-  Do NOT use 0.0-1.0 confidence scores — the 1-10 scale is required
-  Do NOT skip the filter table in the output
-  Do NOT report a finding with Confidence < 7
-  Do NOT combine Phase 1 and Phase 3 (jumping directly from audit to report)
-  Do NOT mentally filter without showing the filter table
