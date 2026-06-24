# Harness Three-Layer Loading Model

This document outlines the architectural breakdown and structural principles of the "Harness Three-Layer Loading Model."

---

## 📘 Layer 1: Resident Layer
* **Characteristics:** Remains present across all message rounds; size constrained to `≤ 8K tokens`.

### 1️⃣ CLAUDE.md + CLAUDE.local.md
* **Core Components:** * Role Definitions
  * Code Style Preferences
  * Process Trigger Rules
  * G1-G8 Gate / Checklist Quick Reference
* **Usage Note:** Self-contained and does not rely on `@import` rules. Simply copy this file over to any new project for immediate use.

### 2️⃣ rules / 7 Atomic Rules
* **The Rules:**
  1. `build` (Forbidden `-am` flags)
  2. `tdd` (Red → Green)
  3. `atdd` (Given-When-Then)
  4. `branch-hygiene` (Merge into `master` branch first)
  5. `code-search` (Plain `grep` is forbidden; use `kbase` instead)
  6. `task-artifacts` (Artifact archiving)
  7. `auto-learn` (Experience crystallization / continuous learning)
* **Philosophical Note:** Every rule is an epitaph written from a past production incident.

---

## ⬇️ Risk Escalation / Dynamic Stage-Triggered Loading

---

## 📒 Layer 2: On-Demand Layer
* **Characteristics:** Content is only read when entering its specific phase, and is immediately released from context after execution.

### 📂 context / 9 Deep Context Documents
* `orchestrator-flow` (Full End-to-End Process Flow)
* `evidence-chain` 
* `pre-mortem-template`
* `adversarial-debate`
* `memory-evolution`
* `tdd-guide`
* `atdd-guide`
* `code-map` (Full Project Landscape / Overview)
* `harness-full-flow`

### 🔬 Academic References
* **Theory:** *Lost in the Middle (TACL 2024)*
* **Principle:** Proves that LLM attention follows a U-shaped distribution, where information placed in the middle of a long prompt significantly attenuates (fades). Loading context dynamically on-demand prevents middle-bloat and context degradation.

---

## ⬇️ Activated via `dispatcher call_agent`

---

## ⚙️ Layer 3: Execution Layer
* **Characteristics:** Only the explicitly activated agent runs. Execution equals context (context is spun up and torn down strictly
    + planner
    + coder 
    + reviewer
    + tester
    + tdd-enforcer
    + atdd-enforcer
    + risk-guardian
    + kbase-librarian


## 🟧 2.1 Resident Entry Layer: `CLAUDE.md` + `CLAUDE.local.md`

Houses roles, code preferences, workflow trigger rules, and G1–G8 gate quick-references. The critical design is that **`CLAUDE.local.md` is self-contained and does not depend on global `@import`**: when a new project is onboarded, you only need to copy a template into it to make it run independently.

* **Solution**: The workflow specifications of each project are isolated from one another, preventing any cross-contamination.
* **Effect**: Reduces the resident context of the main conversation to ≤8K, leaving the valuable window for actual code.

## 🟧 2.2 Atomic Rule Layer: `rules/` (7 items)

Each rule has a single responsibility and can be referenced on demand. Their essence is to avoid stepping on...

| Rule File | Specification / Requirement | Consequence / Reason |
| :--- | :--- | :--- |
| `build.md` | Disable `mvn -am`, compile single module only | `-am` triggers a full parsing that freezes the system |
| `code-search.md` | Ban Grep searching Java structures, ban LSP; force the use of `kbase` | Symbol search false positives, wasting context |

For exmaple, Grep is low-performance and swallows your massive token, you can define your tool for search
```markdown
# Code Search Protocol

- **CRITICAL**: Do not use the `Grep` tool or run bash grep commands across the whole Java directory. This creates false positives and wastes tokens.

- **DO**: Use your built-in `Glob` tool to target file paths or list structures first.

- **DO**: If you need to check class layouts, use the `LSP` tool (Language Server Protocol) features to pinpoint files instead of doing blind text searches.
```

## 🟧 2.3 Role Agent Layer: `agents/`

This is the engine of the entire framework, breaking down an "all-powerful main session" into a pipeline with clearly defined responsibilities:

* **Workflow Scheduling**: `dispatcher` reads `state.json` + `workflow.yaml` to decide the next step—acting as a **traffic cop**, managing routing only, not business logic.
* **Review Synthesis**: `orchestrator` reads the perspectives of three roles written into `phases/*.md`, synthesizes conclusions, and confirms with the user—acting as a **meeting secretary**, managing synthesis only, not scheduling.
* **Three-Role Review**: `requirement-analyst` (Business) / `tech-architect` (Architecture) / `quality-guardian` (Quality)... [Synthesis saves more tokens instead].
* **Workflow Execution**: `plan-generator` → `developer` → `verifier` → `deployer` → `tester`, moving step-by-step from proposal to acceptance.

---

### Categorized by Purpose (Four Classes):

| Category | Role | Why They Cannot Be Merged |
| :--- | :--- | :--- |
| **Scheduling + Synthesis** | `dispatcher` (Routing)<br><br>`orchestrator` (Synthesis) | The `dispatcher` only reads `state.json` to make routing decisions, while the `orchestrator` only reads `phases/*.md` to make synthesis confirmations. Orthogonal responsibilities; merging them would lead to "acting as both referee and player." |
| **Review** | `requirement-analyst`<br>`tech-architect`<br>`quality-guardian` | (Truncated/Implicit: Kept separate for specialized perspectives). |
| **Workflow Execution** | `plan-generator` → `developer` → `verifier` → `deployer` → `tester` | The system prompt and available tools for each node are completely different (e.g., `developer` has Edit/Bash tools, whereas `verifier` only has Read/Bash). Merging them means exposing all tools simultaneously, which equals having no guardrails. |

> 💡 **Key Takeaway**
> What we truly need to watch out for is not having "too many agents," but rather **"high coupling between agents."** As long as inputs and outputs are clean files or JSON formats without requiring session-level negotiation, the sheer quantity of agents is not an issue.



## 🟧 2.4 On-Demand Context Layer: `context/` (10 items)

Detailed full workflows, Pre-Mortem templates, adversarial debate templates, chain-of-evidence specifications, and TDD/ATDD guidelines, as well as the memory evolution mechanism—all are placed in this layer. They are only Read when entering the corresponding phase. As a result, the resident layer remains highly streamlined; deep content acts like ammunition, "only brought up when it's time to fight."

This isn't just based on intuition—LLM attention spans exhibit a U-shaped distribution, where accuracy significantly drops for information located in the middle (Stanford's "Lost in the Middle", TACL 2024[4]). Furthermore, among models claiming to support 32K+ contexts, only half can maintain reliability at that length (NVIDIA RULER[5]).

> **Design Principle**: Context is not an "the bigger, the better" free buffer; it is a scarce resource that requires meticulous management. Each context file should contain only what is strictly necessary for that specific phase.

---

## 🟧 2.5 Execution Support Layer: `skills/` (22 items) + `commands/` (12 items) + `evals/`

* **`skills/` (22 items)**: Encapsulates internal CLIs and R&D toolchains into capabilities that the AI can invoke. The core is the `ubase` suite—a single request like "help me look at the logs for `wrate` over the last half hour" can automatically construct an SLS query, calculate the time window conversion, and aggregate the hit results into an anomaly summary, instead of dumping the raw logs entirely back into the context. 

* **`commands/` (12 items)**: Entry points for slash commands. `/init-harness` for one-click onboarding of new projects, `/harness-audit` to check the health of current configurations, and `/learn` to precipitate lessons learned from pitfalls into rules.


* **Three-Stage Experience Evolution (Rule-Driven via `auto-learn`)**: This is the core mechanism that makes the harness "smarter the more it is used." Taking the `mvn -am` system freeze as an example:
  1. The first time the pitfall is encountered, it is written down as a **lesson** (a single record).
  2. The second time it happens in a different project, it is generalized into a **pattern** ("Mac + system-scope dependency = Ban `-am`").
  3. The third time it is verified, it gets promoted to an **instinct**, which automatically injects the rule into the `build.md` of all new projects.
  
  *Every stage of promotion requires manual user confirmation to prevent erroneous experiences from spreading.*


## 🟧 2.6 Stability Fulcrum: eval Detection + hook Interception

The five layers described above define "how it should be done," but if no one checks "whether it actually got done," all constraints are nothing more than paper talk. **What makes the harness truly stable is not the rules themselves, but the verification mechanism.**

In harness, this "inspection layer" is supported by two mechanisms: 



* **G1–G8 Gate Wall (eval-style hard validation)**: Each gate is a deterministic Python function that checks whether artifacts exist, compilation passes, and unit tests succeed. After the `verifier agent` finishes running, it writes to `phases/verification.json`. If any single gate FAILS, the workflow rolls back to `DEVELOPING`——**this is not a "suggestion", it is a "hard block"**.


| Gate | Inspection Name | Core Check / Intent (Where AI heavily tends to fail) |
| :--- | :--- | :--- |
| **G1** | Static Syntax Check (Linting) | Matching brackets/parentheses, proper indentation, and checking for any low-level syntax errors. |
| **G2** | Dependency & Security Scanning | Verifying whether any unsecure or unauthorized third-party libraries/packages have been introduced. |
| **G3** | Compilation / Build Check | Confirming if commands like `mvn compile` or `npm run build` can execute successfully without breaking. |
| **G4** | Unit Testing (Regression) | Running existing unit tests to ensure the AI did not break previously functioning features (regression prevention). |
| **G5** | New Unit Test Coverage | Checking if the unit tests written by the AI for the newly added feature meet the required coverage threshold (e.g., >80%). |
| **G6** | Interface / Integration Testing (API) | Validating cross-module calls, mock data layer connections, and proper database read/write behaviors. |
| **G7** | Policy & Architecture Compliance | Checking adherence to earlier architectural definitions like `build.md` (e.g., ensuring banned commands weren't snuck in). |
| **G8** | Artifact Integrity Verification | Final check to guarantee that the delivered directory structures, generated binaries, and config files are complete. |

G1-8 Gates concept comes from this https://github.com/sd0xdev/sd0x-dev-flow 

* **hook Interception (runtime hard constraints)**: Claude Code's hook mechanism intercepts tool execution right before it happens. I use it for two things: 
  1. State file write operations are only allowed to be triggered by scheduling-layer agents (any attempts to bypass this are rejected directly).
  2. Dangerous operations (`git push --force`, `rm -rf`) pop up a confirmation prompt. Hooks are not a post-incident audit; they are a **real-time perimeter fence**.

This concept of "externalized quality gates" is becoming an industry consensus. sd0x-dev-flow[7] summarizes it with four keywords: **"hook-enforced dual review, state-machine gates that survive context compaction, and fail-closed safety"**——note the phrase "survive context compaction"; it directly addresses the issue of losing workflow state during the Claude Code Auto-Compact phase. Apache Burr[8] (which has entered the Apache Foundation) has turned this pattern into a generic framework: each Agent's decision-making is expressed as a state machine node + pluggable persistence + a real-time tracking UI.


## How-to-Use 

### Phase 1: Adoption
* **What was done:** Directly adopted community open-source templates (OpenSpec / oh-my-claudecode).
* **Effect:** Stepped up from "using AI rawly" to "using AI with some rules."
* **Newly Exposed Issues:** Generic rules do not fit the specific business scenarios; edge cases rely entirely on makeshift patches.

### Phase 2: Heavy Prompt Constraints ❌
* **What was done:** Wrote all workflow specifications into `CLAUDE.md` as imperative constraints[cite: 1].
* **Effect:** It genuinely works well for the first few days.
* **Newly Exposed Issues:** **Collapsed after three days**——selective compliance / context explosion / self-contradicting rules.

### Phase 3: Offloading + Layered Loading
* **What was done:** Keeping resident text ≤8K / invoking `rules/` on-demand / `context/` files are only loaded upon entering specific phases and released immediately after use.
* **Effect:** Context no longer explodes; the model has breathing room to understand the actual code.
* **Newly Exposed Issues:** In long-running sessions, context gets gradually filled up by code and tool outputs. Although rules are present, they become heavily diluted—the AI forgets which workflow path to take after it finishes writing code.

### Phase 4: Agent Scheduling + Streamlining  
* **What was done:** Streamlined 24 agents down to 9 / each agent boots up from a clean context / workflow states are externalized to files / hooks catch critical errors at the bottom layer.
* **Effect:** `orchestrator` manages the workflow without touching code; `developer` writes code without worrying about workflow—**contexts no longer cross-pollinate, states are never lost, and workflows can gracefully resume.**


**Core Paradigm Shift: Moving from "Constraining AI with more words" to "Constraining AI with better architecture"**




🎯 Core Insight (The Middle Column)

The central pillar of the diagram defines the engineering thesis of this entire architecture:

    The Layering Criterion: The standard for structuring your prompt/context layout is NOT "classification by functionality" , but rather "classification by when it is read".

    The Strategy: Keep the resident/always-on context ultra-small, and load deep context strictly on-demand .

🔍 Structural Breakdown: Before vs. After
❌ Left Side: The "Legacy" Problem (Monolithic Rule Loading)

    The Trap: Jamming everything into the context window (CLAUDE.md Full Rules ≈ 30K tokens).

    The Content: A dense wall of text covering everything from Requirement Analysis, Architecture Design, TDD Specs, Code Comments, Error Handling, CI Environments, to Deploy Checklists.

    The Consequence: Rules are always resident = Token budget is completely exhausted . It leaves almost zero room for the actual "Code Space" at the bottom.

Right Side: The Three-Layer Loading Model (Context Windows)

By moving to a tiered system, the Code Space becomes abundant/roomy .
1. Always-on Entry Layer (CLAUDE.md ≤ 8K)

    Always Present: It contains Role Definitions, Trigger Rules, and Gate Quick References.

    CLAUDE.local.md is self-contained so that different projects do not interfere with one another.

2. Atomic Rules Layer (rules / × 7)

    Loaded On-Demand: * build.md → Forbids mvn -am (addresses critical build hanging incidents).

        branch-hygiene.md → Merge to master first (addresses missing symbol errors).

        code-search.md → Forbids plain Java Grep (addresses context pollution incidents).

    Motto: Every rule is an epitaph of a past production incident.

3. On-Demand Context Layer (context / × 10)

    Phased Loading & Release: * tdd-guide.md → Read during the Coding Phase.

        pre-mortem-template.md → Read during the Risk Assessment Phase.

        debate-template.md → Read during the Adversarial Debate Phase.

    Ammunition Metaphor: "Like ammo—bring it up when fighting a battle, unload it when the battle is done."

⏱️ Loading Sequence Timeline (Bottom Graphic)

The workflow timeline demonstrates how the prompt context shifts dynamically across project phases:

[Design Phase]     ──>     [Coding Phase]     ──>     [Verification]     ──>     [Deployment]
  + Resident Layer           + Resident Layer          + Resident Layer        + Resident Layer
  + debate-template          + tdd-guide               + verification-spec     + deploy-checklist
  (Loaded ⬆️)                (Loaded ⬆️)               (Loaded ⬆️)             (Loaded ⬆️)
                             - debate-template         - tdd-guide
                             (Released ⬇️)             (Released ⬇️)

    Key Takeaway: Every phase only loads the "ammunition" needed for that specific step, keeping the context window stable and tightly controlled.

📊 Summary of Effects

    The Solution: Shifting from ~30K permanently resident tokens to ≤8K permanently resident tokens + dynamic on-demand loading finally leaves the LLM with enough "brain capacity" (working context) to read, process, and write code effectively.

    The Next Challenge (New Problem Identified): In long conversations, the code generated and tool outputs will eventually fill up the context window anyway. The rules, while still present, will inevitably be pushed into the "attention decay zone" (the middle of the context window) due to the Lost in the Middle phenomenon.