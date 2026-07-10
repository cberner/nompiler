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

## Index page

The landing page should have a navigation bar with links to all the top-level pages of the Nom UI.

At the top of the page is a mini dashboard of metrics of the most recently active Code branch. It shows a gauge of
completion percentage indicating how much of the charter has been implemented and tested. It also shows a metric of
the code quality, and a metric of the number of open issues.

There is a button to launch a new run from the current Charter. It opens a modal to select the name for the run, which Nom
uses to name Design and Code branches as `design/<name>` and `code/<name>`. An "Advanced" toggle can be used to manually
specify the Design and Code branches to use. An existing branch may be selected in this case.

It should show a list of runs, with a filter to show only active runs, or include completed and/or failed runs.

It should show a list of Design and Code branches, sorted by the most recently modified at the top.

## Run page

Click a run to see its details leads to the Run page, which shows the status of the run, including the current phase,
task, and agent status, as well as the token usage and remaining run budget.

It also shows the current Charter, Design, and Code revisions.

It also displays any feedback from the agents to the user and allows the user to review and approve any proposed change
to the canonical Charter.

## Code page

Click a Code branch on the landing page to see its completion, code quality...etc metrics on a dedicated page.

## Design page

Click a Design branch on the landing page to see its completion, churn (amount of feedback from the Implementer and other agents that we addressed, or remains open)