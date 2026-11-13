# What Mindflow offers, by need

Use this to map each part of the use case to a building block, and to sketch the flow.
Labels are the Studio's own.

## Starting a run (triggers)

| Need | Trigger |
|---|---|
| A tool sends an event / calls a URL | **Webhook** (async, or sync with a reply within 25 s); API key protection, IP allowlist |
| On a schedule | **Scheduler**: daily, weekly, monthly, yearly or once; several hours and minutes can be ticked |
| An email arrives | **Emailhook**: the flow gets an address; sender, subject, text and HTML body, and SPF/DKIM results. Whether attachments or a forwarded original come through isn't covered here: if they matter, mark it "to check" (or read the message through the mail platform's API) |
| A person asks in a chat | **Chathook**: mention the flow in an AI Room |
| Another flow calls it | **Flowhook** (sub-flow with typed inputs; also how a flow becomes an AI agent tool) |
| Someone clicks Run | **Manual** |

A flow can have several triggers.

## Talking to systems

- **Native integrations**: pick an operation from the integration's catalogue, with a
  credential. Which integrations exist is shown in the user's Studio (Integrations).
- **HTTP Request**: any HTTP API — methods, headers, query, JSON / text / XML /
  URL-encoded / multipart / binary bodies. GraphQL and SOAP go through it.
- **Credentials**: API key (header or query), Basic, Bearer, OAuth 2.0 (client
  credentials, authorization code, password), Digest, SMTP, SSH, LDAP, AWS (SigV4, assume
  role), Google service account, Azure, HashiCorp Vault and vendor-specific types. Secrets
  stay in the credential. **Credential Lookup** picks a credential by name per run (for
  several tenants or instances).
- **Email out**: Send email (Mindflow's sender) or over the customer's SMTP server.
- **SFTP**, **SSH** and **MCP** steps (call tools on an MCP server).
- **Private tools**: IP allowlist of Mindflow's addresses, or the **Relay Agent**
  (outbound-only connector in the customer's network, set up with Mindflow).

## Logic and data

- **Condition** (if / else if / else; equals, contains, regex, empty, greater than…;
  several tests joined with And / Or).
- **Loop** (for N, while, do while), **For each** over a list, with parallelism.
- **Wait** (fixed seconds), **Transform data** (variables), **fx functions** (over 200:
  JSON Path, JSONata, regex, dates and time zones, arrays, maths, text, encoding, hashing).
- **Run script**: Node.js or Python, for logic no step covers.
- **Call a flow**: reuse a sub-flow, wait for its result or run it in the background.
- **Artifacts**: a key-value store per environment to keep state between runs (seen IDs,
  last cursor, pending requests), with a retention period.
- Large payloads (megabytes) are handled.

## People in the loop

- **Approvals**: a step sends a link in a message (email, a chat tool such as Teams or
  Slack through its integration or API, or a ticket); the approver clicks it, the run
  pauses until someone answers or a timeout passes, and branches on the answer or on
  silence (e.g. escalate after 30 minutes).
- Notifications through email, chat tools or ticketing integrations.

## AI

- **AI Task** step: text generation or classification, tool calls (integration
  operations), sub-flows as tools, memory across messages.
- **Simple agents** (Create › Agent): an agent with tools, testable in a chat, usable in
  **Rooms**.
- Good for triage, summaries, enrichment, drafting replies, routing. Keep a human
  approval in front of irreversible actions.

## Running it

- **Environments** (e.g. test and production), each with its own credentials and
  variables; **versions** to freeze what runs; run **logs** with each step's input and
  output; replay with the same input; email alerts on failure.
- Sync API pattern: a flow can act as an API endpoint answering within 25 s.

## Not what Mindflow is for

- Driving screens or desktop software (no RPA).
- Continuous real-time control loops.
- Replacing a data warehouse or a full ETL pipeline for very large batch volumes (it can
  orchestrate them, calling the tools that do the heavy lifting).
