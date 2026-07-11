# Nom UI

This document defines durable user-facing requirements for the Nom UI.

The user launches Nom by executing `nom` for a Git repository. Running `nom` starts the local Nom UI, which should let
the user:

1. See the available Code branches and select one to continue, or select a Code starting point for a new lineage.
2. Start a Nom run with a required token budget.
3. See the current Charter, Design, and Code revisions, active phase, current task, agent status, token usage, remaining
   run budget, and reported weekly quota status.
4. See and answer product-level clarification questions.
5. Review and approve any proposed change to the canonical Charter before Nom commits it.
6. See when a direct Charter change is detected, when the Architect reconciles it, and when a successor run is offered.
7. See validation, review, and quality status.
8. Pause, resume, retry a blocked operation, or cancel the run.
9. See the Architect's completion decision and resulting Design and Code revisions.
10. Cut a Release branch from a selected Code revision and start its testing and polishing process.

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

Clicking a run opens the Run page, which shows its current phase, task, agent status, token usage, and remaining run
budget.

It also shows the current Charter, Design, and Code revisions.

It also displays any feedback from the agents to the user and allows the user to review and approve any proposed change
to the canonical Charter.

## Code Page

Clicking a Code branch on the landing page opens a dedicated page with its completion, code-quality, and related
metrics.

## Design Page

From a Code branch, the user can open its Design page to see the current Design revision and completion status, along
with Design findings that have been addressed or remain open.
