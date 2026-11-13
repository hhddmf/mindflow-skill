# Scoring a use case

Two separate measures. Keep them separate in the report.

## 1. Feasibility score (access only)

Rate each system the use case needs, then take the **lowest rating among the systems the
use case can't do without**. A nice-to-have system with no API becomes a step that stays
manual; list it, but it doesn't lower the score.

| Score | Meaning | Typical cases |
|---|---|---|
| **High** | Every essential system has an API (REST, GraphQL, SOAP, or a supported protocol below) and Mindflow can reach it | SaaS tools with an API; on-prem tools with an API, reached through an IP allowlist or the Relay Agent; APIs on a paid tier the customer already has |
| **Medium** | The APIs exist, but reaching one of them is genuinely hard | The network forbids both exposing the tool (even with an IP allowlist) and running the Relay Agent; an isolated or air-gapped segment (OT, industrial, classified); the only path is a VPN or jump host that needs an interactive login; the security team must approve a new network path, with an uncertain outcome |
| **Low** | An essential system has **no API**, but another machine channel exists | Scheduled exports dropped on SFTP; emails or notifications the tool sends (read by an Emailhook); a database a team can expose through a small API; an API only on a licence tier the customer doesn't have |
| **Not feasible** | An essential system has no API and no machine channel, or cannot be reached at all | Desktop-only or screen-only software (Mindflow doesn't drive screens), terminal "green screens" included; paper or phone steps; a network that allows no connection in or out, permanently, with no export |

**Grey zones:**
- An API exists but policy forbids any connection *today*, and an exception or review
  is possible → **Medium** (the review is an open point). Only when no path can ever be
  approved and there's no export is it **Not feasible**.
- A path exists but can't run unattended (VPN or jump host with an interactive or
  hardware-token login) → **Medium**, with "approve an unattended path" as the open point.
- An essential system's API is only on a licence tier the customer may not have → score
  as if they had it, confidence **To check**, and say the score drops to Low otherwise.
- A latency or volume target the systems themselves can't meet (an answer in 5 s when
  the APIs take 60 s) is a **fit note**, not a lower score.
- Say what **Low** means when you give it: the data is reached through a file, an email
  or another indirect channel instead of an API. The use case can still work well.

### What counts as an API or machine channel

- HTTP(S) APIs of any style: REST, JSON or XML, GraphQL, SOAP, webhooks the tool sends.
- Protocols Mindflow has steps or credentials for: SMTP (email out), email in
  (Emailhook), SSH, SFTP, LDAP, MCP servers, and cloud provider APIs (AWS, Azure, Google
  Cloud…).
- The tool can call Mindflow: it sends webhooks or can call a URL → push works.

Not machine channels: screens and GUIs, raw TCP/UDP or serial protocols with no
HTTP gateway, files on a desktop.

### Connectivity: normal vs hard

**Normal (no impact):** SaaS on the internet; an on-prem tool that pushes events out to
Mindflow's webhook URL (it only needs outbound internet access; if that's blocked, poll
the tool through the Relay Agent instead); a SaaS or on-prem tool with an IP
allowlist (Mindflow's outbound addresses come from Mindflow support); a private tool
reached through the **Relay Agent** (a small connector the customer runs inside their
network, outbound connection only, set up with Mindflow). Internal certificates: fixable
on the proxy or tunnel side. List these as **prerequisites**.

**Hard (lowers to Medium):** see the table. Ask before assuming: "Could your network
team either allowlist Mindflow's addresses or run a small outbound-only connector?" If
yes to either, it's normal.

## 2. Build effort (never affects the score)

| Effort | Typical content |
|---|---|
| **Light** | 1–3 systems, native integrations or simple HTTP calls, linear flow with a few conditions |
| **Moderate** | One or two of: HTTP-only or GraphQL/SOAP APIs, pagination, a loop or For each, polling with a "last seen" marker, an approval, an AI step, a sub-flow, a small script |
| **Heavy** | Three or more of the Moderate items together, or: many systems or teams, multi-step approvals, several sub-flows, AI agents with tools, large custom scripts, a two-way sync, a rollout across many tenants or environments |

Pick one value (no "Light to Moderate"); name what would push it up a level.

Things that raise effort only:
- No native integration (→ HTTP Request + a credential).
- GraphQL or SOAP (→ HTTP Request with a JSON or XML body).
- Polling instead of push (→ Scheduler + state kept in Artifacts).
- Scripts (Node.js or Python steps), data reshaping, large data.
- Sub-flows, loops, parallel processing, approvals, AI decisions.
- Rate limits and pagination.

## 3. Fit notes (no impact on the score, but say them)

Some requirements need a design choice; state the constraint and the usual answer:

| Requirement | Note |
|---|---|
| The caller must get an answer synchronously | A synchronous webhook must answer within **25 s**; longer work → answer "accepted" and call back, or let the caller poll |
| One step takes very long | A step times out at **840 s** at most; split the work or poll a job status |
| A human must answer | Approvals pause the run; waits of hours or days work. For weeks, store the request and resume from a scheduled check |
| Very frequent schedule | The Scheduler has no "every N minutes" option, but many minute slots can be ticked; event-driven (webhook) is better when available |
| Sub-second or continuous real-time control | Mindflow runs flows per event, with seconds of overhead; it isn't a real-time control loop |
| Very high volume of events | Fine in principle; flag it so their Mindflow contact can size it |

## Confidence

Add a confidence level to the score:
- **Confirmed**: the user confirmed an API (or channel) and an access path for every
  essential system. Unknown auth types don't matter here.
- **Likely**: every essential system is identified, but some API or access is assumed
  from what's typical for that kind of tool.
- **To check**: an essential system isn't identified yet, or a deciding fact (API
  access on their plan, a network approval) is unknown.

Uncertainty goes into the open points, not into a lower score.
