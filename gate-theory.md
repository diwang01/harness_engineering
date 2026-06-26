# Context: The 8-Gate Framework 

A structured quality wall for agentic software development. Each gate maps to an industry-standard
discipline, so the framework speaks both to AI-agent workflows and to classical engineering practice.

---

## The Core Idea

Traditional CI pipelines check *build correctness*. The 8-gate framework extends that to cover the
full lifecycle of an agentic change: from intent capture through human sign-off to self-improvement.
The key insight is that **agent failures cluster at boundaries** — between human intent and machine
interpretation (G1–G3), between generation and execution (G4–G5), between execution and correctness
(G6), and between correctness and trust (G7–G8).

Gates run in order. Any failure returns the work to the developer — not to the next gate.

---

## Gate Map

| Gate | Name | Industry Equivalent |
|------|------|---------------------|
| G1 | Intent Capture | Agent Orchestration & Prompt Routing |
| G2 | Ambiguity Resolution | Agent Orchestration & Prompt Routing |
| G3 | Plan Sign-off | Agent Orchestration & Prompt Routing |
| G4 | Constraints & Syntax | Static Analysis & Guardrails |
| G5 | Execution Sandbox | Secure Runtimes / Containerised Execution |
| G6 | Auto-Verification | LLM-as-a-Judge / Automated Eval Pipelines |
| G7 | Human-in-the-Loop | HITL Review / PR Checkpoints |
| G8 | CI/CD Evolution | LLMOps & Continuous Eval |

---

## Phase 1 — Intent & Planning (G1–G3)

**Industry mapping: Agent Orchestration & Prompt Routing**

Before any code is written, the agent must prove it understood the request. This phase exists
because the most expensive bugs are misunderstood requirements — no amount of test coverage fixes
code that solves the wrong problem.

- **G1 — Intent Capture:** The agent restates the requirement in its own words and produces an
  explicit acceptance criteria checklist. This is the contract the rest of the gates check against.

- **G2 — Ambiguity Resolution:** Any unclear or contradictory requirement is surfaced and resolved
  with the human *before* coding starts. A pre-mortem is run: what assumptions is the plan making,
  and what could go wrong?

- **G3 — Plan Sign-off:** The implementation plan maps each step to an acceptance criterion. Files
  to change, new classes, test strategy, and market/scope boundaries are all identified. A human
  confirms the plan before any file is touched.

> **Why this matters for agents:** LLMs are fluent — they will generate plausible-looking code for
> the wrong spec without hesitation. G1–G3 force the agent to externalise its interpretation so a
> human can catch the mismatch early.

---

## Phase 2 — Static Guardrails (G4)

**Industry mapping: Static Analysis & Guardrails**

The generated code is checked before it runs. This phase catches the class of bugs that
static analysis has always caught — syntax errors, forbidden dependencies, policy violations —
plus the new class of bugs that agents introduce: hallucinated APIs, leaked internal types,
ignored code-style rules.

- **G4a — Static syntax:** Checkstyle / linter passes.
- **G4b — Dependency scan:** No unapproved or unmanaged dependencies introduced.
- **G4c — Compilation:** Zero compiler errors, including Lombok-generated symbols.
- **G4d — Policy compliance:** Architecture rules enforced (layer boundaries, naming conventions,
  no magic literals, market parity).

> **Why agents need this:** Code generation is statistically good at syntax but poor at
> respecting *project-specific* constraints. G4 encodes those constraints as checkable rules
> rather than hoping the agent remembers them from the prompt.

---

## Phase 3 — Execution Sandbox (G5)

**Industry mapping: Secure Runtimes / Containerised Execution**

The code runs in a controlled environment. Tests prove behaviour, coverage proves that the
tests are exercising the right code.

- **G5a — Unit regression:** All existing tests stay green. New behaviour is tested in both
  market implementations where applicable.
- **G5b — New coverage:** New/changed code meets the coverage floor (≥ 80% line coverage).
  Coverage is checked per changed class, not just globally.

> **Why "sandbox":** The parallel to secure runtimes is deliberate — just as a runtime sandbox
> limits what generated code *can do* at execution time, the test suite limits what generated
> code *can break* in the codebase. Both are containment strategies.

---

## Phase 4 — Auto-Verification (G6)

**Industry mapping: LLM-as-a-Judge / Automated Eval Pipelines**

A second, automated pass checks correctness at a higher level than unit tests. Where G5 checks
*that code runs*, G6 checks *that the right thing is returned*.

- **G6a — Interface / API:** Integration tests (MockMvc slices, TestContainers) verify HTTP
  contracts — correct status codes, correct JSON bindings, Spring context integrity.
- **G6b — Contract conformance:** Response shapes match the acceptance criteria from G1 and any
  OpenAPI spec. Error shapes are consistent.

> **The LLM-as-a-Judge parallel:** Automated eval pipelines use a second model to judge the
> first model's output. G6 does the same with integration tests — a separate, higher-level
> assertion layer that can catch what unit tests miss (wiring errors, serialisation mismatches,
> security filter gaps).

---

## Phase 5 — Human-in-the-Loop (G7)

**Industry mapping: HITL Review / PR Checkpoints**

No change ships without a human seeing a plain-language summary and confirming it. This gate
closes the loop between automated verification and human trust.

- **G7a — Artifact integrity:** The working tree is clean — no stray build outputs, no
  accidentally staged secrets or local config files.
- **G7b — Human confirmation:** The agent presents gate results, diff stats, and the acceptance
  criteria checklist from G1. The human explicitly confirms before staging.

> **Why explicit HITL:** Automation builds confidence, but the agent's confirmation bias is
> real — it will tend to report success. G7 forces the agent to surface the summary to a human
> who can override. The gate cannot be auto-passed.

---

## Phase 6 — CI/CD Evolution (G8)

**Industry mapping: LLMOps & Continuous Eval**

The harness improves itself from each run. This is the gate that makes the framework compound
over time rather than staying static.

- **Lessons:** New pitfalls (Lombok quirks, DynamoDB mapping gaps, security filter gaps) are
  logged immediately after the run.
- **Coverage ratchet:** Classes that rise above the 80% floor note their new high-water mark,
  which becomes the floor for future runs.
- **Graduation:** A lesson seen across multiple sessions is proposed for promotion into a
  permanent rule file (`.claude/rules/*.md`), so it becomes a G4d policy check — not just advice.

> **The LLMOps parallel:** Continuous eval means feeding production signal back into the
> model/pipeline. G8 feeds run signal back into the *agent harness itself* — the rules, the
> guardrails, the coverage floors. The agent gets better at this project with every ticket.

---

## Flow Summary

```
Human writes ticket
        ↓
  G1 Intent Capture       ← agent restates + acceptance criteria
  G2 Ambiguity Resolution ← open questions resolved, pre-mortem run
  G3 Plan Sign-off        ← human confirms plan before any file is touched
        ↓
  [DEVELOPING]
        ↓
  G4 Constraints & Syntax ← static analysis, compilation, policy
  G5 Execution Sandbox    ← unit tests + coverage
  G6 Auto-Verification    ← integration tests + contract conformance
  G7 Human-in-the-Loop    ← plain-language summary → human confirms
        ↓
  [STAGED / PR OPEN]
        ↓
  G8 CI/CD Evolution      ← lessons logged, ratchet updated, graduations proposed
```

Any gate fails → return to DEVELOPING. G8 runs regardless of outcome to capture lessons.

---

## Reference

Full gate procedures (exact commands, pass criteria, verifier agent instructions):
`.claude/context/g-gates.md`
