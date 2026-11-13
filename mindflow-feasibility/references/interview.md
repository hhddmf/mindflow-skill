# Interview guide

Ask one question at a time. Skip anything the user already said. Prefer questions whose
answer changes the score, the effort or the flow sketch. Order of priority:

## 1. The process (always)

- "Walk me through it: what starts it, what happens next, and what's the end result?"
- "How is it done today, and roughly how often (per day / week)?"

## 2. Each system (the questions that decide the score)

For every system that appears, find out:
- **What it is**: product name and whether it's a SaaS or hosted by the company.
- **API**: "Does it have an API you can use?" If unknown: "Do you know whether
  [product] has an API? Most do; their documentation or admin will know." Note it as
  "to check", don't block on it.
- **If no API**: "Can it export data on a schedule (to a file server or SFTP), send
  emails or notifications, or call a URL when something happens?"
- **If hosted internally**: "Could your network team either allowlist a few fixed
  addresses for incoming calls, or run a small outbound-only connector in your network?"
  Only if both are refused, or the system sits on an isolated network, is connectivity
  hard.
- **Authentication** (kind only, never values): API key, OAuth, user/password, service
  account. Unusual methods (client certificates, signed requests) are worth noting.
- **Essential or nice-to-have** for the use case.

## 3. Trigger and timing

- "Should it start on an event, on a schedule, from an email, from a chat, or on demand?"
- "Can [tool] notify us when the event happens, or do we need to check it regularly?"
  (Either works; polling just adds effort.)
- "Is someone or something waiting for an immediate answer?" → if yes, how fast (the 25 s
  sync limit).

## 4. Logic, data and people

- Decisions: "What rules decide what happens? Are they clear-cut, or a judgement call?"
  (Judgement → an AI step, usually with a human check.)
- Volume and size: "How many items at once? Large files or attachments?"
- People: "Does anyone need to approve or answer? How fast do they usually answer, and
  what should happen if nobody does?"
- State: "Does it need to remember things between runs (already handled items, a
  last-processed date)?"

## 5. Constraints (only if relevant)

- Compliance or data residency requirements, data that must not leave a system.
- Who will build and maintain it, and whether there's a test environment of each tool.

## Stopping rule

The restatement in step 2 counts as one question: end it with "Is that right?" only.

Stop when you can fill the report: each essential system has an access rating (even
"to check"), you can draw the flow sketch, and the effort is clear. Put the rest in
"open points". A typical interview is 4–10 questions.

## Handling the user

- **Vague idea** ("automate our SOC"): ask them to pick one concrete process first,
  and note the tools they name; offer to qualify the next process afterwards.
- **A system with no API that is only nice-to-have**: don't dig for other channels;
  mark it manual and move on.
- **Several use cases at once**: qualify one, then offer to do the next.
- **"I don't know"**: note it, give the usual answer for that kind of tool, move on.
- **Pressure for a promise** ("just tell me it's 100% doable"): give the score with its
  confidence and what remains to check. No guarantees on connectors, prices, dates or
  roadmap: their Mindflow contact can answer.
- **Questions about Mindflow's internals, other customers or security documents**: don't
  answer; point to their Mindflow contact or support. You may still explain the
  capabilities in these references (e.g. secrets stay in credentials, Credential Lookup)
  and any limit that applies to their use case.
- **Secrets pasted by mistake**: don't repeat them, tell the user to remove the message
  where possible and to rotate the secret, and continue without it.
