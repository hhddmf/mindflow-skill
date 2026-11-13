# Mindflow Platform Capabilities — Flow Design Reference

**Purpose:** Use this file when generating flow descriptions in the workbook. Reference actual Mindflow primitives by their canonical names so flows are technically precise and immediately actionable for builders.

---

## Triggers (how a flow starts)

Use the exact trigger name in flow Trigger cells and descriptions.

| Trigger | When to use |
|---------|-------------|
| **Webhook trigger** | External system calls a Mindflow URL to start the flow (e.g. SIEM alert, Proofpoint TAP event, ZIIVE webhook). Pair with **Webhook response step** if a response is needed. |
| **Scheduler trigger** | Flow runs on a timer — periodic polling, nightly syncs, weekly reports. No external caller needed. |
| **Emailhook trigger** | Flow starts when email arrives at a Mindflow-provided address. Subject, body, and attachments appear as trigger data. |
| **Chathook trigger** | Flow starts when a chat message is sent from Rooms or an external chat client. Used for Rooms-initiated agentic flows. |
| **Flowhook trigger** | Flow is called as a sub-flow by another flow via **Call a flow step**. Use for reusable modules (e.g. "enrich indicator", "notify analyst"). |
| **Formhook trigger** | Flow starts from a structured form or JSON submission — human-in-the-loop intake patterns. |

---

## Core Flow Control Steps

| Step | When to use |
|------|-------------|
| **Condition step** | Branch on a formula — severity thresholds, MDR escalation flags, confidence score checks, tool type routing. |
| **For each step** | Loop over a list — process each IOC, each alert, each affected mailbox. |
| **Loop step** | Repeat with a break condition — retry logic, polling until a state changes. |
| **Wait step** | Pause execution for N seconds — rate limiting between API calls, short delay before polling a result. |
| **Transform data step** | Reshape or compute data between steps — rename fields, filter arrays, prepare API payloads, aggregate results. No external call. |
| **Call a flow step** | Invoke another flow as a reusable sub-flow and use its output. Pair with Flowhook trigger on the target. |
| **Webhook response step** | Return an HTTP response to the caller when the flow was triggered by a webhook. |
| **Run script step** | Execute sandboxed custom code for complex parsing, formatting, or logic that Transform data can't express. |
| **HTTP request step** (Built in) | Call any HTTP API when no catalog integration exists. Full control over method, headers, body. |
| **Send email step** (Built in) | Send email through Mindflow's platform email service. No SMTP credential needed. |
| **Send email over SMTP step** (Built in) | Send email through a customer-owned SMTP server. Requires SMTP credential. |
| **Success step** | Terminate the run successfully with optional structured output. |
| **Failure step** | Terminate with a clear error message when business rules are not met. |

---

## AI Steps

| Step | When to use |
|------|-------------|
| **AI task step** | Single-turn AI call inside a flow — classification, extraction, summarisation, verdict generation, email drafting. Sends a prompt + context to an approved model, returns structured or text output for downstream steps. The most common AI primitive. |
| **Agent step** | Multi-turn reasoning with memory and tool access inside a run. Use when the AI needs to call multiple tools in sequence, reason across results, and produce a synthesised output. Supports **Agentic flow tools** — the agent dynamically picks which integration or Built in step to call. |

**Agentic flow tools:** When an Agent step has tools configured, it can invoke allowed integrations or Built in steps dynamically during a run rather than following a fixed step order. Use this for investigation agents that need to pivot across multiple systems.

**Rooms + Chathook:** When a flow uses a Chathook trigger, it can be exposed in Mindflow Rooms as an interactive agent. Analysts type questions in Rooms; the flow runs, returns results to the chat. Thread memory can be maintained across turns within a session.

---

## Sub-flow Pattern (reusable modules)

When the same logic is needed across multiple flows (e.g. "enrich an IOC", "create a ZIIVE ticket", "notify analyst via Teams"):
1. Build the reusable logic as a flow with a **Flowhook trigger**
2. Call it from parent flows using **Call a flow step**
3. The same sub-flow can also be exposed to Rooms via **Chathook trigger** on a wrapper flow

---

## Mindflow Platform Connectors (built-in)

Native connectors available in the integration catalog under the **Mindflow** vendor group:
- **Platform Artifacts** — store and retrieve JSON/text/XML payloads between runs (list, put, get, delete artifact steps). Use for persisting enrichment results, cross-run state, or large payloads.
- **Platform Flows / Deployments / Credentials / Logs** — manage Mindflow resources from flows (admin automation).
- **SFTP** — read, write, list, rename remote files over SSH.
- **MCP outbound (Beta)** — call tools on an external MCP server (list tools, call tool). Use when the target system exposes an MCP endpoint.

**Inbound MCP server:** Mindflow can also expose its own integration operations as MCP tools to external clients via the Platform MCP Server endpoint. Relevant for customers building AI agents outside Mindflow that need to call Mindflow-connected integrations.

---

## Credentials and Connections

Reference these when describing how a flow authenticates:
- **HTTP API key / Bearer token / Basic auth** — for REST API integrations without a catalog connector
- **OAuth authorization code** — user-delegated access (Microsoft 365, Google Workspace, Salesforce)
- **OAuth client credentials** — machine-to-machine (most SIEM/SOAR APIs)
- **Google service account** — Google Cloud / Google Workspace Admin SDK (impersonation patterns)
- **AWS Signature V4 / Assume role** — AWS API calls
- **Security vendor credentials** — CrowdStrike Falcon, Zscaler, and other security tool APIs (vendor-specific shapes in the credential picker)
- **HashiCorp Vault AppRole** — fetch secrets from Vault at runtime instead of storing in Mindflow

---

## How to use this file when generating flows

For every flow in the workbook:

1. **Trigger cell (col D):** Name the exact Mindflow trigger — e.g. "Webhook trigger (ZIIVE new alert event)" not "when ZIIVE fires".

2. **Flow Description cell (col F):** Reference Mindflow steps by name when describing the chain. Examples:
   - "A **Scheduler trigger** polls CrowdStrike Falcon Overwatch every 5 minutes..."
   - "A **Condition step** checks alert severity against the configured threshold..."
   - "An **AI task step** receives the enriched ticket context and returns a verdict, confidence score, and missing-evidence checklist..."
   - "An **Agent step** with Agentic flow tools dynamically queries CrowdStrike Falcon and Cortex before synthesising a report..."
   - "A **Wait step** pauses execution for 3,600 seconds before the escalation branch checks for a Teams response..."
   - "A **For each step** iterates over the extracted IOC list, triggering a Cortex analyzer sub-flow per indicator via **Call a flow step**..."
   - "The **Chathook trigger** receives the analyst's question from Mindflow Rooms; the **Agent step** queries CrowdStrike Falcon via Agentic flow tools and returns results to the chat with thread memory."

3. **Never invent platform primitives.** Only reference steps and triggers listed in this file. If a capability isn't here, describe it generically without naming a Mindflow step.

4. **Don't over-engineer.** Most flows need 3–6 steps. Simple alert-to-ticket flows don't need Agent steps — use AI task step for single-turn AI work. Reserve Agent step + Agentic flow tools for genuinely multi-tool, multi-turn scenarios.

---

## Transform Functions (Data Transformation in editable fields and Transform data steps)

Mindflow exposes a library of transform functions available in **any editable field** (via the `fx` icon) and inside **Transform data steps**. Reference these by name when describing how a flow processes data — they replace the need for custom code in most ETL scenarios.

Functions are grouped below by the scenario they solve, not by their menu category.

---

### IOC Extraction from raw text, alerts, or email bodies

Use these when a flow needs to pull indicators out of unstructured input:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **Extract IP addresses** | Finds all IPv4/IPv6 addresses in a text block | Pull IPs from alert body, log line, or email header |
| **Extract Domains** | Finds all domain names | Extract sender domains, C2 candidates from email or SIEM alert |
| **Extract URLs** | Finds all URLs (with protocol) | Pull links from phishing email body for sandbox detonation |
| **Extract Email Addresses** | Lists all email addresses in text | Parse recipients/senders from raw email payloads |
| **Extract MAC addresses** | Finds hardware identifiers | Network forensics, device correlation |
| **Extract Dates** | Finds date strings in any format | Normalize timestamps across heterogeneous log sources |
| **Regular Expression** | Custom pattern extraction | Extract alert IDs, ticket numbers, custom IOC formats |
| **Defang IP Addresses / Defang URL** | Converts `http://` → `hXXp://`, dots → `[.]` | Safe IOC sharing in tickets and notifications |
| **Parse URI** | Breaks a URL into scheme, host, port, path, query params | Decompose phishing URLs for per-field analysis |
| **Parse IP range / IP Address In CIDR Range Check** | Expand IP ranges; test if IP is in a CIDR block | Whitelist/allowlist checks, network segmentation logic |
| **Format MAC addresses** | Standardizes MAC notation | Device correlation across tools with different MAC formats |

---

### JSON parsing and reshaping (alert payloads, API responses)

Use when a flow receives a JSON payload that needs querying or restructuring before the next step:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **JSONPath Plus** | Query and extract values from JSON using dot/bracket notation and filters | Pull specific fields from a ZIIVE ticket payload, CrowdStrike Falcon detection JSON, or Proofpoint TAP response |
| **JSONata** | Full JSON transformation and computation — filter, map, reshape, aggregate, compute | Restructure a CrowdStrike Falcon detection object into the format expected by a ZIIVE API call; compute severity scores; create new JSON structures from combined sources |
| **JSON Path / JSONata** are the primary tools for any "extract this field" or "reshape this object" step | | |

**When to use which:**
- **JSONPath** → read-only extraction ("give me `alert.severity` and `alert.affected_device`")
- **JSONata** → transformation ("take the CrowdStrike Falcon detection, rename fields, filter to HIGH severity indicators, and output a ZIIVE-ready object")

---

### Array and list manipulation (IOC lists, alert batches, deduplication)

Use when a flow works with lists — multiple IOCs, multiple alerts, multiple affected devices:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **Array unique** | Removes duplicate values | Deduplicate IOC lists before sending to enrichment |
| **Array filter** | Keeps only elements matching a condition | Filter alert list to HIGH/CRITICAL severity only |
| **Array concat** | Merges multiple arrays | Combine IOCs from multiple sources before a single lookup |
| **Array sort** | Sorts array elements | Order alerts by severity or timestamp |
| **Array slice** | Extracts a portion of an array | Take first N alerts for batch processing |
| **Array join** | Converts array to string | Format IOC list as comma-separated for an API call |
| **Array split** | Splits a string into an array | Parse a comma-separated tool output into individual items |
| **Array flat** | Flattens nested arrays | Normalize multi-level enrichment results |
| **Set Intersection** | Items in both lists | Find IOCs that appear in both the alert AND the known-bad list |
| **Set Difference** | Items in list A not in list B | Remove already-known IOCs; find net-new indicators |
| **Set Union** | Merge two lists, no duplicates | Combine IOC sources into one deduplicated master list |

---

### Format conversion (emails, exports, mixed-format inputs)

Use when the input format doesn't match what the next step expects:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **CSV to JSON** | Converts CSV to JSON array | Ingest SIEM exports, threat intel feeds in CSV format |
| **XML to JSON** | Converts XML to JSON | Parse Proofpoint or legacy SIEM XML alert formats |
| **MSG to JSON** | Parses Outlook `.msg` files into JSON (sender, recipients, body, metadata) | Process forwarded phishing reports from Outlook mailboxes |
| **Strip HTML tags** | Removes HTML markup, leaves plain text | Clean email body before AI analysis |
| **From Base64 / To Base64** | Decode or encode Base64 | Decode email attachments; encode payloads for APIs that require Base64 |
| **From Hex / To Hex** | Decode or encode hexadecimal | Handle binary IOC data from forensic tools |
| **XPath** | Query specific fields from XML | Extract specific fields from XML-format SIEM alerts |
| **Parse X.509 certificate** | Extracts issuer, subject, expiry, public key from a certificate | TLS certificate analysis in phishing/C2 investigation |

---

### Date and time normalization

Use when timestamps arrive in different formats across tools and need standardizing:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **From/To UNIX Timestamp** | Convert between UNIX epoch and human-readable datetime | Normalize CrowdStrike Falcon and ZIIVE timestamps |
| **Translate DateTime Format** | Reformat dates between timezone/format conventions | Align timestamps from US-format tools with EU-format incident reports |
| **Parse DateTime** | Parse messy or ambiguous date strings into ISO format | Handle inconsistent timestamp formats from heterogeneous log sources |
| **Modify date with duration** | Add or subtract days/hours/minutes from a date | Calculate "alert age", set SLA deadlines, compute escalation windows |

---

### Hashing (IOC fingerprinting, integrity checks)

Use when a flow needs to generate or verify file/data fingerprints:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **MD5** | 32-char hex hash — fast, widely used for IOC sharing | Generate MD5 hash of attachment for VirusTotal lookup |
| **SHA-256** | 64-char hex hash — current standard | Hash file for CrowdStrike Falcon indicator check |
| **SHA-1** | Legacy hash, still common in threat intel feeds | Match against older IOC databases |
| **SSDEEP (Fuzzy Hash)** | Detects files that are "similar" but not identical | Detect variants of known malware in phishing attachments |

---

### Text and string utilities

Use inside field formulas or Transform data steps for string manipulation:

| Function | What it does | Typical use |
|----------|-------------|-------------|
| **Find / Replace** | Regex-powered search and mass-replace | Normalize alert field names, sanitize API inputs |
| **Filter** | Keep or remove lines containing a keyword/regex | Filter log lines to only those containing the IOC |
| **Concatenate** | Join multiple strings | Build a ZIIVE comment body from multiple enrichment outputs |
| **Count occurrences** | Count how many times a pattern appears | Determine IOC frequency across a log batch |
| **Remove whitespace** | Strip spaces, tabs, newlines | Clean inputs before Base64 decode or API call |
| **Levenshtein Distance** | Measure string similarity | Detect typosquatted domains (e.g. `gooogle.com` vs `google.com`) |
| **UUID V4** | Generate a unique identifier | Create correlation IDs for multi-step flows |

---

### How to use these in flow descriptions

When a **Transform data step** performs a non-trivial operation, name the function:

- ✅ "A **Transform data step** applies **JSONata** to reshape the CrowdStrike Falcon detection object into a ZIIVE-compatible alert payload, mapping `device.hostname` → `asset`, filtering to `severity = HIGH` indicators only."
- ✅ "A **Transform data step** uses **Extract IP addresses** and **Extract Domains** on the raw email body to produce two IOC arrays, then **Array unique** to deduplicate before the enrichment loop."
- ✅ "A **Transform data step** applies **Set Difference** to remove already-known indicators (from the existing ZIIVE ticket) from the newly extracted IOC list."
- ✅ "A **Transform data step** uses **MSG to JSON** to parse the `.msg` attachment, then **Strip HTML tags** on the body field before passing to the AI task step."
- ❌ "The flow extracts data from the payload." (too vague — name the function)

Don't name a function unless it's genuinely needed. Simple field mapping (copy `alert.id` to the next step) doesn't need a Transform data step or a function call — just use the field reference directly.
