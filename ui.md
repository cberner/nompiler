# Nom UI

This document defines durable user-facing requirements for the Nom UI.

The user launches Nom by executing `nom` for a Git repository. Running `nom` starts the local Nom UI, which should let
the user:

1. See the available Code branches and select one to continue, or select a Code starting point for a new lineage.
2. Start a Nom run with a required token budget.
3. See the current Charter, Design, and Code revisions, workflow phase, run status, current task, agent status, token
   usage, remaining run budget, and reported weekly quota status.
4. See and answer product-level clarification questions.
5. Review and approve any proposed change to the canonical Charter before Nom commits it.
6. See when a direct Charter change is detected, when the Architect reconciles it, and when a successor run is offered.
7. See validation, review, integration, recovery, and quality status.
8. Pause, resume, retry an operationally blocked run when appropriate, add budget to a budget-paused run, or cancel the
   run.
9. See the Architect's completion decision and resulting Design and Code revisions.
10. Cut a Release branch from a selected Code revision and start its testing and polishing process.

## API

The Nom API is RESTful, used by the web UI, available to the user for programmatic interaction, and used by the
agent-facing CLI. It exposes the same data and actions as the UI while enforcing the same safety and approval boundaries.

## Agent CLI

Nom exposes the agent-facing CLI through `nom agent` in the same `nom` binary. It interacts with the Nom API within the
agent's run, role, and workspace authority and cannot bypass workflow gates.

## Page Design Rules

All the pages described below should be designed for human users. Data should be presented in a clear manner, and an
appropriate widget used. For example, the list of tasks on the run page should be rendered in a table/list like format,
with a toggle to collapse it. Coloring should be used appropriately to highlight important information, especially
warnings or errors.

Raw JSON or other raw data may be displayed, but only behind clearly marked "raw data" or "raw output" controls.

All information should be available without requiring the user to view the raw data.

Human-readable and raw views must redact credentials, secret values, and other protected data. The UI must not claim to
show private model reasoning or other information that the agent runtime did not provide.

All pages should share consistent CSS styling and a consistent design theme.

## Index Page

The landing page should have a navigation bar with links to all the top-level pages of the Nom UI.

At the top of the page is a mini dashboard of metrics of the most recently active Code branch. It shows a gauge of
completion percentage indicating how much of the charter has been implemented and tested. It also shows a metric of
the code quality, and a metric of the number of open issues.

There is a button to launch a new run from the current Charter. It opens a modal to select the run name, which Nom uses
to name the Code branch as `code/<name>`. Nom stores the run's canonical Design under `.nom/design/` on that branch. An
"Advanced" toggle allows the user to select an existing Code branch or another Code starting point. Nom must validate
and reconcile the selected lineage before starting the run.

It should show a list of runs, with a filter to show only active runs, or include completed and/or failed runs.

It should show a list of Code branches, including each branch's current Design status, sorted by most recent activity.

## Run Page

Clicking a run opens the Run page, which separately shows its current workflow phase and run status, as well as its task,
agent status, token usage, remaining run budget, and weekly quota status. Waiting-for-user, paused, operationally blocked,
failed, cancelled, and completed statuses must be visually distinct.

Clicking into an agent session opens the Session page.

It also shows the current Charter, Design, and Code revisions.

It also displays any feedback from the agents to the user and allows the user to review and approve any proposed change
to the canonical Charter.

The page shows validation, review, integration, correction, and recovery history. A current failure explains the evidence,
next responsible actor, and whether Nom is correcting it automatically or waiting for user action. Candidate rejections
stay in the automatic workflow; retry controls are for operationally blocked work.

## Session Page

The Session page renders the events Nom receives from the agent runtime in a chat-like format, including messages,
available reasoning summaries, tool calls and results, and status.

## Code Page

Clicking a Code branch on the landing page opens a dedicated page with its completion, code-quality, and related
metrics.

## Design Page

From a Code branch, the user can open its Design page to see the current Design revision and completion status, along
with Design findings that have been addressed or remain open. It also shows the repository policy considered by the
Design and the findings that caused it to be revised.
