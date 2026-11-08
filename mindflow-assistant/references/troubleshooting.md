# Troubleshooting by symptom

Find the user's symptom, confirm the likely cause with them (error text, run log, a
screenshot of the step), then give the fix. Causes are listed most likely first.

## Triggers and calls

### The webhook answers 200 "Stay tuned…" but no run appears
The webhook answers 200 to every call, including calls it rejects. Rejected calls are
dropped silently.
- **API key missing or misplaced.** If the webhook has a key (trigger › Auth tab), send it
  as header `x-api-key`, or as query `?apikey=` (lowercase, exact spelling; `apiKey`,
  `api_key`, `key` or `token` are ignored).
- **Wrong URL.** Copy it again from the trigger; each deployment has its own URLs.
- **The caller's IP isn't allowed** (webhook Settings › Advanced › Allowed IPs, or the
  organization's Allowed IPs).

Tell the user to judge success by the run list, never by the HTTP code.

### A sync call returns 400 "took too much time or no Webhook response"
- **The flow timeout is above 25 s** (Flow settings › Timeout). Sync flows must stay ≤ 25 s.
- **The first call after an edit** spends its time redeploying the flow. Send one
  async call first, then the sync one.
- **A branch ends without a Webhook response step.** Every path needs one.
- **The flow failed** before answering: open the run in Logs.

### "can only be used when the flow is triggered with a synchronous webhook call"
The flow has a Webhook response step but was called on an async URL. Use the
`run-sync-qp` URL.

### A Scheduler, Formhook, Variables tab or Credential Lookup option is missing
These are switched on per Mindflow platform deployment. If the entry is absent, the
feature is off for this organization; the user's Mindflow admin or contact can say.

## Network

### A step to an internal tool fails with `Timeout of …ms exceeded`, `ENOTFOUND`, `ECONNREFUSED` or a certificate error
The tool isn't reachable from Mindflow's cloud (private IP, internal DNS name, firewall)
or uses an internal certificate. It needs either a public address restricted to
Mindflow's outbound IPs, or Mindflow's Relay Agent (a Cloudflare Tunnel inside the
company network). See [private-endpoints.md](private-endpoints.md).

## Data and pills

### "Data referenced by X was not found" (or "Data resume referenced by X…")
- **The data is legitimately absent** on this path (a failed or timed-out step has no
  `code`, GraphQL success has no `errors`, nobody answered an approval). Open the pill's
  popover and turn **Required** off: it then resolves to an empty value.
- **The pill was picked from a failed run.** A step that failed with *Stop* stores an
  error object, so the picked path became `data.code` instead of `code`. Re-pick it from
  a successful run.
- **The step is inside a Loop or For each** and is read from outside or from the Loop's
  own condition. See "A loop never ends" below.

### "Failed to evaluate function JSON Path … Cannot read properties of undefined (reading 'startsWith')"
The JSON Path matched nothing and its **Wrap** option is off. Fix the path. If the field can be absent, the simplest fix is JSONata with a default
(`$exists(a.b) ? a.b : ''`). With JSON Path, turn Wrap on (no match then gives `[]`) and
test for `[]` before extracting anything with `$[0]`. See [missing-null-and-large-data.md](missing-null-and-large-data.md).

### My "Is empty" test doesn't catch a null, 0, false, `[]` or `undefined`
*Is empty* only matches missing keys and empty strings. `null` arrives as the text
`null`, a JSONata with no match as `undefined`, and `0`, `false`, `[]`, `{}` as written.
Test them with *Is equal to*, or join several tests with Or. See
[missing-null-and-large-data.md](missing-null-and-large-data.md).

### The fx preview says "This function was skipped … input data exceeds the preview limit"
The input is too large for the preview. Only JSONata does this; real runs still work. While designing, put an **Array slice** (e.g. 0 to 100) before the function,
build the query, then remove the slice. See
[missing-null-and-large-data.md](missing-null-and-large-data.md#large-data-hundreds-of-kb-and-more).

### A pill shows greyed with "Failed to evaluate pill"
It reads a variable that doesn't exist in this environment (often an environment
variable that was never created in the target environment). Create the variable;
no redeploy is needed.

### A value I typed (or a pill I inserted) disappeared
The Studio saves fields shortly after typing. Moving on too fast, or a stale page, can
lose a value without warning. Re-open the step and check each field; reload the page if
the designer has been open through several deploys, then re-enter it.

### JSON Path gives `"\"CRITICAL\""` or `["HIGH"]` instead of `CRITICAL` / `HIGH`
New fx functions start with **Wrap** and **Quotes** on: turn both off. A recursive path
(`$..x`) always returns a list: add a second JSON Path `$[0]`.

### A number or list arrives as text
For each iteration outputs and many step outputs are strings (`"100"`). Convert in a Run
script, or compare as text.

### An AI Task prompt is cut short
Remove angle brackets (`<`, `>`) from the prompt; write the text without them.

## Credentials and environments

### It works in one environment and fails in another
Check, in this order:
1. **Credentials are mapped per deployment.** After deploying to a new environment,
   open each step that uses a credential and pick that environment's credential.
2. **Environment variables are per environment.** Create the same keys in the target.
3. **Credential Lookup only works on a deployment that follows "latest".** A deployment
   pinned to a named version runs the step with no credential: map one on the
   deployment instead.
4. The feature may be switched off on that platform deployment (see above).

### The real vendor answered 401, or data went to the vendor's public address
The step had **no credential** (unmapped after a deploy, or a Credential Lookup whose
name came out empty). Mindflow then calls the connector's default URL without auth.
Map the credential, or define the variable the lookup reads, **before** the first run in
a new environment; add a Condition that stops if the looked-up name is empty.

### Credential Lookup: "No credential named …", or the name comes out empty
- The name must match a credential **of the same service** in the run's environment (or
  Global), and only one.
- **A variable used only by the lookup isn't loaded.** Also reference it in an ordinary
  step (a Transform data that copies it is enough).
- The lookup can read constants, environment variables, flow variables defined in Flow
  settings, and the trigger input — not step outputs, including Transform data
  variables. For a payload field, put the trigger pill directly in the lookup name.

### OAuth 2.0: "Error while getting the token from the auth server › invalid_client"
Wrong client id or secret of an **OAuth 2.0** credential (this message only comes from
OAuth). Creating a credential doesn't test it, so mistakes show up at the first run.
Edit the credential and retype the secret. With Basic, API key or Bearer credentials, a
wrong secret shows as the API's own 401 / 403 in the step's Response.

### I need to stop a deployment from running while I fix it
Click the `ENVIRONMENT — version` badge in the breadcrumb and use the undeploy icon on
that deployment (deploy it again afterwards; its webhook URLs may change, so re-check
them). For a flow nobody calls yet, simply don't share the URL. A Scheduler fires on its
own, so undeploy flows that have one.

### My request body was replaced
An **HTTP Body Secret** credential replaces the whole step body with its own value. Use
it only for login-style endpoints; otherwise pick another credential type.

### Digest authentication fails with 401
Digest has failed in testing against a server that accepts the same user and password
from other clients. Test against the real target before relying on it, or use another
auth type if the API allows it.

## Versions and deployments

### My edit went live without deploying / didn't go live at all
- A deployment on **latest** runs the saved draft: every edit reaches it at its next
  trigger (it redeploys first).
- A deployment pinned to a **named version** is frozen. Create a new version, then use
  **Update version** on that deployment.

### After "Restore this version", runs fail or emails stopped
Restore brings back the steps but **not the credentials**: the deployment keeps the
credentials it has now, which may not suit the restored steps (e.g. a credential chosen
later for a different URL or auth type gives a 401). It also resets the flow's email
notifications. Re-pick each step's credential, re-set Flow settings › notifications,
and reload the page before editing again.

### The canvas is hatched and nothing can be edited
The flow is **Locked** (toolbar): click it to unlock. A deployment pinned to a version
always opens locked; edit "latest" instead, then publish a new version.

## Runs

### A run seems stuck (spinner, frozen duration)
Most often a step is **waiting for an approval** (a Resume webhook URL in its input).
The run list shows no "waiting" status. It ends when someone answers, when the step's
resume timeout passes, or when the flow timeout passes — whichever comes first. The user
can **Stop** it from Logs.

### "The execution was stopped by the user", but nobody stopped it
A **timeout** ended a run that was waiting for an approval: either the step's resume
timeout or the flow timeout (which counts the waiting time). Open the paused step's log:
it says "Step resume timeout". To handle silence as an outcome, see the "no answer"
branch in [patterns.md](patterns.md#approval-with-a-timeout).

### The approver clicked the link and saw "Thank you!", but nothing happened
The run had already ended (stopped or timed out), or someone had already answered: the
first answer wins, and later clicks still show "Thank you!".

### A run "completed" although a step failed
That step's **On failure** is *Continue execution*: the run goes on by design. Read the
step's status, or branch on its `code`.

### "No version has been attached to this run"
The run was opened from a direct link. Open it from the Logs list instead.

### A run that waits for approval shows an empty detail panel
Its detail loads only once it ends when opened from the Logs list. Started from the
designer's Run button, the log panel follows the run live.

## Building

### A loop never ends, or its condition can't see a step inside it
The Loop condition can only read the iteration index, outer steps and VARIABLES. Add a
**Transform data** step at the end of the loop body that writes the value to a variable,
and test that variable. Keep **Maximum iterations** as a safety stop.

### For each is slow
**Max concurrency** defaults to 1 (one item at a time), and each item costs roughly
1–2 s of overhead. Raise it; with a rate limit, raise it step by step and watch for 429
errors (see [patterns.md](patterns.md#processing-a-list-in-parallel-for-each)).

### A variable set inside a For each is empty after the loop
Variables set in a For each belong to their iteration (which also keeps parallel
iterations from overwriting each other). Add the value to the **Iteration output** and
read the For each output after the loop.

### One bad item stops the whole For each
Set the inner step's **On failure** to *Continue execution*, and output its `code` (with
Required off) in the iteration output to see which items failed.

### Call a flow doesn't list my sub-flow, or its outputs are empty
- The sub-flow needs a **Flowhook** trigger and a deployment **in the same environment**.
- Outputs are read at `output.data.<key>`, and keys are the **lowercased** output titles
  (`messageId` → `messageid`).
- If the sub-flow failed, the parent sees `null`: open the sub-flow's own run.

### ELSE can't point at the step I want
ELSE can't target a step that already has a link. Use an ELSE IF with the opposite test.

### A GraphQL call "succeeds" but the data is missing
GraphQL returns errors as HTTP 200 with an `errors` array, so retries and On failure
never trigger. Branch on `body.errors` **Is not empty** (Required off).

### A SOAP call returns 415
The XML body type sends `Content-Type: application/xml`. Add the header
`Content-Type: text/xml; charset=utf-8` (SOAP 1.1) or `application/soap+xml` (SOAP 1.2).

### A List artifacts prefix like `customers/` also returns `customersettings`
The trailing `/` was swallowed by the pill menu (typing `/` opens it), so the prefix sent
was `customers`. Re-type the prefix, press Escape to close the menu, and check the field
shows the slash. As a safety net, filter the returned keys before acting on them
(especially before a Delete).

### Send email fails on the sender
`from` is the **name** of a sender configured for the organization (for example
`mindflow`), not a free email address.

### An artifact read fails with 404, or a stored artifact vanished
A missing key fails the Get step (put it on Continue and branch on `code`). Artifacts
are kept **1 day** by default: set Data retention on every write that must last. Each
environment has its own store.

## The Studio itself

### "Failed to do Create step", buttons that don't respond, results that don't refresh
The page is stale. Reload the designer and try again. If a Create seemed to do nothing,
check the list before retrying: it may have created the flow anyway.

### AI Build only proposes HTTP or API steps, or keeps spinning
AI Build only creates HTTP Request and integration steps. If it hangs, build the steps
by hand.
