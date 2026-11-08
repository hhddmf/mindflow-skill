# Triggers

A flow can have several triggers: right-click the empty canvas and pick another (the
ones already present are greyed out; one Scheduler at most). A new trigger links to the
same first step. To branch on how a run started, use **Built-in variables › Mindflow ›
Trigger type** (`WEBHOOK`, `SCHEDULE`, `MANUAL`…).

## Webhook

Its URLs are shown on the trigger (each deployment has its own, and they don't change
when the deployment's version changes):
- `…/run-async-qp` — asynchronous; API key in the query.
- `…/run-async` — asynchronous; API key in the header.
- `…/run-sync-qp` — synchronous: the caller waits for a **Webhook response** step.

**Protecting it** (trigger › Auth tab): no key means anyone with the URL can start the
flow. **Generate API key**, then send it as header **`x-api-key`** (works on either URL)
or as query **`?apikey=`** (lowercase, exact). Prefer the header: query strings end up in
logs. Settings › Advanced adds mTLS URLs and **Allowed IPs**; an organization admin can
also restrict callers for all webhooks (Organization › Allowed IPs).

**The webhook answers 200 "Stay tuned. Your playbook is going to be executed..." to every
call**, including calls it rejects for a wrong or missing key or a wrong URL. Callers can't
tell from the response: check the run list. (What a caller from a non-allowed IP
receives hasn't been checked here; judge by the run list too.)

**Synchronous calls**
- Flow settings › Timeout must be **≤ 25 s**.
- Put a Webhook response (status + JSON body from pills) at the end of **every** branch.
- The same flow called asynchronously fails ("can only be used when the flow is
  triggered with a synchronous webhook call").
- The first call after an edit redeploys a "latest" deployment, which can use up the
  25 s. Send one async call first after editing. A caller that times out gets HTTP 400
  "took too much time or no Webhook response".
- The flow keeps running after it has answered, but still within the 25 s.

A webhook sent while a deployment is redeploying is queued and runs afterwards.

## Scheduler

*At regular interval* (Daily / Weekly / Monthly / Yearly) or *Once*. Daily has an hour
grid and a minute grid, both multi-select: the schedule is every combination ("every day
at 12:48, 12:50, 13:48, 13:50"). There is no "every N minutes" option, but ticking several minutes gets close (e.g.
hours 8–17 × minutes 0, 15, 30, 45). Weekly lets you pick a day and time; whether it
offers the same multi-select grids hasn't been checked here — look at the panel. To
skip weekends with Daily, add a weekday check at the start of the flow. Times use the time zone shown on the trigger. The trigger outputs
`{date, type: "schedule"}`; its pills appear after a first scheduled run. The Scheduler
may be switched off on some platform deployments.

## Manual

The **Run** button in the designer, or *Run* from the flow card menu. See
[runs-and-logs.md](runs-and-logs.md#replaying-an-input) for running with the previous
input.

## Flowhook

Makes the flow callable by **Call a flow** in another flow, and usable as an AI agent.
Define its inputs (with types and required fields); they become the caller's parameters.

## Emailhook

The deployment gets an email address; each email starts a run with sender, subject,
text, HTML and SPF/DKIM results. Reply with Send email to `from.value[0].address`. An
organization admin can restrict which senders may trigger flows (Organization › Allowed
emails).

## Chathook

Lets the flow answer in an AI Room: mention it with `@` in the room. If the mention menu
doesn't appear, move the cursor after typing `@name`.

## Formhook

May appear on some platform deployments (a schema of form fields; each submission
starts a run). Don't plan on it unless the user sees it in their trigger list.
