# AI agents

There are two ways to put an AI agent to work:

| | **Simple agent** (Flows › Create › **Agent**) | **AI Task step** in a canvas flow |
|---|---|---|
| Built in | a dedicated form (tabs), no canvas | the designer canvas, as one step among others |
| Started by | a chat: the editor's **Test agent** panel, or `@` mention in a **Room** | any trigger (webhook, schedule, email, another flow…) |
| Good for | assistants people talk to | automations where AI is one step (triage, summarise, decide) |

Both use the same building blocks: a model, instructions, an input, **tools**
(integration operations), **agents** (other flows it can call), memory and model settings.

## Creating a simple agent

Flows › **Create** › **Agent**: give it a title and a **default environment** (it is
deployed there on "latest" right away). The editor opens with:

- **Input** tab: **Model** (vendor shown on the right), **Instructions** (the system
  prompt: who the agent is and how it should behave), **Input** (the message sent to the
  model, pre-filled with "You are an assistant. Please help resolve the following user
  query:" and an **Input message** pill = what the user typed), **Add file**. The footer
  shows the chosen model's cost in credits, intelligence, speed, tool support and region.
- **Tools** tab, **Agents** tab, **Settings** tab (below), **Logs**, **Info**.
- The **Test agent** chat on the right: talk to the agent while you build it.

## Tools: what the agent may do

Tools › **Add a tool** › search an integration (e.g. AbuseIPDB). For each tool:
- **Credential** (required): a credential of that integration in the environment, a
  Global one, or **Credential Lookup**. The tool uses it for every call. With Credential
  Lookup, type the credential's name (or insert a pill) in *Credential name*; it must
  match a credential of that integration. If the name points at the wrong credential,
  the agent gets the vendor's 401 and usually reports it in its answer: test with the
  Test agent chat after any change.
- **Name** and **Description**: the model reads them to decide when to use the tool;
  describe what it is for in one sentence.
- **Operations**: a new tool starts with **0 operations allowed** (warning icon) — the
  agent then cannot use it. Tick the operations it may call (or **Auto select
  operations**, or **Allow all**).

Guidance:
- **Allow only the operations the agent needs.** Every allowed operation is described to
  the model on each message, which costs tokens and makes wrong choices more likely.
- Prefer **read-only** operations. For actions with side effects (isolate a host,
  disable a user, delete data), keep them out of the agent, or put them in a deterministic
  flow with an approval step and expose that flow instead.
- Give the tool's credential the **least privilege** that works.

## Agents: delegating to other flows ("swarm")

Agents › **Add an agent** — "Choose a flow for the AI Task to interact with". The list
shows flows **with a Flowhook trigger, deployed in this environment**; pick one, then a
**version** ("latest" or a named one). The agent can now call that flow and use its
outputs. A **View swarm** button appears in the header: a map of the agent and its
sub-agents with their descriptions (fill the descriptions in: they help the model
choose).

The sub-flow's inputs come from its Flowhook schema; its outputs are what it returns in
Success. Pinning a named version keeps the sub-agent stable while you edit it.

## Settings: model and memory

- **Model**, then **Model configuration › Advanced**: Output format (Text or JSON),
  Frequency penalty, Presence penalty, Temperature, Top P. Lower temperature for more
  consistent answers.
- **Memory › Mode**:
  - **None** (default): **every message is answered on its own.** Even in the same chat,
    the agent doesn't remember the previous message.
  - **New Memory**: starts a fresh memory.
  - **Advanced**: **Append AI-Task job to memory** + a **Memory ID**. Messages that use the
    same Memory ID share one conversation memory.

**Memory ID: who shares the memory.** In a simple agent, the only data offered for the
Memory ID is the input message itself, so in practice the ID is a fixed text — and then
**every user and every conversation shares the same memory**: one user's details can
surface in another user's answers. Use a fixed ID only for a single-user or demo agent.
For per-user or per-conversation memory, build the agent as an **AI Task in a canvas
flow** and use a Memory ID that comes from the caller (e.g. a `sessionId` sent with each
message): that pattern has been verified.

In a canvas AI Task there is also **Previous Step Memory** (continue from an earlier AI
step in the same run).

## Untrusted content in prompts

Emails, tickets, alerts and web pages that you pass to an AI step are **data written by
someone else**. They can contain text aimed at the model ("ignore your instructions…").
- Say so in the Instructions: the content is to be analysed, never followed, and text
  addressed to an AI is itself a warning sign.
- Don't let the AI step's answer alone trigger an action with side effects: check it with
  a Condition, add deterministic checks, and send uncertain cases to a person.
- Keep tools read-only and credentials least-privilege, so a manipulated answer can do
  little harm.

## Testing and running

- Each message in the **Test agent** chat is a **real run** (listed in Logs as a WEBHOOK
  run), with real tool calls. A settings change redeploys the agent first, so the first
  message after an edit is slower.
- **Rooms:** open Rooms, pick the agent's environment at the top of the room list, and
  type `@` + the agent's name (if the menu doesn't appear, move the cursor after typing).
  Each new conversation becomes a room, titled automatically.
- **Calling a simple agent from outside:** a simple agent is a flow (Webhook trigger → AI
  step → Success returning `agentResponse`), but the chat passes the message in a way a
  plain webhook call doesn't: posting `{"Input message": "…"}` to its webhook reaches the
  agent **with an empty message**. To call an agent from another system, build it as a
  canvas flow (Webhook → AI Task → Webhook response) whose input reads the webhook body.
- An AI step can't be mocked: testing an agent always calls its model (and uses credits).

## What to check when an agent misbehaves

| Symptom | Likely cause |
|---|---|
| "I don't have a tool for that" / never calls the tool | the tool has **0 operations allowed**, or its name/description doesn't match the request |
| The tool call fails with 401/403 | the tool's credential is wrong or lacks rights |
| Forgets what was said a message ago | Memory is **None** |
| Mentions things another user said | Advanced memory with a **fixed** Memory ID shared by everyone |
| Doesn't use a sub-agent | the sub-flow has no Flowhook trigger, isn't deployed in this environment, or its description is empty |
| Answers but ignores part of the instructions | instructions too long or contradictory; also avoid angle brackets in AI prompts (they cut the text in AI Task prompts) |
| Wrong or invented facts | normal model behaviour: ask it to use tools for facts, lower the temperature, and check important answers |

Read the run in Logs: the AI step's log shows the answer, the memory ID used and the
token usage; the tool or sub-flow calls appear as their own runs or entries.
