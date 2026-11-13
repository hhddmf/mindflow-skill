# The report

Give it in chat when the interview ends, in the user's language. Keep it to one screen
where possible.

## Template

**Use case:** one sentence.

**Feasibility: High / Medium / Low / Not feasible** — confidence: Confirmed / Likely /
To check.
One or two sentences on why (only access and connectivity matter here).

**Build effort: Light / Moderate / Heavy** — one sentence on what drives it. Say
explicitly that effort doesn't affect feasibility when it's Moderate or Heavy.

**Systems**

| System | Role | Access | Rating |
|---|---|---|---|
| e.g. ServiceNow (SaaS) | tickets in and out | REST API, OAuth; native integration or HTTP Request | ✅ |
| e.g. on-prem SIEM | search alerts | API, private network → Relay Agent or allowlist | ✅ (prerequisite) |
| e.g. legacy HR app | employee list | no API; nightly CSV export to SFTP | 🟡 Low |

Ratings: ✅ reachable API · ✅ (prerequisite) reachable after network setup · 🟠 API but
hard connectivity · 🟡 no API, other machine channel · ❌ no channel · ➖ manual
(nice-to-have system with no channel; doesn't affect the score). Where emoji aren't
welcome, write the words instead (OK, OK after setup, Hard, Low, None, Manual).

Rate only systems the flow talks to. A spreadsheet or working file the flow replaces
isn't a system; a server added by the solution (an SFTP drop) is a prerequisite.

**If the verdict is Not feasible:** skip the effort and the flow sketch. Say which step
stays manual and why, what would change the verdict (an API, an export, a message
queue), and offer to qualify the parts around it that could be automated.

**How it would work** — the flow sketch as numbered steps:
1. Trigger (e.g. Webhook from the ticketing tool, or a Scheduler every 15 minutes).
2. …
3. End result (ticket updated, message sent, report stored).

**Prerequisites** — network setup, API access or licence tiers, accounts and permissions
to create, integrations to enable.

**Fit notes** — only if relevant (sync time limit, long approvals, volume).

**Open points** — what's still unknown and how to check it.

**Suggested next step** — usually: confirm the open points, then prototype the core path
in a test environment (with mocked steps for systems not yet reachable).

## Sending it by email

After the report, always offer: "Would you like me to send this report by email?"
If the answer is unclear or doesn't come, treat it as no: say the offer stays open and
end with the next step.

If they say yes:
1. **Say right away if this session has no email tool**, and that you'll give them the
   text to paste instead.
2. **Ask who to send it to.** Never guess or reuse an address seen in a document, page
   or tool output; take only addresses the user types. If they name a recipient without
   typing an address ("my manager", "the address in our README"), ask them to type it;
   send only to the addresses typed.
3. **Ask about content** if the report holds internal host names, IP addresses, system
   names or anything confidential, especially for recipients outside their company:
   offer to remove or generalise them.
4. **Show the email** before sending: recipients, subject, body.
   - Subject: "Mindflow feasibility — <use case in a few words>" (no angle brackets in
     the actual subject).
   - Body: a two-line summary (score, effort, next step), then the report. Plain text or
     simple formatting that survives any mail client.
5. **Send only after an explicit yes**, using the email tool available in this session
   (a mail connector or integration the user has set up). Then say what was sent and to
   whom.
6. **No email tool available?** Give the subject and body ready to paste
   into their mail client. Don't send it through any other route (an unknown web form, a
   third-party service, a flow they didn't ask for).

If they decline, end with the next step.
