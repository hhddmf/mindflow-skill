# Runs and logs

## Where

- **Flow settings › Logs**: flow runs.
- **Logs panel** beside the canvas: flow runs and single-step tests (tagged *Flow* /
  *Step*), with *Load more*.
- Filters: status (**Success / Failure** — there is no "running" or "waiting" filter) and
  trigger type.
- Run history is kept for a limited time set by the organization (Organization › Audit
  TTL); 7 days is a common setting.
- **Analytics** (organization-wide usage) is visible to organization admins.

## Reading a run

The run's tabs: the incoming **HTTP Request**, **Error** (the chain of causes, e.g. "Failed
to execute flow › Failed to execute step [HTTP Request] › 401 - Unauthorized") and
**Variables**. Each step expands to Request / Response / Info. On the canvas, each step
shows a green or red count for the selected run (failed retries count separately). A
toast shows the outcome live, with **Stop** while it runs.

Tips:
- A run under *Continue execution* can **complete although a step failed**: read the
  step, not just the run status.
- A Condition's log lists every branch with the values it compared.
- Open runs from the Logs list; a direct link to a run may show "No version has been
  attached to this run".

## Replaying an input

Both create a **new manual run** on the **current draft** (a "latest" deployment
redeploys first); the original run is untouched. Use them to retry a failed input after a
fix without asking the caller to send it again.

- **Rerun icon** (Logs › open a run › top right): replays **that run's** input, at once.
- **Run button** (designer toolbar): when the flow has received input before, a dialog
  **"Run flow with previous input?"** names the latest input ("the last input of type
  WEBHOOK, recorded on …"). **Run** replays it; **Run without input** starts empty.

The replay is the whole recorded request (body, headers, query), so steps that read
headers see the original values. Only the trigger type changes, to manual. There is no
rerun from a single step.

## Stopping a run

Stop (in the toast or the Logs list) asks for confirmation; the run then fails with "The
execution was stopped by the user".

## Runs waiting for an approval

A step that puts a **Resume webhook URL** in its output (e.g. in an email) pauses the run
until someone opens the link.
- In the Logs list it looks like any running run: spinner, frozen duration, Stop button.
  There is no "waiting" status.
- Opened from the Logs list, its detail stays empty until it ends. Started from the
  designer's Run button, the log panel follows it live and shows the step as waiting.
- It ends when someone answers, at the step's **resume timeout** (min 180 s; the exact
  moment can vary by a minute), or at the **flow timeout**, which counts the waiting
  time. Set the flow timeout well above the resume timeout. Waits of 2 days work; keep
  waits to days rather than weeks, because a platform update can interrupt a run that is
  still waiting.
- A timeout and a manual stop both show "The execution was stopped by the user" on the
  run; the paused step's own log says "Step resume timeout". To turn silence into a
  proper outcome, use the "no answer" branch in
  [patterns.md](patterns.md#approval-with-a-timeout).
- **The first answer wins.** Later clicks, or clicks after the run ended, still show
  "Thank you!" but change nothing.
- Anyone holding the link can answer: send it only to the approver.

## Notifications

Flow settings › **Email notifications**: none / on success / on any failure / both, with
recipients. Changes can take up to 15 minutes to apply.

## Rooms

A **Room** is a chat with Mindflow's AI assistant. A flow with a **Chathook** trigger
answers when mentioned with `@`. Rooms can also use a model chosen among those the
organization allows, ask for **approval before calling a tool**, and mention flows or
credentials of the current environment with `@`.
