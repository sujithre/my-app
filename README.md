# Agent Skills for the Software Development Lifecycle

A portable toolkit of **skills**, **prompt commands**, and **subagents** for GitHub Copilot in
VS Code. Drop the `.github/` folder into any repository and your coding agent gains a
structured, opinionated process for every stage of the SDLC — from a vague idea, through
specification, planning, implementation, review, and launch.

This repository is not an application. It is the process layer you point an agent at *before*
it starts building one.

## The problem this solves

Coding agents are strong at generating code and weak at deciding what to build, in what order,
and when to stop. Left alone they skip specifications, write everything in one pass, mark work
done without running it, and quietly invent requirements nobody asked for.

These skills encode the discipline that prevents that: write the spec first, slice work into
verifiable tasks, prove behaviour with a failing test before implementing, review across
multiple axes, and gate anything irreversible behind human approval.

## Repository layout

```
.github/
  skills/     24 skills — domain knowledge the agent loads on demand
  prompts/     8 slash commands — workflow entry points you invoke directly
  agents/      4 subagents — specialist personas for focused, delegated work
```

## Installation

Copy the `.github` directory into the root of the repository you want to work in:

```bash
git clone https://github.com/sujithre/my-app.git
cp -r my-app/.github /path/to/your-project/
```

On Windows PowerShell:

```powershell
git clone https://github.com/sujithre/my-app.git
Copy-Item -Recurse my-app\.github C:\path\to\your-project\
```

Reload VS Code. The prompts become available as `/` commands in Copilot Chat, and the agent
discovers the skills and subagents automatically.

## Prompt commands

Each prompt is a workflow entry point. Type it in Copilot Chat, optionally with arguments.

| Command | What it does |
|---|---|
| `/spec` | Start spec-driven development — interviews you, then writes a structured specification before any code |
| `/plan` | Break the spec into small verifiable tasks with acceptance criteria and dependency ordering |
| `/build` | Implement the next pending task: failing test, implementation, regression run, build, commit |
| `/build auto` | Implement the entire plan in one approved pass, still test-driven, one commit per task |
| `/test` | Run the TDD workflow; for bug reports it uses the Prove-It pattern to reproduce first |
| `/review` | Five-axis code review — correctness, readability, architecture, security, performance |
| `/code-simplify` | Reduce complexity without changing behaviour |
| `/webperf` | Web performance audit via the web-performance-auditor persona |
| `/ship` | Pre-launch checklist fanned out to specialist personas, synthesized into a go/no-go call |

### The core loop

```
/spec  ──→  /plan  ──→  /build  ──→  /review  ──→  /ship
  │           │           │            │            │
  ▼           ▼           ▼            ▼            ▼
SPEC.md   tasks/plan.md  code +      findings    go / no-go
          tasks/todo.md  commits
```

Each stage gates the next. `/build auto` refuses to run without a real spec at `SPEC.md`,
`docs/SPEC.md`, or under `spec/` — a README does not count, precisely so the agent cannot
invent requirements and call them yours.

## Example

```
/spec A CLI todo app in TypeScript, JSON file storage, add/list/done/delete
```

The agent surfaces its assumptions, asks about storage location, argument parsing, and test
framework, then writes `SPEC.md` covering objective, commands, project structure, code style,
testing strategy, and Always/Ask-first/Never boundaries.

```
/plan
```

It reads the spec, maps the dependency graph, slices the work vertically — one complete user
path per task rather than horizontal layers — and writes `tasks/plan.md` and `tasks/todo.md`
with acceptance criteria, verification steps, and human checkpoints between phases.

```
/build auto
```

It presents the plan once, waits for unambiguous approval, then works through every task
test-first, committing each one separately so any point is a clean rollback. It stops and
asks whenever it hits an ambiguity or anything irreversible.

The `tasks/` directory in this repository holds real output from that first `/plan` run, kept
as a sample of what the commands actually produce.

## Skills

Skills are loaded on demand when the task matches their description. You rarely invoke them
directly — the prompts and the agent pull them in.

**Getting started**
- `using-agent-skills` — the meta-skill governing how the others are discovered
- `context-engineering` — configuring context and rules files for a project
- `interview-me` — extracting what you actually want, one question at a time
- `idea-refine` — sharpening a vague idea through divergent and convergent thinking

**Specify and plan**
- `spec-driven-development` — write the specification before the code
- `planning-and-task-breakdown` — dependency graphs, vertical slicing, task sizing
- `api-and-interface-design` — stable contracts and module boundaries
- `documentation-and-adrs` — recording decisions worth preserving

**Build**
- `test-driven-development` — red, green, refactor; the Prove-It pattern for bugs
- `incremental-implementation` — small verified steps instead of one large pass
- `source-driven-development` — ground every decision in official documentation
- `frontend-ui-engineering` — accessible, responsive, production-quality interfaces
- `code-simplification` — reduce complexity without changing behaviour

**Verify**
- `code-review-and-quality` — the five-axis review
- `debugging-and-error-recovery` — systematic root-cause analysis over guessing
- `doubt-driven-development` — adversarial review of confident decisions
- `browser-testing-with-devtools` — real runtime data via Chrome DevTools MCP
- `security-and-hardening` — untrusted input, auth, storage, integrations
- `performance-optimization` — frontend, backend, queries, databases

**Ship and operate**
- `git-workflow-and-versioning` — commits, branches, semver, changelogs
- `ci-cd-and-automation` — pipelines and quality gates
- `shipping-and-launch` — pre-launch checklist, staged rollout, rollback
- `observability-and-instrumentation` — logs, metrics, traces, alerting
- `deprecation-and-migration` — sunsetting systems and moving users

`skills/references/` holds shared checklists — definition of done, security, accessibility,
performance, observability, testing patterns, and orchestration patterns — referenced by
multiple skills rather than duplicated across them.

## Subagents

Specialist personas with their own context window. Delegating to one keeps its research and
reasoning out of your main conversation.

| Agent | Use for |
|---|---|
| `code-reviewer` | Thorough pre-merge review across all five dimensions |
| `security-auditor` | Vulnerability detection, threat modeling, hardening |
| `test-engineer` | Test strategy, writing suites, coverage analysis |
| `web-performance-auditor` | Core Web Vitals, loading, rendering, network |

## Design principles

- **Gate every phase.** Specify, plan, build, and ship each end in a human review. The agent
  does not advance itself past a gate.
- **Acceptance criteria over instructions.** "Make the dashboard faster" becomes "LCP under
  2.5s on 4G, CLS under 0.1" — something you can actually test against.
- **Surface assumptions before writing.** Ambiguity resolved silently is the most expensive
  kind of bug.
- **Small, verified steps.** No task should touch more than about five files, and each one
  leaves the system working.
- **Boundaries are explicit.** Every spec declares what to always do, what to ask about
  first, and what never to do.

## Compatibility

Built for GitHub Copilot in VS Code, using its skills, prompt files, and custom agents. The
skill and prompt files are plain Markdown with YAML frontmatter, so they port to other agent
tooling with minor frontmatter adjustments.

## License

MIT
