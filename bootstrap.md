---
kind: bootstrap-contract
project: Nompiler
short_name: Nom
status: active
scope: bootstrap-v0
purpose: Scope and acceptance contract for the bootstrap MVP
---

# Bootstrap v0 Scope and Acceptance

## 1. Purpose

Nom v0 is the minimum self-developing version of Nom.

It is not intended to implement every feature in the Charter. It must implement enough of the Charter-to-Design-to-Code
pipeline that Nom can continue developing itself when the user adds or changes requirements on the Charter branch.

## 2. Bootstrap Path

The initial bootstrap path is:

1. The user writes the Nom Charter on the Charter branch.
2. The user manually prompts Codex through the Nom workflow.
3. This produces the `design/bootstrap` branch and the `code/bootstrap` branch.
4. The resulting implementation is Nom v0.
5. Nom v0 reads the Charter branch and generates a new Design branch.
6. Nom v0 autonomously writes, tests, debugs, and refines `code/1.x`.
7. The resulting implementation is Nom v1.

Before using the prompts below, commit all Charter inputs and confirm that the `master` branch worktree is clean. Ensure
that both generated branches exist, creating only those that are missing:

```bash
git switch master
git show-ref --verify --quiet refs/heads/design/bootstrap || git branch design/bootstrap master
git show-ref --verify --quiet refs/heads/code/bootstrap || git branch code/bootstrap master
```

Existing generated branches may contain partial, divergent, or otherwise unexpected bootstrap work. Do not reset or
recreate the branches merely to manufacture a clean starting point. Record their actual heads and lineage, inspect their
content, and treat it as generated input that may be preserved, modified, or removed to satisfy the current
requirements. If reconciliation exposes a Charter or product conflict, report that conflict instead of inventing a
resolution.

### 2.1 Bootstrap Helper

The initial implementation must add a `nom bootstrap` subcommand early, as soon as a usable `nom` executable and basic
repository inspection are available. Until then, the human follows this document directly. The helper is a read-only
advisor for the manual bootstrap, not a second workflow engine and not authoritative runtime state.

`nom bootstrap` is CLI-only. It must print its report and next-action guidance to the terminal and then exit. It must
not start the local web server, launch or open the web UI, or require a browser. Running `nom` without the `bootstrap`
subcommand remains the way to launch the normal local web UI.

When run from the project repository, `nom bootstrap` must inspect the current bootstrap state across the `master`
branch, the `design/bootstrap` branch, and the `code/bootstrap` branch without requiring the human to check out each
one. It should report at least:

- Missing branches, current revisions and lineage, dirty relevant workspaces, and unusual or conflicting branch state.
- Whether the Charter inputs used by the Design still match the current `master` branch.
- Open or blocking questions in `QUESTIONS.md`, including questions found in an unexpected branch or location.
- The latest recorded Design status and whether the Design needs an Architect pass.
- Undispositioned entries in `DESIGN-FINDINGS.md` that require the Architect.
- Whether `BOOTSTRAP-TASKS.md` matches the current Design and which implementation tasks remain or are blocked.
- The latest QA status in `BOOTSTRAP-QA.md`, including whether repeatable checks and live self-development acceptance
  apply to the current Design and Code revisions.

The command must explain inconsistencies rather than guessing or modifying the repository. It should finish with one
primary next action and the reason for it, choosing among initializing missing branches, answering `QUESTIONS.md`,
running or resuming the Architect prompt, running or resuming the Implementer prompt, running a fresh QA prompt,
manually reconciling unsafe state, or declaring the manual bootstrap complete. It may also list secondary blockers.

## 3. Human-Driven Bootstrap Prompts

The initial manual bootstrap intentionally uses only three prompts. It does not attempt to simulate Nom's eventual
runtime orchestration before Nom exists. During this manual stage, the Architect may advance the `design/bootstrap`
branch, and the Implementer and quality agent may advance the `code/bootstrap` branch. The generated Nom implementation
must still satisfy the safe integration, durable recovery, and review requirements elsewhere in this contract.

Run 3.1 and resume or repeat it until the Architect reports `COMPLETE`. Do not start 3.2 while the Design is
`INCOMPLETE` or a blocking question remains unanswered. Run 3.2 once and resume it as needed until every implementation
task is complete or genuinely blocked. Then run 3.3 in a fresh thread for every iteration until it reports `PASS`. If
implementation or QA exposes an architecture-level problem, rerun 3.1 until the revised Design is `COMPLETE`, then
resume 3.2.

At the beginning of each prompted session, the agent should resolve the named branch heads to full commit IDs and report
the commits used. When the Architect needs a product-level answer, it must write the question to `QUESTIONS.md` on the
`design/bootstrap` branch rather than asking the user directly. Each question should include a stable ID, an `OPEN` or
`ANSWERED` status, context, options, the Architect's recommendation, whether it blocks progress, and a reference to the
user's answer on the Charter branch when available. The user records and commits the answer on the `master` branch but
does not edit `QUESTIONS.md`. On the next Architect session, the Architect verifies the answer in the current Charter,
changes the question to `ANSWERED`, and records the exact Charter revision and file that contain the answer. Until then,
the question remains `OPEN`. This keeps user answers durable and available to other Design sessions while preserving the
Architect's ownership of Design artifacts.

Design- and product-level findings from implementation and QA are appended to `DESIGN-FINDINGS.md` on the
`code/bootstrap` branch. Each finding must have a stable ID, type, source, relevant Charter, Design, and Code revisions,
evidence and impact, and whether it blocks progress. Entries are append-only. On every Architect run, the Architect must
read the file and record a disposition for each previously undispositioned finding in the canonical Design decision and
revision log. Product-level findings that require a user decision become questions in `QUESTIONS.md`.

Each QA iteration appends a record to `BOOTSTRAP-QA.md` on the `code/bootstrap` branch containing the Design and Code
revisions assessed, quality status, repeatable checks and results, whether the live self-development test ran, and its
result. The bootstrap helper reads these coordination files only to advise the human; normal Nom orchestration must not
use them as runtime state or as a substitute for its own validation.

### 3.1 Generate the Bootstrap Design

Use this prompt for the Architect on the `design/bootstrap` branch:

```text
Act as the Architect for the Nom bootstrap.

Read the exact Charter snapshot at the head of the `master` branch, including
charter.md, ui.md, and bootstrap.md. Work on the `design/bootstrap` branch in a
dedicated worktree.
Do not implement Nom or edit the Charter inputs.

Inspect all existing content and lineage on the `design/bootstrap` branch rather
than assuming a clean starting state. If the `code/bootstrap` branch contains
DESIGN-FINDINGS.md, read it and inspect relevant Code when needed. Record a
clear disposition for every finding not already addressed by the canonical
Design, revising the architecture and tasks where appropriate.

Generate and commit all canonical Design artifacts required by the Charter. The
Design must include the run manifest and exact inputs, architecture, interfaces
and data model, security and execution assumptions, implementation and
integration strategy, testing and acceptance strategy, quality plan, decision
and revision log, and completion criteria. Resolve technical choices yourself
unless they change product intent.

Record the current COMPLETE or INCOMPLETE Design status and its Charter input in
a documented Design artifact so `nom bootstrap` can inspect it. Define stable,
human-readable conventions for question status, finding dispositions, task
progress, and QA records without turning those files into Nom runtime state.

Write any product-level question to QUESTIONS.md on the `design/bootstrap`
branch with its context, options, your recommendation, and whether it blocks
progress. Do not ask the user directly, and continue all work that is not
blocked by the answer. When the current Charter contains an answer, mark the
question ANSWERED and record the exact Charter revision and file. Otherwise keep
it OPEN.

Create a canonical tasks.md with stable task IDs, dependency order, acceptance
criteria, and required tests. Start each implementation task with an unchecked
Markdown checkbox. Make each task small enough to implement and verify
independently. The Implementer will mirror these task IDs and completion status
into BOOTSTRAP-TASKS.md on the `code/bootstrap` branch; that mirror must not
redefine the canonical tasks.

Make an early implementation milestone produce the `nom` executable with a
read-only `nom bootstrap` subcommand. It may initially report incomplete state,
but it must remain useful and accurate as later tasks add coordination artifacts
and acceptance evidence.

Create an implementation-independent black-box acceptance harness and fixtures
on the `design/bootstrap` branch. Freeze them in the Design commit before Code
implementation begins. Provide separate entry points for the repeatable
orchestration suite and the live self-development test, and document them in the
canonical Design.
Implementers must not modify the harness to make generated Code pass. Document
exact Podman commands that run each entry point from its Design commit against
the generated `nom` executable. The commands must use disposable container-local
repository copies and must not mount the real repository or any of its worktrees
writable.

Before finishing, review the entire Design end to end. Check every artifact for
internal consistency and trace every Charter and bootstrap requirement to
architecture, tasks, acceptance criteria, and tests. Verify task
dependencies and coverage, look for missing behavior and contradictory
decisions, and confirm that the frozen acceptance harness exercises the required
system behavior. Resolve every issue you can and repeat the review after changes.

Then output exactly one Design status: COMPLETE or INCOMPLETE. Use COMPLETE only
if the whole Design is coherent, every normative requirement is covered, the
task plan can implement it, the acceptance strategy can verify it, and no
blocking question or known Design issue remains. Otherwise use INCOMPLETE and
list the blockers and remaining Design work.

Commit the reviewed Design and report the exact Design commit, Design status,
review evidence, ordered task list, dispositions of DESIGN-FINDINGS.md entries,
whether QUESTIONS.md needs a user answer, and the first ready task.
```

### 3.2 Implement the Complete Bootstrap Design

Use this prompt for the Implementer on the `code/bootstrap` branch:

```text
Act as the Implementer for the complete Nom bootstrap Design.

Work in a dedicated writable worktree on the `code/bootstrap` branch. Read the
heads of the `master` branch and the `design/bootstrap` branch and record all
resolved commit IDs. Do not edit the Charter inputs or canonical Design
artifacts.

Inspect the existing `code/bootstrap` branch without assuming that it shares a
clean starting point with the Charter or Design branches. Treat its content as
generated input that may be preserved, modified, or removed. Reconcile existing
Code and progress records with the current Design, and stop only when continuing
requires a product or architecture decision.

Read the complete Design and its canonical tasks.md. Create or update
BOOTSTRAP-TASKS.md on the `code/bootstrap` branch. Record the exact Charter and
Design commits, then mirror every canonical task ID as a Markdown checkbox. The
file tracks implementation status only; do not rewrite task definitions there.
If the Design changed, add new or revised task IDs and mark superseded IDs
explicitly rather than silently deleting history.

BOOTSTRAP-TASKS.md is only a progress ledger for the initial human-driven
bootstrap. The `nom bootstrap` advisor may read it, but normal Nom orchestration
must not use it for scheduling, recovery, completion, or any other runtime
behavior. Nom must keep its operational task state in its durable runtime state.

Implement tasks in dependency order. For each task, implement the full behavior,
add or update tests for every acceptance criterion, and run the relevant
formatting, static checks, tests, and integration checks. Check off the task in
BOOTSTRAP-TASKS.md only after its implementation and required tests pass. Include
the checkbox update in the same commit or merged diff as the Code that completes
the task. Never mark partial, failing, or unverified work complete.

Run every test, and every build or validation step that can execute
project-controlled code, through the Podman-based entry points defined by the
Design. Do not execute them directly against the host repository or expose the
real repository, its worktrees, or Git metadata as writable container storage.
If Podman is unavailable, report the check as blocked rather than falling back
to host execution.

Prioritize the early Design milestone that creates the `nom` executable and
`nom bootstrap` advisor. Test it against missing, partial, divergent, blocked,
ready-for-implementation, ready-for-QA, and completed bootstrap states. Its
advice must be based on repository evidence and must never modify that state.

You may use internal task branches when useful, but merge each completed task's
Code, tests, and BOOTSTRAP-TASKS.md update into the `code/bootstrap` branch
before starting the next task. Do not modify the frozen acceptance harness on
the `design/bootstrap` branch.

Continue until every task is checked off or progress is genuinely blocked. Send
Design- or product-level findings to the Architect by appending them to
DESIGN-FINDINGS.md with stable IDs, types, relevant revisions, evidence, impact,
and blocking status. Reuse an existing ID when appending evidence to the same
finding, and do not duplicate or rewrite earlier entries or invent product
behavior.
Commit the findings to the `code/bootstrap` branch. At the end, run the full
project test suite and report the final Code commit, completed and blocked task
IDs, checks run with outcomes, known limitations, and any Design revisions still
required.
```

### 3.3 Test, Debug, QA, and Improve Code Quality

Run every iteration of this prompt in a fresh Codex thread on the `code/bootstrap` branch after implementation. Do not
resume the Implementer thread or a previous QA thread. Repeat until an iteration reports `PASS`:

```text
Act as the senior test, debugging, QA, and code-quality agent for the generated
Nom bootstrap implementation.

Independently inspect the implementation rather than relying on the Implementer's
summary or assumptions.

Work in a dedicated writable worktree on the `code/bootstrap` branch. Resolve
and record the heads of the `master` branch, the `design/bootstrap` branch, and
the `code/bootstrap` branch. Read the Charter, complete Design,
BOOTSTRAP-TASKS.md, DESIGN-FINDINGS.md when present, BOOTSTRAP-QA.md when
present, and the frozen acceptance harness. Do not edit the Charter, canonical
Design, or frozen acceptance harness.

Inspect existing Code and progress records without assuming a clean bootstrap
lineage. Treat manual changes as generated state that may be preserved, modified,
or removed. Do not test or integrate work based on a stale revision.

Use the Podman-based commands documented by the Architect to build Nom and run
all unit, integration, end-to-end, and black-box tests. Run static analysis in
Podman whenever it can execute project-controlled code. Use only disposable
container-local repository copies; never mount the real repository or its
worktrees writable. Run purely mechanical checks such as formatting as
documented by the Design. Exercise normal user flows and adversarial failure
paths, including restart and resume, mid-run Charter updates, safe-turn
delivery, stale-work rejection, controlled integration, repository hygiene,
sandbox approval handling, browser security protections, durable recovery, and
revision traceability.

Investigate every failure and significant warning. Reproduce bugs, identify root
causes, fix the Code, and add regression tests. Improve correctness, security,
test coverage, error handling, maintainability, and performance where the
benefit is concrete and consistent with the Design. Avoid speculative rewrites
or product changes. Append every Design- or product-level finding to
DESIGN-FINDINGS.md with a stable ID, type, relevant revisions, evidence, impact,
and blocking status, then commit it to the `code/bootstrap` branch. Reuse an
existing ID when appending evidence to the same finding rather than duplicating
it. Do not mask these problems in Code.

Commit fixes and tests to the `code/bootstrap` branch, then rerun all affected
checks and the repeatable suite. Continue within this run until no known
actionable defect remains or an issue requires an Architect or user decision.
Do not mark an incomplete Design task complete merely to make QA pass.

Do not run the live self-development test while a repeatable check is failing or
after this QA iteration has changed Code or tests. Run it only when the current
iteration has found no required change and all repeatable checks pass. Use the
separate live-test command documented by the Architect. Treat any live-test
failure as an actionable defect or blocker. If it fails, investigate and fix it
as above, rerun the repeatable checks, and report NEEDS_WORK; leave the next live
attempt to a fresh independent QA iteration.

Then output exactly one quality status: PASS, NEEDS_WORK, or BLOCKED. Use PASS
only if every current Design task is checked off in BOOTSTRAP-TASKS.md, every
repeatable check and the live self-development test passes, no blocking finding
or known actionable defect remains, and this QA iteration made no Code or test
changes. If this iteration committed any fix, use NEEDS_WORK even when all
repeatable checks now pass so a new independent no-change QA iteration can run
the live test. Use BLOCKED when progress requires an Architect or user decision.

Append this iteration's exact Design and assessed Code revisions, quality status,
checks and results, live-test status, and remaining blockers to BOOTSTRAP-QA.md.
Commit the record to the `code/bootstrap` branch. Do not rewrite prior iteration
records or treat this file as Nom runtime state.

Report the final Code commit, quality status, commands and scenarios tested,
failures found, root causes and fixes, regression coverage, remaining risks, and
blocked issues. State explicitly whether the live self-development test ran and
its result.
```

## 4. Bootstrap Operating Profile

Nom v0 is:

- Local-only.
- Single-user.
- Operated through a local web UI.
- Limited to one active Nom run per repository.

The web UI does not require login or application authentication. It must bind only to the loopback interface, with no
network-binding override in v0. Lack of login does not remove normal browser protections such as CSRF, origin, and host
validation.

The required `nom bootstrap` subcommand assists the initial human-driven bootstrap. Nom v0 may provide other internal
debugging or maintenance commands, but a CLI is not the normal product interface.

“Local-only” applies to Nom's UI and orchestration service. Codex may still use the service connectivity required by
its configured runtime.

The local web UI must implement the in-scope user flow defined in the Charter. User-triggered Release branch creation
and stabilization are deferred under the explicit v0 non-goals.

## 5. Codex Agent Runtime

Nom v0 uses Codex as its only agent runtime. The bootstrap Architect chooses and records a supported local integration
that can start and resume agents, observe their progress and approval requests, apply appropriate sandbox boundaries,
obtain reliable completion results, report token usage, and expose the remaining weekly usage quota when Codex provides
it.

Each new agent run must receive:

- A unique run and role identifier.
- An isolated workspace based on immutable repository revisions.
- The relevant Charter, Design, task, and Code context.
- Sandbox boundaries appropriate to the role.

Agents must receive only the filesystem, network, and tool access needed for their role. Writable access should normally
be limited to the agent's isolated workspace, and unrestricted full-filesystem access must not be the default. Nom uses
Codex to enforce the selected sandbox while remaining responsible for choosing its boundaries and agent context.

An agent run may contain multiple turns on the same resumed Codex thread and in the same isolated workspace. A
continuation turn is not a new agent run.

## 6. Implementation Loop

Nom v0 executes implementation tasks sequentially. Parallel implementation and conflict resolution are deferred until
after v0.

For each task, Nom must:

1. Resolve the current Charter, Design, and Code inputs to immutable revisions.
2. Run the Implementer agent in an isolated workspace based on those revisions.
3. Run the task's required tests and checks.
4. Run an independent review against the Charter, Design, task, and resulting changes.
5. Return review findings to the implementation agent for correction when appropriate.
6. Return architecture-level findings to the Architect.
7. Integrate approved work only if the selected Code branch still has the expected contents and the active Design has
   reconciled all observed Charter changes.

Rejected or stale work must not be silently integrated. If the Design or Code base changes, Nom treats the new content
as generated state, has the Architect reconcile or replan as needed, and retries or restarts invalidated work. A manual
change to a generated branch does not by itself require user intervention.

## 7. Asynchronous Charter Updates

Nom v0 must implement the asynchronous Charter-update behavior defined by the Charter, including safe-turn delivery,
Architect reconciliation, stale-work rejection, and durable recovery. The bootstrap acceptance suites must exercise
that behavior.

## 8. Durable State and Recovery

Nom v0 must keep durable local operational state separately from generated project branches. It must retain enough
information to recover runs and tasks, Codex threads and isolated workspaces, repository revisions and lineage, pending
agent messages, user questions and answers, token budgets and reported usage, reported weekly quota status, validation
results, and failures or blocked work.

After restart, Nom must reconcile this state with the repository and agent workspaces before continuing. Retrying an
operation must not silently duplicate an already integrated change. The Architect chooses the storage technology and
schema.

## 9. Bootstrap Completion Constraints

Nom v0 follows the completion authority and quality requirements in the Charter. The Architect may exclude a requirement
only when it is an explicit bootstrap non-goal or Charter exception; it may not waive requirements by relabeling them as
out of scope. Architecture and quality loops have no fixed iteration limit. Operational retries may be bounded, but
exhausting them blocks progress and never permits incomplete work to be reported as complete. The user may cancel at
any time. Token-budget exhaustion and the weekly-quota safety threshold pause work without weakening the completion
criteria.

## 10. Bootstrap Acceptance Tests

Nom v0 is accepted only after both of the following pass.

The repeatable suite should run throughout implementation and QA. Reserve the live self-development test for a candidate
that has passed the repeatable suite without requiring Code or test changes in the current independent QA iteration. Any
later Design, Code, or test change invalidates the live result and requires another live test before acceptance.

Both suites must run inside Podman against disposable repositories. Neither suite may modify the real repository or its
branches.

The live self-development test's synthetic Charter updates and generated `design/*` and `code/*` branches are discarded
after evidence is collected. The later production Nom v0-to-v1 run described in the bootstrap path is a separate,
user-initiated event after Nom v0 is accepted.

### 10.1 Repeatable Orchestration Test

Automated, repeatable tests may use a deterministic scripted agent stub to demonstrate orchestration behavior. Codex
remains the only production agent runtime. The tests must demonstrate:

- Local web run creation and status reporting.
- Required per-run token budgets, usage tracking, and safe stopping when a budget is exhausted.
- Safe stopping when a scripted Codex report places the remaining weekly usage quota below 15 percent.
- Architect-generated Design artifacts.
- Isolated workspaces for agent runs.
- Architecture and sequential implementation loops.
- Review and correction handling.
- Durable queued messages at turn boundaries.
- Mid-run Charter update reconciliation.
- Restart and resume.
- Stale-work detection.
- Reconciliation of manual changes to active Design and Code branches without requiring user intervention.
- Safe failure without false completion.
- Durable revision traceability.
- Read-only `nom bootstrap` guidance for missing, inconsistent, blocked, in-progress, QA-ready, and completed states.
  This item applies only to Nom v0 on the `code/bootstrap` branch. The `nom bootstrap` advisor exists for the initial
  human-driven bootstrap, and the harness must not require it of Nom v1 or later generated versions.

### 10.2 Live Self-Development Test

Using real Codex agents and a disposable repository inside Podman, Nom v0 must:

1. Run from an immutable revision of the `code/bootstrap` branch with a user-provided token budget.
2. Read an immutable Nom Charter revision.
3. Have the Architect generate the complete bootstrap Design without user-authored Design artifacts.
4. Create or update a `code/1.x` lineage derived from an immutable Code revision.
5. Receive a controlled Charter update while a Codex turn is active, deliver it at the next safe turn boundary, and
   carry it through a new Design revision into reviewed Code without discarding the run.
6. Restart the Nom process during the run and resume without losing the Charter event, duplicating integration, or
   requiring the user to recreate state.
7. Implement, test, review, and refine the work until the Architect declares completion.
8. Produce a working Nom v1 that passes the implementation-independent acceptance harness and fixtures frozen on the
   Design branch before implementation began.
9. Launch the generated Nom v1 and have v1 complete a small fixture Charter-to-Design-to-Code change without
   user-authored Design artifacts or direct manual prompting of its agents.
10. Trace the completed work to the Charter, final Design, and reviewed Code revisions, including the Charter update
    received during the run.

The self-development test compares required behavior, tests, and traceability. It does not require source-code equality
or a cleanroom reproduction.

## 11. Explicit Non-Goals for v0

The following are outside the required bootstrap scope:

- Hosted or network-accessible UI.
- Authentication, accounts, or multiple users.
- Remote Git pushes or Git-provider integration.
- Parallel implementation agents.
- Multiple active runs or distributed Nom orchestration for one repository.
- Multi-repository projects.
- Distributed scheduling.
- Agent runtimes other than Codex.
- Additional isolation for agent execution beyond the configured Codex sandbox. This does not exclude the required
  Podman isolation for test execution.
- User-triggered Release branch creation and stabilization.
- Numeric universal code-quality scoring.
- Exact source reproduction or cleanroom self-hosting convergence.

## 12. Decisions Delegated to the Bootstrap Architect

The bootstrap Architect should choose and record technical details that do not change the product contract, including:

- Web framework.
- Durable state storage and schema.
- Exact Design artifact schemas and filenames.
- Codex prompt and structured-output schemas.
- Charter monitoring and change-ordering mechanism.
- Isolated workspace and internal branch strategy.
- Traceability metadata and safe publication protocol.
- Project check discovery and execution.
- Retry timing and operational backoff.
- The concrete quality rubric used by the Architect.

The Architect should escalate only if a choice changes user-visible behavior, the accepted bootstrap scope, or a major
product constraint.
