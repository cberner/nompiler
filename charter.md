---
kind: charter
project: Nompiler
short_name: Nom
status: ready for bootstrap
purpose: Human-authored source of truth for the Nom project
---

# Charter: Nompiler / Nom

## 1. Project Summary

**Nompiler**, shortened to **Nom**, stands for **Neural Compiler**. It is a coding assistant workflow orchestrator. Its
purpose is to take high-level user intent and turn it into working software through a structured multi-agent process.

Nom should let a user describe a software project at a high level: project goals, features, expected user interactions,
constraints, and preferences. Nom then analyzes the Charter, researches relevant background information, designs the
architecture, generates tasks, implements code, reviews the results, debugs failures, and iterates until the project is
complete.

Nom is intended to behave like a neural compiler:

```text
Charter  ->  Design  ->  Code  ->  Release
human        generated   generated  stabilized
source       IR          software   output
```

The user should primarily interact with:

1. The **Nom UI**
2. The **Charter branch** (i.e. `master`)

Generated Code and Release branches, including their Design collateral, are mostly agent-owned. The user may inspect or
edit them for debugging, auditing, or advanced control, but normal users should not need to.

## 2. Core Goal and Design Rule

Nom should make it possible to build substantial software from high-level product intent without requiring the user to
micromanage implementation details.

The central design rule is:

> The Charter branch is the canonical human source. Design artifacts, Code branches, and Release branches are generated
> outputs of Nom.

The user is responsible for product intent and important product-level decisions. Nom is responsible for architecture,
planning, implementation, review, debugging, and refinement.

Generated Design and Code may influence future work, but they must not silently become the Charter. If Nom discovers
that the Charter is incomplete or wrong, it should ask the user or propose a Charter change.

This preserves the compiler-like structure of the system and makes self-hosting possible without letting old generated
artifacts pollute future generations.

## 3. Key Concepts

### 3.1 Repository and Branch Model

Nom uses a single Git repository with separate branch namespaces for Charter, Code, and Release. Design is a versioned
artifact stored with the Code lineage it governs rather than in a separate branch namespace.

Nom must reserve the `code/*` and `release/*` namespaces. A repository must not contain both a top-level branch such as
`code` and a namespaced branch such as `code/1.0`. Nom must stop and report a namespace collision rather than creating
or rewriting conflicting refs.

Manual changes on active Code branches, including changes to Design artifacts, are treated as ordinary generated state,
not protected human intent. Nom must detect them before integrating work so it does not act on a stale revision, then
continue as if Nom had made the changes itself. It may preserve, modify, or remove their content without asking the
user. Manual changes to the Charter remain human-owned input and follow the Charter reconciliation rules.

#### 3.1.1 Charter Branch

The Charter branch contains the human-authored source of truth for the project. It should be named `master`.

Nom must identify the immutable Charter revision used by each run and account for Charter changes observed while the run
is active.

#### 3.1.2 Code Branches

Code branches contain active software implementation lineages.

Each Code branch stores its canonical generated Design artifacts under `.nom/design/`. These artifacts are Nom's
intermediate representation and are primarily intended for Implementer agents, while remaining available for debugging,
auditing, and comparing revisions. Versioned Design collateral is not Nom runtime state; live execution state must be
stored separately from generated project branches.

Each run uses a selected Code branch or Code starting point. Its Design records the Charter revision and Code baseline
for the run. Nom identifies each published `.nom/design/` state as an immutable Design revision. Later
implementation-only Code revisions continue to reference that Design revision until the Architect publishes another.

Nom works directly on a selected Code branch. The user does not review a generated candidate branch to approve
changes. Instead, the user tells Nom to work from a Charter and the branch to target, and Nom autonomously writes,
tests, debugs, reviews, and refines that branch.

Example Code branches:

```text
code/1.x
code/2.x
```

A Code branch may continue from a previous Code branch, or it may be a clean generation from the Charter. Code branches
contain both the generated Design history and the implementation it governs.

#### 3.1.3 Release Branches

Release branches are stabilized outputs cut from Code branches. The user initiates a Release branch when they want to
preserve a stable version of the software. Nom then performs extra testing of the Release branch to fix bugs and ensure
stability.

The Nom UI must provide a button to cut a Release branch from a selected Code revision. Cutting the branch starts a
release-stabilization process in which Nom tests, fixes, and polishes the Release branch.

Example Release branches:

```text
release/1.0
release/1.1
release/2.0
```

Release branches are useful when the Code branch continues evolving but a stable version needs to be preserved.
They retain `.nom/design/` for traceability, although packaged binaries and other distribution artifacts need not
include it.

### 3.2 Agent Roles and Artifact Authority

Nom has several roles for agents, further described in later sections:

- **Architect**: Creates and maintains the Design, answers questions from Implementers, incorporates findings, and makes
  the final completion decision. The Architect should use the strongest available coding model for research synthesis,
  architecture, task planning, and other judgment-heavy work.
- **Implementer**: Writes code and implements features based on the Charter and Design. Implementers may use capable but
  cheaper models for well-scoped implementation, tests, bug fixes, local refactors, and mechanical changes.
- **Reviewer**: Independently evaluates completed work and reports findings to the Architect and Implementer. The
  Reviewer should use the strongest available coding model for code, security, performance, and technical-debt review.

Feedback may move between roles without transferring authority over canonical artifacts. The Architect may ask the user
questions and propose Charter changes, but the Architect must never modify the canonical Charter. After the user
explicitly approves a proposed change, the Nom system may apply it to the Charter branch. The user may also change the
Charter directly.

Implementers and Reviewers may ask the Architect questions and report Design findings, but only the Architect may
publish substantive changes under `.nom/design/` within Nom's agent workflow. Nom may validate Design artifacts and add
mechanically verifiable run metadata. Nom must reject an Implementer or Reviewer result that modifies Architect-owned
Design paths. Reviewers report Code findings to the Architect for canonical disposition and to Implementers for any
resulting Code changes.

### 3.3 Charter

The Charter captures what the user wants, not how the software should be implemented.

It is free form and may include things like:

- Project purpose
- Target users
- Desired features
- Important user flows
- User-facing constraints
- Product-level decisions
- Non-goals
- Answers to clarification questions

The Charter is sacred. Agents may ask questions and propose changes, but no agent, including the Architect, may modify
the canonical Charter. The Nom system may apply a proposed change only after explicit user approval. The user may also
modify the Charter directly.

An active scoped contract may narrow the work required for a milestone, but it must not override the Charter's product
intent or durable rules. If normative inputs conflict without a clear interpretation, the Architect must stop and ask
the user rather than silently choosing one.

### 3.4 Architecture and Design Artifacts

Nom's Architect agent reads the Charter and produces generated architecture documents under `.nom/design/` on the
selected Code branch.

The canonical Design artifacts for a run must include, as appropriate to the project:

- A run manifest describing inputs, lineage, and Design revisions
- System architecture and major components
- Interfaces, APIs, and data model
- Key dependencies, security assumptions, and execution assumptions
- An implementation and integration strategy
- A testing and acceptance strategy
- A quality plan
- Structured implementation tasks with dependencies, acceptance criteria, and testing expectations
- A research summary when research materially informs the Design
- An Architect decision and revision log
- Canonical summaries of implementation and review findings that affect the Design
- A final completion assessment

The architecture is mainly for Nom's Implementer agents. The user can read it, but the system must not depend on
the user doing so.

### 3.5 Tasks

The Architect agent converts the generated architecture into implementation tasks and owns their canonical definitions
under `.nom/design/`.

Each task should include:

- Goal
- Relevant architecture context
- Expected behavior
- Acceptance criteria
- Testing expectations
- Dependencies, if any

Task definitions are versioned with the Design on Code branches. Live execution state must be maintained separately from
the canonical task definitions so it can change without rewriting the Design.

## 4. Workflow

Nom operates as a high-level reconciliation loop. Each run separately records:

- A **workflow phase** describing the work Nom is performing.
- A **run status** describing whether that work can currently progress. Nom must distinguish at least running, waiting
  for an agent, waiting for the user, reconciling an update, paused by the user, paused for token budget, paused for
  weekly quota, operationally blocked, failed, cancelled, and completed.

Correctable feedback changes the workflow phase without making the run operationally blocked. Any nonterminal phase may
be paused, cancelled, interrupted by a Charter update, or affected by an operational failure. Nom preserves its phase
and immutable inputs so it can reconcile and resume safely.

### 4.1 Charter Intake

This is the initial workflow phase.

Allowed transitions:

- If intake succeeds, the run moves to Research and Architecture. Minor ambiguities do not prevent this transition;
  Nom records and displays non-blocking clarification questions while continuing work that is unaffected by them.
- If an unanswered product question prevents safe progress, the phase remains Charter Intake and the run status becomes
  waiting for the user as described in Charter Clarification and User Decisions.

The user writes or updates the Charter branch, or provides input through the Nom UI. Nom reads the Charter and
identifies important ambiguities.

Nom asks the user clarification questions only when the answer materially affects product behavior, user experience, or
major constraints.

The Architect may draft questions or proposed Charter changes, but it must not modify the Charter branch. After explicit
user approval, the Nom system writes the approved answer or change to the Charter branch.

The user may also commit Charter changes directly while a Nom run is active. Nom durably detects committed Charter
revisions and notifies active agents at the next safe turn boundary. The run continues, but stale work is not integrated
until the Architect has reconciled the new Charter into a new Design revision.

Charter changes are not merged or rebased into Code branches. The Architect compiles them into Design revisions.
Implementation work may synchronize only with the current Code revision after the active Design has reconciled the
Charter change.

If the Charter changes after a run completes, the completed run remains an immutable record of its exact inputs. Nom
offers the user a successor run with the previous final Code revision as its baseline. The user may select a new or
existing Code branch for that run. Nom must not start the successor run without the user's approval.

### 4.2 Charter Clarification and User Decisions

The Architect asks the user only about product decisions or safety-sensitive policy changes and identifies the work the
answer blocks. Unaffected work continues. If no safe work remains, the run waits for the user rather than becoming
operationally blocked. After an answer through the UI or a relevant Charter revision, Nom records it, reconciles Charter
Intake, and resumes the appropriate phase.

### 4.3 Research and Architecture

Allowed transitions:

- If the architecture is ready, the run moves to Task Planning.
- If a product-level answer is required, Nom follows Charter Clarification and User Decisions while continuing any work
  not blocked by that answer.
- A Design-level finding from a later phase returns the run here with its evidence and immutable inputs.

Nom researches relevant prior art, libraries, papers, design patterns, and comparable products when useful.

The Architect agent then reads the Charter and produces generated architecture artifacts under `.nom/design/` on the
selected Code lineage. The Architect should resolve technical details itself when reasonable. It escalates to the user
only for product-level decisions.

Architecture is a continuing reconciliation loop, not a one-time generation step. The Architect revises the Design when
the Charter changes or implementation and review reveal new information.

### 4.4 Task Planning

Allowed transitions:

- When the Design and task plan are ready, their candidate moves to Candidate Validation before publication and before
  implementation begins.
- If planning reveals a Design issue, the run returns to Research and Architecture.

The Architect converts the architecture into implementation tasks and revises them as the Design evolves.

Tasks should be as independent as practical so Nom can execute them in parallel when appropriate.

### 4.5 Implementation

Allowed transitions:

- When an Implementer produces a task candidate and its required tests have run, the candidate moves to Candidate
  Validation. Producing a candidate does not complete the task.
- If the Implementer reports a Design-level finding or any other failure it cannot correct, the run returns to Research
  and Architecture with its report and evidence for Architect triage.
- A task becomes complete only after its exact approved candidate passes Controlled Integration. When all planned tasks
  are complete, the run moves to Iteration and Refinement.

Nom gives each active agent an isolated writable workspace based on an immutable Code revision. No two active agents may
share a writable checkout. Nom must retain enough durable state to recover the workspace and attribute its results.

Architect-owned Design-only revisions may advance the externally meaningful Code branch after required Design
validation. Implementation changes may advance it only after the required validation and independent review. Before
publishing a Design revision, integrating implementation work, or completing a run, Nom must verify that the relevant
Charter, Design, and Code revisions are still current and reconciled. If an active Code branch changes manually, Nom
must reconcile the new revision before continuing and must not integrate work based on the stale revision. After
reconciliation, the manual content has no special preservation status.

If an Implementer agent hits ambiguity, it asks the Architect. The Architect answers, revises Design artifacts if
necessary, or escalates to the user through the Nom UI.

### 4.6 Candidate Validation

Nom validates every Design-and-task candidate before publishing it and every implementation candidate before sending it
to an independent Reviewer. Applicable validation includes artifact-ownership rules, required project checks,
repository hygiene and secret scanning, traceability, and candidate consistency. Validation that can execute
project-controlled code must follow the Test Isolation requirements.

A failure rejects the candidate and follows System Error Handling and Autonomous Recovery. Passing Design work may be
published and begin Implementation; the exact passing implementation candidate moves to Review.

### 4.7 Review

Allowed transitions:

- Approval of the exact validated candidate moves it to Controlled Integration.
- A correctable Code finding returns to Implementation.
- A Design-level finding returns to Research and Architecture.
- A stale candidate is rejected and reconciled from current immutable inputs.

An independent Reviewer evaluates validated candidate work against:

- The task
- Generated architecture artifacts
- Existing codebase conventions
- Tests
- Security expectations
- Correctness expectations

The Reviewer can recommend approval, request changes, or identify follow-up work. The Architect receives the findings,
writes the canonical review summary, and decides whether to accept the work, revise the Design, or create follow-up
tasks.

### 4.8 Controlled Integration

Nom integrates only the exact implementation candidate that passed Candidate Validation and independent Review. Before
advancing the selected Code branch, Nom revalidates repository safety, verifies the candidate is unchanged, and ensures
its Charter, Design, and Code inputs are current. Integration advances the branch atomically from its expected revision;
rejections follow System Error Handling and Autonomous Recovery.

After successful integration, Nom begins the next ready task in Implementation. When all planned tasks are integrated,
the run moves to Iteration and Refinement.

### 4.9 Iteration and Refinement

Allowed transitions:

- Findings that require Design changes return to Research and Architecture.
- Findings covered by the current Design become new or revised tasks through Task Planning and Implementation.
- When no unresolved blocking findings remain, the run moves to Completion Assessment.

Nom repeats the workflow phases above until the requested feature set is complete.

After initial completion, Nom runs refinement passes for:

- Tech debt
- Bugs
- Test coverage
- Security
- Performance
- Maintainability

Findings become additional tasks. Nom continues until the Architect determines that the current Design is completely
implemented and the Code has reached a good level of quality.

The Architect's completion judgment must identify the immutable Charter, Design, and Code revisions it assessed and be
supported by passing required checks, resolved blocking findings, and a written quality assessment. A failed, paused,
cancelled, or blocked run must not be reported as complete.

### 4.10 Completion Assessment

If the completion criteria above are not met, the findings return to the appropriate phase. Otherwise, the run becomes
completed. Completed, cancelled, and failed are terminal; a later Charter change may create a successor run but does not
alter the completed run.

### 4.11 Pause, Operational Block, Failure, and Resume

The user may pause or cancel a run at any time. Pausing takes effect at a safe boundary and preserves durable state;
resuming reconciles that state before continuing. Cancellation is terminal and must never be reported as completion.

Token-budget exhaustion may occur from any non-terminal workflow phase. It pauses the run at a safe agent-turn boundary
and prevents Nom from scheduling another agent turn until the user authorizes additional budget. Weekly-quota safety
pauses behave likewise and may resume only when the safety condition clears. Nom must report either pause without
scheduling an over-budget agent turn.

Operationally blocked is reserved for an external capability that Nom cannot provide or safely substitute after
autonomous recovery and that requires external action. Failed is reserved for proven impossibility under the immutable
Charter and safety policy, or irrecoverable corruption that prevents safe reconstruction. Failed checks, unavailable
tools or dependencies, unresolved findings, and agent-reported blocked or failed outcomes are not by themselves
terminal conditions. The UI must clearly indicate the condition and required user action.

## 5. System Architecture

### 5.1 System Error Handling and Autonomous Recovery

Every safeguard and gate must fail closed without automatically blocking the surrounding run. When an agent's work or
decision causes a correctable failure, Nom records and returns the evidence to that agent so it can correct the failure
and continue. The workflow governs any resulting transition without transferring artifact authority.

Agent findings, classifications, and outcomes are evidence, not authoritative workflow transitions. Any failure that
cannot be corrected in the current workflow returns to the Architect with its report and evidence, regardless of its
classification; a well-formed agent result is never an invariant violation. The Architect selects the next workflow
transition under this Charter. Nom must not treat exhausted retries as a failure or repeat an equivalent action without
changed inputs; when no different recovery path remains, it returns the evidence to the Architect.

Nom automatically repairs and retries operational failures when it can do so safely. Equivalent failures must not loop
silently, but legitimate architecture, implementation, and quality refinement have no fixed iteration limit. A run
becomes operationally blocked or failed only under the definitions in Pause, Operational Block, Failure, and Resume.

Only the Architect may ask the user for a product decision or propose a Charter change; other agents route such matters
through it. Nom may surface operational actions that only the user can perform, such as adding budget or credentials.

### 5.2 Durable State and Recovery

Nom must store enough operational state durably, separately from generated project branches, to recover active work,
pending input, workflow phase and run status, recovery attempts, policy revisions, validation results, token-budget usage,
reported quota status, and completion status. On startup, Nom reconciles that state with the repository and agent
workspaces before scheduling more work. Retrying an operation must not duplicate an integrated change, lose a pending
Charter update, or convert interrupted work into accepted work.

### 5.3 Run Budgets and Usage Safety

The user must provide a token budget when starting a run. Nom must track the Codex token usage reported for that run and
stop scheduling new agent turns when the budget is exhausted. Work stops at a safe agent-turn boundary, and budget
exhaustion must never cause incomplete work to be reported as complete.

Whenever Codex reports the remaining weekly usage quota, Nom must stop scheduling new agent turns if the reported value
is below 15 percent. The run pauses at a safe boundary without losing work and may resume only after the safety
condition no longer applies. Nom must show the run's token budget, reported usage, and weekly quota status in the UI.

### 5.4 Security and Safety Model

Nom is designed to run on a dedicated machine, without valuable user data, and it should prioritize autonomous
execution and delivering high-quality results, over strict safety guarantees.

It should make a best effort attempt to prevent actions which create material risk to the user, host, repository,
credentials, external systems, privacy, or unexpected cost. Nom must deny agent actions that are outside the effective
policy or create such material risk. It must durably record every disposition and surface denials through the Nom UI;
approval requests should not be surfaced to the user.

Nom's main security approach is to rely on the Codex built-in sandboxing, and to execute all generated code in a Podman
container. However, the latter is more to protect the Nom itself and the host from unintended consequences, rather
than for a hard security boundary. Nom must permit task-required actions that the effective policy allows within those
Podman containers.

### 5.5 Execution

Before agents work, Nom gives them the effective execution environment and repository policy. The Design records that
policy, and changes are reconciled like other stale inputs. The Architect may make durable exceptions to the default
policy within the Charter's safety requirements; weakening those requirements needs explicit user approval.

Nom must make capabilities permitted by the effective policy available in the managed execution environment.
Within the execution environment, agents may autonomously install packages and toolchains, download public dependencies,
build or modify container images, configure caches and registries, and otherwise repair their execution environment as
needed to complete the project. Routine environment changes must not require user configuration or approval.

Nom should provide sensible defaults. Missing or unsuitable tools or inputs trigger autonomous recovery rather than by
themselves blocking or failing a run. Reproducibility should be captured in the generated project through lockfiles,
configuration, scripts, or Design records where practical; it must not prevent autonomous progress.

Nom and its agents must not fabricate or misrepresent project inputs, evidence, or results to satisfy a gate. Third-party
inputs must be authentic and verified with available integrity mechanisms. Intentional substitutions are normal Design
or Code changes subject to applicable validation and independent review.

### 5.6 Configuration

Nom should expose the following configuration options to the user as command-line flags:

- Port number (default 7777) for the local web UI
- Coding model to use (default GPT-5.6-Sol)
- Architect reasoning level (default extra-high)
- Implementer reasoning level (default medium)
- Reviewer/QA reasoning level (default high)

Additional configuration options should only be added if strictly necessary. The user MUST NOT be required to configure
minor environment details such as container images, registries, toolchains, or scanner paths. Nom must provide its own
operational tools, including the default secret scanner, and the managed execution environment for agents and projects.
These must use pinned or immutable versions and must not depend on tools being present on the host `PATH`.

Nom may require a user-provided value only when the Charter or an approved policy says the user will provide it. Work that
does not require a missing value continues.

### 5.7 Repository Hygiene

Implementation agents may autonomously modify files and create local commits, but Nom must prevent unsafe or irrelevant
content from entering durable generated branches or being pushed to a remote. This includes:

- Oversized files
- Forbidden file types
- Build artifacts
- Dependency caches
- Generated binaries
- Large logs
- Secrets
- Accidental datasets or archives

This preserves autonomous execution while preventing one bad agent action from polluting durable Git history.

Full agent logs, model traces, operational state, isolated workspace internals, build artifacts, test-output archives,
dependency caches, and other large execution artifacts must stay outside project branches. Design artifacts may
reference externally stored evidence when needed.

Remote pushing is disabled until the user enables a specific remote for the repository. Once enabled, Nom may push its
generated Code and Release branches after they pass the repository-safety gate. Pushing the human-owned Charter branch
requires separate explicit enablement.

Nom applies its repository-safety gate before publishing a Design revision, before independent Review of an
implementation candidate, and again before integrating the exact reviewed candidate. The gate inspects every Git object
the candidate would make newly reachable from its expected base. The default limits are:

- 2 MiB for any single blob.
- 10 MiB for the aggregate size of all newly introduced objects.

A durable repository policy may change these limits or allow required source assets. Binary files are not forbidden
merely because they are binary; irrelevant generated artifacts, caches, logs, databases, and archives remain forbidden
unless the policy explicitly permits them.

Nom scans the same content for secrets with TruffleHog by default; a durable policy may select an equivalent maintained
scanner. Verified secrets and other high-confidence findings reject the candidate unless a durable allowlist identifies
a known false positive. Nom must not bypass its scanner or a remote provider's secret protection.

Before a push, Nom applies the same checks to every object the push would make newly reachable from the remote. New
submodules, repository-escaping or absolute symlinks, and unavailable Git LFS objects reject a candidate or push unless
the policy explicitly permits them. Nom must fetch remote state and push only the intended branch when its expected head
is current. By default, it must not force-push, delete remote references, use wildcard refspecs, or bypass branch
protection. A rejected push and its reason must be visible to the user.

### 5.8 Traceability and Stale-Work Safety

Every stage must operate from identifiable, immutable revisions rather than moving branch names. Nom must retain a
durable, inspectable association between:

- Each Design revision and the Charter revision, policy, and Code baseline it considered.
- Each implementation, validation, review, and integration result and the Design, task, candidate, and Code baseline it
  assessed.
- Each completion decision and the exact Charter, final Design, and reviewed Code revisions it assessed.

Before accepting work or reporting completion, Nom must detect whether relevant inputs or branches changed and ensure
that all observed Charter changes have been reconciled. Changes to active generated branches become new Nom-owned input;
Nom may later preserve, modify, or remove their content. It must not unknowingly integrate a stale result or publish a
partially completed transition. Traceability and completion state must remain recoverable after interruption.

The Design may choose how to represent this metadata, store operational records, and safely publish related branch
updates.

### 5.9 Test Isolation

Nom must run project tests in Podman containers so test behavior cannot modify the real repository or its worktrees.
Tests must operate on disposable repository copies and container-local writable storage. The real repository may be
provided as read-only input, but a test must not receive a writable bind mount or another capability that would let it
change the repository, its Git metadata, or externally meaningful branches.

This requirement applies to all tests Nom runs while developing generated software.
Builds, static analysis, and other validation that can execute project-controlled code must use the same
isolation. The Design chooses the container topology and artifact-transfer mechanism, but isolated commands must not
silently fall back to execution against the host repository when Podman is unavailable.

## 6. Self-Hosting Nom

Nom should be able to build itself.

Self-hosting means a Nom version produced through this workflow can process a later Charter change through a new
Architect-generated Design revision and a reviewed Code change without the user authoring Design artifacts or directly
prompting implementation agents.

All of the requirements for developing a project apply to Nom's own development, so its tests and acceptance suite must be sandboxed...etc.

A new Nom version can either continue from the previous Code branch or create a clean new Code branch from the Charter.

For example:

```text
code/2.x may branch from code/1.x
```

or:

```text
code/2.x may be a fresh cleanroom generation from the Charter
```

The lineage is not assumed. It is recorded in the Design run manifest.

## 7. User Interaction Model

The normal user should not need to understand Design collateral, Code branches, or Release branches.

The user interacts with Nom through:

1. The Charter branch
2. The Nom UI

See [ui.md](./ui.md) for details on the Nom UI.

The user may inspect Design collateral, Code branches, or Release branches when they want to debug, audit, or understand
what Nom is doing. These generated artifacts are not the main user interface.

## 8. Implementation Details

- Nom must be written in Rust and built as a single binary named `nom`.
- Nom is dual-licensed under MIT OR Apache-2.0. Generated Code and Release branches must retain the `LICENSE-MIT` and
  `LICENSE-APACHE` files, and generated dependency manifests must declare the license as `MIT OR Apache-2.0`.
