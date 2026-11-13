# Canvas and steps

## Getting around the designer

- **Add a step**: hover the step it should follow and use its add button. To add a second
  trigger, right-click the empty canvas.
- **Open a step**: the gear that appears when hovering the step's right edge. Clicking a
  step's name renames it (select, click the name, type, Return).
- **Delete a step**: right-click its icon › Delete; the neighbours are re-linked.
- **Tidy up**: the magic wand (auto-layout) and zoom-to-fit in the bottom bar; **undo /
  redo** are there too. ✨ *Document workflow* names and describes steps for you (it
  overwrites your names).
- **Locked** flows show a hatched canvas: click Locked in the toolbar to unlock.
- **Flow settings** (gear in the toolbar): Timeout, Email notifications, Output (flow
  outputs), Logs. Also listed there: data retention, global variables, metadata.

## Data pills (the `/` picker)

Type `/` in any field:
- **Triggers**: the trigger's data (webhook `body › …`, `query`, headers; flowhook inputs).
  A flowhook offers a sample from its schema before any real run.
- **Steps**: choose a **past run** of the step, then the field. Pick from a
  **successful** run: a step that failed with *Stop* stores an error object, so the path
  you pick there (`data.code`) won't match a normal run (`code`).
- **VARIABLES**: Transform data outputs and flow variables.
- **FLOW**: For each item / index / count, Loop iteration index, Resume webhook URL.
- **Built-in variables › Mindflow**: Trigger type, Triggered at (UTC), Flow run ID, Flow
  name, Environment name, User email, Parent flow…
- Environment variables appear by name.

**Required** (pill popover): on by default for step data, off for variables. When on,
a **missing** key fails the run (a key present with `null` passes, as the text `null`);
turn it off when the data may legitimately be absent (a failed step's `code`, GraphQL
`errors`, an unanswered approval, a failed sub-flow). How missing, null, empty, 0 and
`[]` behave in pills and tests: [missing-null-and-large-data.md](missing-null-and-large-data.md).

After inserting a pill, press Escape to close the picker. Because `/` opens the picker,
a `/` typed at the very end of a value is swallowed; one in the middle is kept.

## fx functions

The **fx** icon at the right edge of a field opens a pipeline of functions with a live
preview. Full list: [formula-functions.md](formula-functions.md).
- New functions start with **Wrap** and **Quotes** on: turn both off for a plain value,
  or you get `"\"CRITICAL\""`.
- A recursive JSON Path (`$..x`) always returns a list: add a second JSON Path `$[0]`.
- **A JSON Path that matches nothing crashes the step when Wrap is off** ("…reading
  'startsWith'"). If the path may be absent, use JSONata with a default, or turn Wrap on (no match
  gives `[]`; don't chain `$[0]` before checking for `[]`) or use
  JSONata with a default. JSONata that matches nothing returns the text `undefined`.
- Above the preview limit (large inputs), the preview skips JSONata; put an Array slice
  in front while designing. Real runs aren't affected. See
  [missing-null-and-large-data.md](missing-null-and-large-data.md#large-data-hundreds-of-kb-and-more).
- Keys with prefixes (XML namespaces) need bracket notation: `$['soap:Envelope']…`.

## Step settings (most steps)

*Settings* tab: **Timeout** (seconds, max 840), **On failure** = *Stop execution* or
*Continue execution*, **Retry attempts** (2 = 3 tries in total; a non-2xx status counts as
a failure). A step that waits for an approval also has **Resume timeout** (min 180 s).

*Mock* tab: **Enable mock** + status + JSON body. The step shows MOCKED and makes no
network call — handy to build before credentials exist. A mocked approval step doesn't
pause.

## The steps

**Transform data** — sets named variables. Variables are global and can be reassigned
later under the same name (that's how loops accumulate). Typing `[]` stores a real empty
array. Run logs show variable values base64-encoded.

**Run script** — pick a runtime first (Node.js 24 / 22 / 18, Python 3). The JSON input
is a separate field: put pills **without quotes** to pass arrays and objects, in quotes
to pass text. Node reads the input from `mindflowData` (`arguments[0]` also works) and
`return`s at top level; the output appears under `result`. Parse the input defensively
(it may arrive as text or object).

**HTTP Request** — method, URL, query, headers, body (JSON, Text, XML, URL-encoded,
Multipart, Octet stream). The JSON body accepts pills inside strings. JSON responses
are parsed (`body › …`); XML responses arrive as raw text. The XML body type sends
`Content-Type: application/xml`; add your own header to change it.

**Integration (native API) steps** — the integration must be enabled in Integrations,
and a credential for it created in the environment; then pick the operation in the step
picker. See [credentials.md](credentials.md).

**Send email** — `from` is the *name* of a sender configured for the organization, not
an address. To is a list. Pills work in Subject and Text. **Send email over SMTP** sends
through your own server with an SMTP credential.

**Condition** — IF / ELSE IF / ELSE; each branch is a Compare. Operators: Is equal to, Is
not equal to, Is greater than, Is greater or equal to, Is lower than, Is lower or equal
to, Contains, Does not contain, Starts with, Ends with, Matches regex, Is empty, Is not
empty, Is true, Is false (the last four take no second value). A branch can hold several
tests joined with **And** or **Or**. **The first true
branch wins.** ELSE can't target a step that already has a link: use an ELSE IF with the
opposite test. Add a branch with the add button under the Condition, then auto-layout.
The Condition's run log lists every branch as matched / not matched / not evaluated
with the values compared: the quickest way to see why a branch was taken.

**Loop** — For x iterations / While / **Do while**; Settings › Maximum iterations
(default 100). **Its condition can't read steps inside the loop**: write the value with a
Transform data step at the end of the body and test the variable. After the loop, read
results through VARIABLES (inner steps aren't offered). A do-while with an empty
condition runs exactly once. To add a step after the loop, select the Loop and use the
add button under its bottom edge.

**For each** — iterates a list. **Max concurrency defaults to 1** (one at a time). Results
leave the loop through the **Iteration output** (Input › Iteration output › ⊕ ›
*Additional properties*): the For each output is an array in input order, values as text.
Call a flow steps don't appear in the iteration-output picker: copy their result to a
variable with a Transform inside the loop. **A variable set inside a For each belongs to
its iteration**: parallel iterations don't overwrite each other's value (tested with 10
iterations in parallel, each writing then reading the same variable name), and the
value is **not** readable after the loop (it comes out empty). Pass results out through
the iteration output. Above 250 KB of data or 40 parallel
iterations, turn on *Enable large dataset*.

**Wait** — pauses a fixed number of seconds (no condition).

**Call a flow** — lists flows with a **Flowhook** trigger deployed in the same
environment, then a version. Its parameters come from the sub-flow's input schema.
"Wait for completion" returns the sub-flow's outputs at `output.data.<key>`, with keys
in **lower case** (`messageId` → `messageid`). A failed sub-flow shows as `null` in the
parent: open the sub-flow's own run for the reason.

**Flow outputs** — define them in Flow settings › Output; set their values in the
Success step.

**AI Task** (more in [ai-agents.md](ai-agents.md)) — text completion, tool calls (an integration operation with its credential)
and agent calls (a Flowhook sub-flow used as an agent; the step becomes AI-SWARM).
Memory: None / New Memory / Previous Step Memory / Advanced (a Memory ID, e.g. a
`sessionId` from the caller, to continue a conversation across runs). **Attach a file**
lets it read a docx, xlsx, CSV, PDF, PNG or JPEG coming from the trigger, an API step or a
For each item ([ai-agents.md](ai-agents.md#files-letting-the-ai-read-documents-and-images)).
**Don't use angle brackets in prompts**; they cut the prompt.

**Webhook response** — only for sync calls; see [triggers.md](triggers.md).

**Success / Failure** — end the run. A Failure step's message becomes the run's error,
which is clearer for whoever reads the logs.

**Note / AI note** — sticky notes on the canvas; never run.

## Also available (not covered in detail here)

- **Paste from clipboard** in the add-step menu: paste steps copied from another flow.
- **AI-assisted field edit**: AI proposes a formula, mapping or text for one field.
- **MCP steps**: *list tools* and *call tool* on an MCP server, through a Connection
  credential holding its URL. Mindflow's documentation says the call's **timeout and
  authentication come from that Connection credential** (not tested here); one tool call
  per step.
- **SFTP steps**, and **Platform** connectors to automate Mindflow itself (flows,
  deployments, environments, credentials, logs).
- **Simple agents** (Flows › Create › Agent) and AI agents in general: see [ai-agents.md](ai-agents.md).
- **AI Build** proposes steps from a description, but only HTTP Request and integration
  steps; check what it creates.
