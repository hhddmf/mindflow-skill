# Recipes

Each recipe below has been built in the Studio and run end to end. Adapt names, URLs and
fields to the user's case.

## Sync API (the caller waits for the answer)

Webhook (`run-sync-qp`) → work → a **Webhook response on every branch** (status + JSON
body from pills). Flow timeout ≤ 25 s. Inside, a For each with **Max concurrency** above 1
keeps fan-out within the budget. After editing, send one async call to redeploy before
calling it synchronously. For a chat API: AI Task with memory **Advanced**, Memory ID =
the caller's `sessionId`.

## Approval with a timeout

1. In the step that asks for approval (e.g. Send email), insert **FLOW › Resume › Resume
   webhook URL** in the message. The step will pause after running; the canvas draws a
   dashed link and a pause badge.
2. The link accepts a **GET with query parameters**, so the email can carry plain links:
   `<link>?decision=approve` and `<link>?decision=reject`. The approver sees
   "Thank you!".
3. After the paused step, pick its resume data: it appears in the picker as an HTTP
   request (method, url, headers, **query**, body). Use `query › decision`.
4. Step Settings: **Resume timeout** (min 180 s) and **On failure = Continue execution**.
   Set the flow timeout well above the resume timeout, since it counts the waiting time
   (e.g. 600 s for 180 s).
5. On the `decision` pill, turn **Required off** in every branch: an unanswered request
   then reads as empty instead of failing.
6. Condition branches, in this order (the first true one wins):
   - **Approved**: decision *Is equal to* `approve` → continue / Success;
   - **No answer**: decision *Is empty* → a Failure step with a clear message, or a
     reminder;
   - **Rejected**: decision *Is not equal to* **`approve`** → Failure. (Compare with
     `approve`, not `reject`: this branch catches `reject` and any unexpected value.)

   The empty test must come before the "not equal" test, or rejections swallow
   timeouts. Don't use a bare ELSE here: it can't target a step that already has a link.

Good to know:
- The first answer wins; later clicks change nothing but still show "Thank you!".
- Anyone holding the link can answer: send it only to the approver.
- The timeout may fire up to a minute off the set value.
- **How long can a run wait?** A wait of **2 days** works: set both the resume timeout
  and the flow timeout in seconds (2 days = 172,800 s; give the flow timeout some margin
  above it). **Keep waits short (days, not weeks):** a platform update can interrupt a run
  that is still waiting.
  For longer approvals, don't hold a run open: store the request (e.g. in Artifacts, with
  a retention longer than the wait), let the approval link start a second flow, and run
  a daily Scheduler flow that reminds or escalates requests still unanswered.
- To test the "no answer" path quickly, use the minimum (180 s), then set the real value.
- *Continue execution* also covers the send itself: if the email fails, the run lands
  on "No answer". Check the send if that matters.
- A mocked approval step doesn't pause, so test the waiting with a real send.

## Polling until a job is done

Webhook → start the job → **Loop › Do while** [Wait N s → check status → **Transform data**
`jobStatus` = the status] with **Maximum iterations** as a guard → Condition on
`jobStatus` → Success / Failure. The Loop condition tests **VARIABLES › jobStatus** (it
can't see steps inside the loop). Add a branch for "still running after the last
iteration".

## Pagination (follow a "next" link)

1. **Init** (Transform data): `nextUrl` = the first page URL, `acc` = `[]`, `pages` = `0`.
2. **Loop › Do while** `nextUrl` **Is not empty**, with a Maximum iterations guard.
3. In the loop: **Fetch page** (HTTP GET on the `nextUrl` pill) → **Merge page** (Run
   script with input `{"acc": <acc>, "pages": <pages>, "page": <Fetch page body>}`, pills
   without quotes, returning `{acc, pages, next}`) → **Advance** (Transform data
   reassigning `nextUrl`, `acc`, `pages` from the script's `result`).
4. After the loop, read VARIABLES `acc` and `pages`.

For offset paging (`startIndex` / `count`), keep an offset variable instead of `nextUrl`.
If the API answers **404 instead of an empty page**: put Fetch page on *Continue*, pass
its `code` (Required off) to the script, treat 404 as "no more pages", and throw on any
other error so real failures still fail the run.

## Calling an unreliable API with a fallback

Primary call with **Timeout**, **Retry attempts** and **On failure = Continue execution** →
Condition on its `code` (Required off: a timeout has no code) *Is equal to* `200` →
continue; otherwise → fallback call (*Stop* on failure, so a double failure fails the
run) → alert (Send email) → Success. Add Flow settings › Email notifications on failure
as a backstop.

## Processing a list in parallel (For each)

Set **Max concurrency** (default 1). Inner call on *Continue*, so one bad item doesn't
stop the others. Iteration output `{url, code}` (code with Required off). After the loop,
a **Run script** merges `{"items": <For each output>}` into
`{total, okCount, failedCount, partial, failed[]}`; branch or alert on `partial`. The
slowest item sets the loop's duration. **Rate limits:** each item
costs roughly 1–2 s even for a fast call, so with N items in parallel you send about
N ÷ (time per item) requests per second. Start with a modest Max concurrency, watch the
logs for 429 (Too Many Requests), and adjust. Keep Retry attempts low, since each retry
is another request. Only for very strict limits (a few calls per minute) use concurrency
1 plus a Wait inside the loop; Wait counts in whole seconds.

## Business hours (route by the local time)

Goal: during office hours notify a person, otherwise page on-call or queue it.
- **Run script** (Node.js 22), no input needed:
  ```js
  const parts = new Intl.DateTimeFormat('en-GB', {timeZone: 'Europe/Paris',
    weekday: 'short', hour: '2-digit', hour12: false}).formatToParts(new Date());
  const get = t => (parts.find(p => p.type === t) || {}).value;
  const day = get('weekday'), hour = parseInt(get('hour'), 10) % 24;
  return { day, hour,
    businessHours: (['Mon','Tue','Wed','Thu','Fri'].includes(day) && hour >= 9 && hour < 18) ? 'yes' : 'no' };
  ```
  Then a Condition on `result › businessHours` *Is equal to* `yes`, with an ELSE IF for
  `no`. The time zone name handles summer and winter time. Add a public holiday list
  in the script if you need one.
- **Without a script**, to display or log the local time: fx **Translate DateTime
  Format** on an ISO date (for example *Triggered at*), source time zone `UTC`, target
  `Europe/Paris`, output format `EEEE HH:mm` → `Sunday 23:01`. For routing, the script is
  simpler than comparing formatted text.

## Splitting work across flows

Parent: For each input → **Call a flow** (sub-flow with a Flowhook trigger, *Wait for
completion*, On failure *Continue*) → Transform data copying `output.data.<lowercased
key>` into a variable (needed for the iteration output) → Run script to aggregate →
Condition → call a second sub-flow only when needed. Turn Required off on sub-flow
outputs, so a failed sub-flow becomes a "lookup failed" line rather than a failed parent.

## APIs without an integration

**GraphQL** — HTTP Request, POST, Body › JSON:
`{"query": "query($c: ID!) { … }", "variables": {"c": "<pill>"}}`. Pass values through
`variables` rather than inside the query text. Errors come back as **HTTP 200 with an
`errors` array**, which retries and On failure don't see: branch on `body.errors` *Is
not empty* → Failure (Required off, since `errors` is absent on success).

**SOAP / XML** — HTTP Request, POST, Body › XML with the full envelope and a pill for the
value. Add header `Content-Type: text/xml; charset=utf-8` (SOAP 1.1; the default
`application/xml` is often refused with 415) or `application/soap+xml` (SOAP 1.2); some
services also need `SOAPAction`. The response arrives as text: fx **XML to JSON**, then
**JSON Path** with brackets for prefixed keys (`$..['m:sCapitalCity']`, then `$[0]` if
the prefix may vary).

## Storing data (Artifacts)

There's no Artifacts page: use the **Mindflow Platform Artifacts** steps (search
"Artifacts" in the step picker; no credential needed). Each environment has its own
store.
- **Create or overwrite an artifact**: Key; JSON content (⋯ › *Additional properties*,
  then name/value rows); Write mode create-or-replace / create-only / replace-only;
  **Data retention defaults to 1 day** — set it on every write that must last. The
  choices are 1, 2 or 3 days, 1 week, 1, 3 or 6 months, 1 year, or no expiration
  (pick the next one up, e.g. 1 month to remember something for 10 days).
- **List artifacts** (limit, prefix). ⚠️ Typing a `/` at the **end** of the Prefix field
  opens the pill menu and the `/` is dropped: `customers/` is sent as `customers` and
  also matches `customersettings`. Close the menu with Escape and re-check the field,
  or filter the keys afterwards.
- **Get artifact content** (returns the JSON), **Get artifact metadata**, **Delete an
  artifact**.
- **A missing key fails the Get step with 404**: for "read, or create if missing", put the
  Get on *Continue* and branch on `code`.
- To see what's stored, keep a small admin flow with a List step and test that step.
