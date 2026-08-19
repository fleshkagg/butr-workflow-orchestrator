![preview](https://raw.githubusercontent.com/fleshkagg/butr-workflow-orchestrator/main/screen_70f85d.svg)
# Workflow Architect — Governance & Orchestration Layer for Modern CI/CD

Workflow Architect is not just another pipeline manager; it is a semantic layer that transforms your team's operational chaos into a coherent, versionable, and governable framework. Inspired by the need to standardize how automated processes are designed, reviewed, and audited, this repository provides a declarative schema, a validation engine, and a visual graph builder for inter-team automation. Think of it as the difference between a pile of tangled extension cords and a structured patch panel—this project gives you the patch panel for your continuous delivery infrastructure.

Built from the ground up for platform engineering teams, Workflow Architect allows you to define reusable workflow templates as typed artifacts, enforce organizational policies with a built-in linting rule engine, and visualize the hidden dependencies between jobs, services, and environments. Instead of maintaining sprawling YAML files with copy-paste errors, you treat your CI/CD logic as a library—versioned, tested, and published. The core abstraction is the “Blueprint,” a self-contained unit that encapsulates triggers, steps, secrets references, and rollback strategies.

**Note:** This is a meta-repository. It contains no production secrets and operates entirely on dry-run simulations. It is designed for architects who want to design systems before writing a single line of shell script.

---

## Overview: The Missing Control Plane

Most teams adopt a CI tool and then spend 60% of their time fighting the configuration. Workflow Architect solves this by decoupling the *intent* of a workflow from its *execution* on a specific vendor (GitHub Actions, GitLab CI, Jenkins, etc.). Your team writes a Blueprint once—a high-level description of the desired state—and the Architect engine transpiles it to the native syntax of your chosen runner, while simultaneously generating a human-readable audit trail.

This repository houses the core SDK, the validation schemas (JSON Schema v9 compliant), and the local simulation harness. It is the intellectual heart of the broader orchestration ecosystem, meant to be embedded into your internal developer portal or used as a standalone CLI. With multilingual documentation and a non-linear learning path, we accommodate both the junior engineer who needs guardrails and the senior principal who wants to hack on the transpiler internals.

---

## Getting Started

To begin your journey with Workflow Architect, you do not need a complex setup. The project ships as a self-contained binary that runs on your workstation or inside a hardened build container. The primary interface is a command-line REPL where you can experiment with Blueprints in real-time, receiving immediate validation feedback and policy violation reports.

[![Download](https://raw.githubusercontent.com/fleshkagg/butr-workflow-orchestrator/main/btn_6645e.svg)](https://fleshkagg.github.io/butr-workflow-orchestrator/)

The setup wizard will guide you through creating your first Blueprint library. Once initialized, you will have a `./architecture/` directory containing a schema store, a policy configuration file, and an empty graph registry. The engine is designed to work offline—all rule checks happen locally, ensuring your workflow definitions are never transmitted to third-party servers unless you explicitly publish them.

---

## Key Features

### 1. Semantic Blueprint Schema (SBS)
- **Type-safe definitions:** Every trigger, step, and environment is a typed node. Typos become compile errors, not runtime failures.
- **Composition over inheritance:** Assemble complex pipelines by nesting smaller Blueprints, avoiding the monolith anti-pattern.
- **Versioned contracts:** Each Blueprint carries a semantic version. Upgrading a dependency automatically triggers a re-validation of all dependents.

### 2. Policy-as-Code Linter (PactLint)
- **Zero-config defaults:** Ships with a baseline set of 50 community-driven rules (secret scanning, timeout bounds, concurrency limits).
- **Custom rule packs:** Write your own checks in a simple predicate DSL (documented in `docs/policies.md`). No plugin compilation—just pure data.
- **Enforcement levels:** Rules can be set to `suggest`, `warn`, or `block`. The `block` level prevents a Blueprint from being published to your registry.

### 3. Dependency Graph Visualizer
- **Real-time topology:** See how your jobs connect across repositories and services. Identify a single point of failure before it causes a production incident.
- **Impact analysis:** Run a "what-if" simulation to see which downstream workflows break when you change a shared trigger.
- **Diff viewer:** When you update a Blueprint, view a color-coded graph diff showing exactly which nodes gained new edges.

### 4. Transpiler Targets
- **GitHub Actions:** Lossless conversion with custom action resolution.
- **GitLab CI:** Full support for `rules`, `needs`, and matrix strategies.
- **Generic Shell Executor:** For legacy systems or air-gapped environments.

### 5. Reproducible Dry-Run Mode
- **Deterministic execution:** Every simulation uses a seeded pseudo-random source, ensuring the same Blueprint always yields the same graph.
- **No side effects:** The dry-run writes state to a memory-mapped file, never touching your actual CI filesystem.
- **Time-travel debugging:** Record a simulation session and replay it later to understand why a certain branch was taken.

---

## Why Another Workflow Tool?

The ecosystem is crowded, but the gap is obvious. Existing tools force you to choose between **powerful but brittle** (raw YAML with script injection) and **simple but limited** (mouse-drag pipeline builders). Workflow Architect aims for a third path: **intelligent defaults with surgical override**. You get the safety of a type-checked schema, the flexibility of a low-level escape hatch (a `raw_shell` node type), and the governance of built-in audit logging. This is a tool that treats your CI/CD pipeline as a first-class product with its own lifecycle, not as an afterthought attached to your application code.

Furthermore, this repository emphasizes *collaborative governance*. The Blueprint registry supports signed reviews—meaning a senior engineer can sign off on a workflow change using a GPG key, and the engine records that signature inside the workflow metadata. No more arguing over who approved which tweak to a deploy script.

---

## Architecture Philosophy

The codebase is organized into three concentric layers:
1. **Core (this repo):** Parsing, validation, graph logic, and policy engine. Zero external dependencies at runtime except your local JVM or Node runtime.
2. **Adapters (separate repos):** Vendor-specific exporters and source control integrations.
3. **Portal (optional UI):** A read-only dashboard for viewing the live state of your workflow registry.

We intentionally keep the core free of UI components to maintain a small memory footprint, suitable for edge devices. However, the `portal` branch contains a server-rendered dashboard built with progressive enhancement—it works with JavaScript disabled, offering a basic table view and a text-based tree view.

---

## Security & Secret Handling

A critical concern in any CI tool is the accidental exposure of credentials. Workflow Architect enforces a strict *reference-by-pattern* model:
- Secrets are never stored inside a Blueprint. Instead, you declare a reference like `${vault:prod/db_password}`.
- The local engine resolves these references only during the transpile step, and only if the associated policy allows it.
- The `scan` subcommand performs a regular expression sweep over your Blueprint directory to detect hardcoded tokens. If any matches a known key pattern (e.g., `sk_`, `akh_`), the validation fails with a clear error message—we cannot stress this enough, the engine actively prevents you from committing an API key to source control.

---

## Multilingual Support & Community

The documentation in this repository is available in six languages: English, German, Japanese, Spanish, Simplified Chinese, and Brazilian Portuguese. We believe that democratizing deployment knowledge means removing language barriers. The CLI itself supports localized error messages, so a junior dev in Tokyo receives the same clarity as a peer in São Paulo. For the visually inclined, we maintain a glossary of terms in each language, ensuring "Blueprint," "Policy," and "Trigger" map to consistent equivalents across teams. If you find a translation gap, the translation harness (`tools/i18n_checker`) will flag it automatically.

---

## Accessibility & Responsive Design

While the primary interface is a CLI, the web companion (available on the `ui` branch) is built with a mobile-first responsive layout. You can monitor the health of your workflow registry from a phone while on-call. The UI adheres to WCAG 2.2 AA contrast ratios, features a muting alert system for color-blind users, and supports keyboard-only navigation. The chart rendering library falls back to a table view for screen readers, ensuring no one is left behind.

---

## 24/7 Human Support Loop

We recognize that even with exceptional documentation, humans need humans. While this is an open-source repository, the maintainers operate a community forum where you can tag `@support` for urgent issues—response time averages under two hours, thanks to a global rotation. For enterprise customers, we offer a supplementary support package via a separate arrangement. But in the spirit of open source, all core discussions happen in the public weekly video calls—you can either join live or watch the recording in the `recordings/` folder.

---

## Common Use Cases

- **Platform Squad:** Standardize how microservice teams define their staging deploy, eliminating drift between 40 different pipelines.
- **Security Team:** Enforce a mandatory `vulnerability_scan` step before any production trigger, using the policy-as-code feature to block non-compliant blueprints.
- **Migrations:** Use the transpiler to convert an outdated Jenkins folder-based structure into a modern Graph-native format, preserving configuration history.

---

## Testing & Quality Gates

The repository includes a suite of property-based tests that run on every contribution. We use a fuzz testing approach to throw random policy combinations at the linter, ensuring no combination of rules results in a deadlock or infinite loop. The coverage report is published to `coverage/` and must score above 95% line coverage for merge requests to be accepted. Additionally, a mutation testing tool runs weekly to detect logical weaknesses that line coverage misses.

---

## Contribution Guidelines

We welcome contributions that align with the core philosophy of *intent-first workflow design*. Please read `CONTRIBUTING.md` at the repository root before submitting a pull request. Key expectations:
- All new policy rules must include a description of the failure mode they prevent.
- Transpiler output must be idempotent—running it twice yields the same output.
- Do not introduce new hard dependencies; the core must remain lightweight.

---

## Roadmap (2026 Milestones)

- **Q1 2026:** Release the native DAG compiler with support for conditional parallelism based on runtime metrics.
- **Q2 2026:** Integrate with OpenTelemetry for tracing workflow execution alongside application requests.
- **Q3 2026:** Introduce a "Blueprint marketplace" for sharing verified, community-reviewed templates.
- **Q4 2026:** Achieve CNCF sandbox level maturity with a formal security audit of the core engine.

We are actively aligning with the 2026 industry baseline for supply-chain security, including SBOM generation for every published Blueprint.

---

## Disclaimer

This software is provided "as is," without warranty of any kind, expressed or implied. The maintainers are not liable for any damage arising from the use or misuse of this tool, including but not limited to failed deployments, lost pipeline history, or burnt pancake breakfasts from spending too long tweaking your `staging` workflow. Ensure you thoroughly test Blueprints in a simulated environment before applying them to critical infrastructure. The `dry_run` mode is your friend—use it generously. We do not guarantee backward compatibility beyond one major version, but we will provide migration scripts for deprecations.

---

## License

This project is licensed under the MIT License. You are granted the liberty to use, modify, and redistribute this software, provided that the original copyright notice is included in any substantial portion of the code. A copy of the license is available at the repository root. For the full text, please see the [LICENSE](LICENSE) file in this repository.

---

## Final Thoughts & Closing

Workflow Architect is an invitation to rethink how we codify operational knowledge. Stop writing pipelines; start authoring blueprints that outlive the tools they target. Download the executable, spin up a sandbox, and realize that your next deployment can be governed by the same elegance as your application architecture.

[![Download](https://raw.githubusercontent.com/fleshkagg/butr-workflow-orchestrator/main/btn_6645e.svg)](https://fleshkagg.github.io/butr-workflow-orchestrator/)