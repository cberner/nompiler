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
2. The user runs the single seed prompt in Section 2.1 to produce the early `nom bootstrap` workflow helper, then uses
   that helper to conduct every remaining bootstrap role.
3. This produces the `code/bootstrap` branch, with canonical Design artifacts under `.nom/design/`.
4. The resulting implementation is Nom v0.
5. Nom v0 reads the Charter branch and generates a new Design revision on a Code branch.
6. Nom v0 autonomously writes, tests, debugs, and refines `code/1.x`.
7. The resulting implementation is Nom v1.

Before using the prompts below, commit all Charter inputs and confirm that the `master` branch worktree is clean. Ensure
that the bootstrap Code branch exists, creating it only if it is missing:

```bash
git switch master
git show-ref --verify --quiet refs/heads/code/bootstrap || git branch code/bootstrap master
```

The existing generated branch may contain partial, divergent, or otherwise unexpected bootstrap work. Do not reset or
recreate it merely to manufacture a clean starting point. Record its actual head and lineage, inspect its content, and
treat it as generated input that may be preserved, modified, or removed to satisfy the current requirements. If
reconciliation exposes a Charter or product conflict, report that conflict instead of inventing a resolution.

### 2.1 Bootstrap Helper

The user runs the following single seed prompt from the clean `master` worktree. This is an intentional one-time
exception to the Design-first workflow: it may implement only the bootstrap helper before the canonical Design exists.
The later Architect must reconcile the seed implementation as generated input and may design for it to be preserved,
revised, or replaced.

```text
Act as the seed Implementer for the Nom bootstrap helper.

Read bootstrap.md and AGENTS.md from the current master head. Work in a dedicated
writable worktree on code/bootstrap; do not switch or modify the master worktree.
Inspect and reconcile the existing code/bootstrap branch and never reset or
recreate it merely to obtain a clean starting point. Do not modify .nom/design/.

Implement only the `nom bootstrap` command described in Section 2.1. Treat
Sections 3.1 through 3.3 as prompts the command launches, not as work to perform
during this seed step. Treat all other requirements as constraints rather than
additional implementation scope. This seed implementation is the explicit
one-time exception that precedes the canonical Bootstrap Design.

Add focused tests for the helper. Run every build, test, and validation step that
can execute project-controlled code in Podman against a disposable repository
copy; never expose the real repository or its worktrees as writable test storage.

Commit the implementation and tests to code/bootstrap. Continue until the helper
builds and its focused tests pass. Copy the resulting `nom` executable out of the
container without committing it, then report its path, the exact Code commit, and
all checks run with their outcomes.
```

The initial implementation must add a `nom bootstrap` subcommand early, as soon as a usable `nom` executable and basic
repository inspection are available. Until then, the human follows this document directly. The helper is a small,
bootstrap-only workflow engine, not a second implementation of normal Nom orchestration. It does not implement token
budgets, quota monitoring, persistent run state, or other normal Nom workflow features.

The command accepts `--start-role <architect|implementer|qa>` and `--iteration-limit <count>`, with an iteration limit of
100 by default. Without `--start-role`, it derives the next role from repository evidence. A requested starting role is
still subject to the role prerequisites. The iteration limit counts every prompted role session, including transitions
back to the same role; reaching it stops the command without reporting the bootstrap complete.

The bootstrap workflow states are:

- Architect
- Implementer
- QA
- Done

At startup and after every prompted role session, the helper inspects the current `master` and `code/bootstrap` revisions
and their bootstrap artifacts. It launches the corresponding prompt from Section 3 only after checking its prerequisites,
and every QA iteration uses a fresh Codex session.

When a role session finishes, the helper starts a fresh read-only Codex session that receives the final message and the
current repository inspection and returns the next state and its reason as structured output. The helper validates that
recommendation against the repository before continuing. In particular, it enters `Done` only when the current state
satisfies the Section 3.3 `PASS` conditions. If a session fails, the iteration limit is reached, or user action is needed,
the command stops and explains why. Running it again re-inspects the repository and continues from the resulting state.

`nom bootstrap` is CLI-only. It must print its progress to the terminal. The output should include the final messages
from each Codex session as the state transitions happen. It must not start the local web server, launch or open the web
UI, or require a browser. Running `nom` without the `bootstrap` subcommand remains the way to launch the normal
local web UI.

When run from the project repository, `nom bootstrap` must inspect the current bootstrap state across the `master`
branch and the `code/bootstrap` branch without requiring the human to check out either one. It should report at least:

- A missing branch, current revisions and lineage, dirty relevant workspaces, and unusual or conflicting branch state.
- Whether the Charter inputs used by the Design still match the current `master` branch.
- Open or blocking questions in `.nom/design/QUESTIONS.md`, including questions found in an unexpected location.
- The latest recorded Design status and whether the Design needs an Architect pass.
- Undispositioned entries in `DESIGN-FINDINGS.md` that require the Architect.
- Whether `BOOTSTRAP-TASKS.md` matches the current Design and which implementation tasks remain or are blocked.
- The latest QA status in `BOOTSTRAP-QA.md`, including whether repeatable checks and live development acceptance
  apply to the current Design and Code revisions.

The helper itself must not edit role-owned artifacts or guess how to reconcile inconsistent state; repository changes
belong to the prompted roles. If user action is required, such as initializing a missing branch, answering
`.nom/design/QUESTIONS.md`, or reconciling unsafe repository state, the command stops and clearly explains what to do.

## 3. Bootstrap Role Prompts

The `nom bootstrap` helper, not the user, launches the following prompts. The Architect, Implementer, and quality agent
advance the `code/bootstrap` branch sequentially within their artifact-ownership boundaries. The generated Nom
implementation must still satisfy the safe integration, durable recovery, and review requirements elsewhere in this
contract.

The helper runs or repeats 3.1 until the Architect reports `COMPLETE`. It does not start 3.2 while the Design is
`INCOMPLETE` or a blocking question remains unanswered. It runs or repeats 3.2 until every implementation task is
complete or genuinely blocked, then runs 3.3 in a fresh thread for every iteration until it reports `PASS`. If
implementation or QA exposes an architecture-level problem, the helper returns to 3.1 until the revised Design is
`COMPLETE`, then continues with 3.2.

At the beginning of each prompted session, the agent should resolve the named branch heads to full commit IDs and report
the commits used. When the Architect needs a product-level answer, it must write the question to
`.nom/design/QUESTIONS.md` on the `code/bootstrap` branch rather than asking the user directly. Each question should
include a stable ID, an `OPEN` or `ANSWERED` status, context, options, the Architect's recommendation, whether it blocks
progress, and a reference to the user's answer on the Charter branch when available. The user records and commits the
answer on the `master` branch but does not edit `.nom/design/QUESTIONS.md`. On the next Architect session, the Architect
verifies the answer in the current Charter, changes the question to `ANSWERED`, and records the exact Charter revision
and file that contain the answer. Until then, the question remains `OPEN`. This keeps user answers durable and available
to other Design sessions while preserving the Architect's ownership of Design artifacts.

Design- and product-level findings from implementation and QA are appended to `DESIGN-FINDINGS.md` on the
`code/bootstrap` branch. Each finding must have a stable ID, type, source, relevant Charter, Design, and Code revisions,
evidence and impact, and whether it blocks progress. Entries are append-only. On every Architect run, the Architect must
read the file and record a disposition for each previously undispositioned finding in the canonical Design decision and
revision log. Product-level findings that require a user decision become questions in `.nom/design/QUESTIONS.md`.

Each QA iteration appends a record to `BOOTSTRAP-QA.md` on the `code/bootstrap` branch containing the Design and Code
revisions assessed, quality status, repeatable checks and results, whether the live development test ran, and its
result. The bootstrap helper may use these coordination files to choose the next prompt, but must validate their recorded
revisions against the repository. Normal Nom orchestration must not use them as runtime state.

### 3.1 Generate the Bootstrap Design

Use this prompt for the Architect on the `code/bootstrap` branch:

```text
Act as the Architect for the Nom bootstrap.

Read the exact Charter snapshot at the head of the `master` branch, including
charter.md, ui.md, and bootstrap.md. Work in a dedicated writable worktree on
the `code/bootstrap` branch. Do not implement Nom, edit the Charter inputs, or
modify files outside `.nom/design/`.

Inspect all existing content and lineage on the `code/bootstrap` branch rather
than assuming a clean starting state. If it contains DESIGN-FINDINGS.md, read it
and inspect relevant Code when needed. Record a clear disposition for every
finding not already addressed by the canonical Design, revising the architecture
and tasks where appropriate.

The existing `nom bootstrap` seed implementation was intentionally created before
the canonical Design. Treat it as generated input with no special preservation
status. Reconcile it into the Design and tasks without modifying implementation
paths in this Architect role.

Generate and commit all canonical Design artifacts required by the Charter under
`.nom/design/`. The Design must include the run manifest and exact inputs,
architecture, interfaces and data model, security and execution assumptions,
implementation and integration strategy, testing and acceptance strategy,
quality plan, decision and revision log, and completion criteria. Resolve
technical choices yourself unless they change product intent.

Record the current COMPLETE or INCOMPLETE Design status and its Charter input in
a documented Design artifact so `nom bootstrap` can inspect it. Define stable,
human-readable conventions for question status, finding dispositions, task
progress, and QA records without turning those files into Nom runtime state.

Write any product-level question to `.nom/design/QUESTIONS.md` with its context,
options, your recommendation, and whether it blocks progress. Do not ask the
user directly, and continue all work that is not blocked by the answer. When the
current Charter contains an answer, mark the question ANSWERED and record the
exact Charter revision and file. Otherwise keep it OPEN.

Create a canonical `.nom/design/tasks.md` with stable task IDs, dependency order,
acceptance criteria, and required tests. Start each implementation task with an
unchecked Markdown checkbox. Make each task small enough to implement and verify
independently. The Implementer will mirror these task IDs and completion status
into BOOTSTRAP-TASKS.md on the `code/bootstrap` branch; that mirror must not
redefine the canonical tasks.

Make an early implementation milestone produce the `nom` executable with the
`nom bootstrap` subcommand described in Section 2.1. Its first increment may
provide inspection and guidance before Codex session launching is implemented,
but it must grow into the specified workflow engine during this bootstrap.

Create an implementation-independent black-box acceptance harness and fixtures
under `.nom/design/`. Freeze them in the Design commit before Code implementation
begins. Provide separate entry points for the repeatable orchestration suite and
the live development test, and document them in the canonical Design.
Implementers must not modify the harness to make generated Code pass. Document
exact Podman commands that run each entry point from its immutable Design
revision against the generated `nom` executable. The commands must use
disposable container-local repository copies and must not mount the real
repository or any of its worktrees writable.

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

Commit only the reviewed `.nom/design/` changes and report the exact Design
commit, Design status, review evidence, ordered task list, dispositions of
DESIGN-FINDINGS.md entries, whether `.nom/design/QUESTIONS.md` needs a user
answer, and the first ready task.
```

### 3.2 Implement the Complete Bootstrap Design

Use this prompt for the Implementer on the `code/bootstrap` branch:

```text
Act as the Implementer for the complete Nom bootstrap Design.

Work in a dedicated writable worktree on the `code/bootstrap` branch. Read the
heads of the `master` branch and the `code/bootstrap` branch and record all
resolved commit IDs. Do not edit the Charter inputs or anything under
`.nom/design/`.

Inspect the existing `code/bootstrap` branch without assuming a clean starting
point. Treat its content as generated input that may be preserved, modified, or
removed, except that `.nom/design/` is Architect-owned and read-only for this
role. Reconcile existing Code and progress records with the current Design, and
stop only when continuing requires a product or architecture decision.

Read the complete Design and its canonical `.nom/design/tasks.md`. Create or
update BOOTSTRAP-TASKS.md on the `code/bootstrap` branch. Record the exact Charter
and Design revisions, then mirror every canonical task ID as a Markdown checkbox.
The file tracks implementation status only; do not rewrite task definitions
there. If the Design changed, add new or revised task IDs and mark superseded IDs
explicitly rather than silently deleting history.

BOOTSTRAP-TASKS.md is only a progress ledger for the initial bootstrap. The
`nom bootstrap` helper may use it to choose the next prompt but must validate its
recorded revisions against the repository. Normal Nom orchestration must not use
it for scheduling, recovery, completion, or any other runtime behavior.

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
`nom bootstrap` helper. Test its inspection, role selection, stopping, rerun
continuation, and completion behavior against missing, partial, divergent, blocked,
ready-for-implementation, ready-for-QA, and completed bootstrap states. Its
decisions must be based on repository evidence, and only its prompted roles may
modify role-owned artifacts.

You may use internal task branches when useful, but merge each completed task's
Code, tests, and BOOTSTRAP-TASKS.md update into the `code/bootstrap` branch
before starting the next task. Do not modify `.nom/design/`, including the frozen
acceptance harness.

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
and record the heads of the `master` branch and the `code/bootstrap` branch. Read
the Charter, complete Design under `.nom/design/`, BOOTSTRAP-TASKS.md,
DESIGN-FINDINGS.md when present, BOOTSTRAP-QA.md when present, and the frozen
acceptance harness. Do not edit the Charter or anything under `.nom/design/`.

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

Do not run the live development test while a repeatable check is failing or
after this QA iteration has changed Code or tests. Run it only when the current
iteration has found no required change and all repeatable checks pass. Use the
separate live development test command documented by the Architect. Treat any
failure as an actionable defect or blocker. If it fails, investigate and fix it
as above, rerun the repeatable checks, also rerun the live development test and
iterate on further fixes if need, and report NEEDS_WORK; leave the next live
attempt to a fresh independent QA iteration.

Then output exactly one quality status: PASS, NEEDS_WORK, or BLOCKED. Use PASS
only if every current Design task is checked off in BOOTSTRAP-TASKS.md, every
repeatable check and the live development test passes, no blocking finding
or known actionable defect remains, and this QA iteration made no Code or test
changes. If this iteration committed any fix, use NEEDS_WORK even when all
repeatable checks now pass so a new independent no-change QA iteration can run
the live test. Use BLOCKED when progress requires an Architect or user decision.

Append this iteration's exact Design and assessed Code revisions, quality status,
checks and results, live development test status, and remaining blockers to
BOOTSTRAP-QA.md.
Commit the record to the `code/bootstrap` branch. Do not rewrite prior iteration
records or treat this file as Nom runtime state.

Report the final Code commit, quality status, commands and scenarios tested,
failures found, root causes and fixes, regression coverage, remaining risks, and
blocked issues. State explicitly whether the live development test ran and its
result.
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

The required `nom bootstrap` subcommand conducts the remaining initial bootstrap after the early executable exists. Nom
v0 may provide other internal debugging or maintenance commands, but a CLI is not the normal product interface.

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
exhausting them returns their evidence to the Architect and never permits incomplete work to be reported as complete.
The user may cancel at any time. Token-budget exhaustion and the weekly-quota safety threshold pause work without
weakening the completion criteria.

## 10. Bootstrap Acceptance Tests

Nom v0 is accepted only after both of the following pass.

The repeatable suite should run throughout implementation and QA. Reserve the live development test for a candidate that
has passed the repeatable suite without requiring Code or test changes in the current independent QA iteration. Any
later Design, Code, or test change invalidates the live result and requires another live test before acceptance.

Both suites must run inside Podman against disposable repositories. Neither suite may modify the real repository or its
branches.

The live development test's synthetic Charter updates and generated `code/*` branches are discarded after evidence is
collected. The later production Nom v0-to-v1 run described in the bootstrap path is a separate, user-initiated
self-hosting event after Nom v0 is accepted.

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
- Reconciliation of manual changes to active Code branches, including Design collateral, without requiring user
  intervention.
- Safe failure without false completion.
- Durable revision traceability.
- Repository-derived `nom bootstrap` inspection and orchestration for missing, inconsistent, blocked, in-progress,
  QA-ready, and completed states, including validated role overrides, fresh QA sessions, iteration-limit stopping, and
  rejection of unsupported completion claims. This item applies only to Nom v0 on the `code/bootstrap` branch; the
  harness must not require the helper of Nom v1 or later generated versions.

### 10.2 Live Development Test

The frozen acceptance harness must prepare a disposable repository whose immutable Charter describes a base-7
command-line expression calculator. The fixture contract requires:

- An executable named `calc`.
- A single expression supplied as a command-line argument.
- Signed integer arithmetic using `+`, `-`, `*`, and `/`, with standard precedence and parentheses.
- Integer division that truncates toward zero and rejects division by zero with a nonzero exit status.
- Operands and results represented in the active base. Successful evaluation prints only the normalized result followed
  by a newline to standard output and exits successfully.
- An invalid digit to print `nice try` to standard error and exit unsuccessfully. In base 7, digits greater than 6 are
  invalid; after the Charter changes to base 8, digit 7 becomes valid and digits greater than 7 trigger the message.

The harness must also freeze implementation-independent black-box tests for this contract before the run. These tests
remain outside the agents' writable repository. Agent-authored unit and integration tests supplement but do not replace
the frozen tests.

The test must use a prepared container image containing the Codex npm package at the version recorded by the Design.
Preparing this reusable image is harness setup, not part of each live test run.

Using real Codex agents and the disposable repository inside Podman, Nom v0 must then:

1. Build the Nom candidate in Podman and transfer the resulting binary into the disposable test container.
2. After creating an ephemeral container from the prepared image, copy the required Codex authentication credentials
   from the host with restrictive permissions. Never include them in an image, container snapshot, log, test artifact,
   or generated repository, and destroy the container and its copied credential files after the test even if it fails.
3. Start the Nom run with a user-provided token budget and record the candidate and immutable calculator Charter
   revisions.
4. Have the Architect generate the complete Design without user-authored Design artifacts.
5. Create or update a `code/1.x` lineage derived from an immutable Code revision.
6. Receive a controlled Charter update switching the calculator to base 8 while a Codex turn is active, deliver it at
   the next safe turn boundary, and carry it through a new Design revision into reviewed Code without discarding the
   run.
7. Restart the Nom process during the run and resume without losing the Charter event, duplicating integration, or
   requiring the user to recreate state.
8. Implement, test, review, and refine the work until the Architect declares completion.
9. Pass the frozen black-box tests against the final base-8 calculator, including precedence, parentheses, division,
   acceptance of digit 7, and rejection of digits greater than 7. All agent-authored unit and integration tests must
   also pass.
10. Trace the completed work to the initial and updated Charter revisions, final Design revision, reviewed Code
    revision, and completion decision.

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
