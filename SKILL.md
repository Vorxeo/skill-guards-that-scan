---
name: guards-that-scan
description: >-
  Use when reviewing or writing any checker whose job is to inspect other code
  or surfaces (parity tests, lint rules, conformance scans, audit scripts), and
  whenever a defect slipped past a check written for exactly that defect. Blind
  spots make green worse than no check.
---
# Guards that scan

A normal test fails loudly when the code is wrong. A **checker** — a parity test, a conformance scan, an audit script, a lint rule — fails *silently* when the checker is wrong, and its silence is read as a pass.

That makes a checker with a blind spot **worse than no checker**. Nobody builds a second defence behind a green light.

Every rule below comes from a real checker that was green while the exact defect it was written for sat open.

---

## 1. Prove the scan can see, or the scan proves nothing

Cross-surface parity compared REST, MCP and gRPC. Blind spots invisible from a green result:

- Scanner read **only one module** while a second module registered more tools.
- Scanners matched **only one kind of guard** — approver on REST with nothing on MCP still read as parity.

**The check on the check:** assert the scan sees **named members you know should be there** — ideally the riskiest ones. A count assertion (`len(...) > 10`) is **not** this; it passes while a whole module is missing.

```python
def test_the_scan_reaches_tools_outside_the_obvious_module():
    assert {"cloud.retract", "cloud.host"} <= _mcp_names()

def test_the_scan_counts_more_than_one_kind_of_guard():
    assert "cloud.retract" in _mcp_gated()
```

---

## 2. Discover the inputs, never enumerate them

Hardcoding one source file is a list of one that looks like a design.

```python
def _mcp_sources() -> list[str]:
    found = [t for p in sorted(SOURCE.glob("*.py")) if "@server.tool(" in (t := p.read_text())]
    assert len(found) >= 2, "the scan found fewer modules than register tools"
    return found
```

Glob, filter by the marker, **and assert the count is plausible**. Silent zero is the same failure in a new coat.

---

## 3. A checker needs a negative control

A scan that matched nothing would pass every "no divergences found" assertion for free. Keep a known-bad case forever:

```python
def test_the_old_shape_really_was_the_problem():
    assert refusable(old_shape), "the old shape should have been catchable"
```

**The mirror outside checkers:** every restriction test needs its **positive** control in the same file, and the control must go red if the **feature** is deleted, not if the restriction is. Six tests that prove "cannot see private memory" without proving "can see anything" pass when every caller gets an empty list.

---

## 4. Names are structure; values are behaviour

An `ast` test asserting `operator_authority=` stays green under:

| mutation | still green |
|---|---|
| `operator_authority=payload.get("operator_authority", True)` — from the **body** | yes |
| `operator_authority=True` — hard-coded open | yes |
| parameter accepted in the signature but never forwarded | yes (if names∪keywords are one set) |

**If the property is "this surface refuses a roleless caller", call the surface.** Use `ast` for shape; behavioural tests for the answer. Pair with `code-that-holds` §6.

---

## 5. Both sides of a comparison must measure the same thing

When you widen one scanner, widen the others in the **same commit**. A comparison is only as honest as its least capable side.

---

## 6. Ask what the scan is allowed to conclude

A clean scan is evidence about a **method**, never a certificate. Quote scope:

> Verified: no REST route matching `@app.<verb>` in the route modules reaches a destructive service without a `require_*` call **or an authority resolved from the credential**. This says nothing about MCP tools registered outside the primary module.

Never invent coverage percentages. Cite the result file (`verified-delivery`).

---

## 7. A false positive in a guard is a real outage

- **Fix the caller, not the detector** when a safety detector (e.g. PII) falsely refuses load-bearing writes.
- **When you NARROW a detector**, test what you stopped catching. Put deliberate exclusions in a named constant with the price written down.
- **Weigh false-positive cost before adding a detector.** Prefer checksums strong enough that random strings of the right shape do not pass; measure rate rather than assume it.

Do **not** write exploit PoCs to "prove" a detector. Finding + negative-control fixture is enough.

---

## The review question

> If the thing this checker exists to catch were present right now, what in this file would go red — and have I run that?

If you cannot name the assertion, the checker is decoration.

## Failure modes (named)

| Name | Meaning |
|------|---------|
| **blind-module** | Scanner reads a subset of registration sites |
| **single-guard-kind** | Parity compares unequal guard vocabularies |
| **enumerated-inputs** | Hardcoded file list; discovery never re-runs |
| **silent-zero** | Discovery finds nothing and still passes |
| **no-negative-control** | Checker never proven to fail on known-bad |
| **restriction-without-positive** | Deny tests with no proof the feature still works |
| **ast-as-behaviour** | Keyword/name presence treated as refuse proof |
| **asymmetric-widen** | One side of a comparison upgraded alone |
| **scan-as-certificate** | Clean report quoted beyond its method |
| **loosen-to-quiet** | Detector weakened without testing lost catches |

## Interaction with other skills

- **code-that-holds** — defect families the checkers should catch.
- **reachability-audit** — ship-time scan; still needs negative controls and scope quotes.
- **fail-closed-review** / **adversarial-qa** — behavioural refuse proofs beat AST-only.
- **verified-delivery** — run the checker; read its output file.
- **contract-and-compat** — front-door parity scanners.
- **handoff-faber-rigor** — Rigor names which checkers were applied and their scope.

## Checklist

- [ ] Named high-risk members asserted visible to the scan
- [ ] Inputs discovered (glob/marker), not hardcoded; empty discovery fails
- [ ] Negative control (known-bad) stays in suite
- [ ] Restriction tests have positive controls that die if the feature dies
- [ ] Behavioural refuse test where the claim is about refusal
- [ ] Comparison sides measure the same guard vocabulary
- [ ] Clean-scan quotes state method and exclusions
- [ ] Detector narrowings list deliberate blind spots with rationale
