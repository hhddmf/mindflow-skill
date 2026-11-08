# Credentials

## Where they live

- **Created per environment**: Environments › the environment › Credentials › **Add
  credential**. Credentials in the **Global** environment are offered everywhere.
- **Chosen per deployment**: picking a credential in a step's Settings saves it on that
  deployment straight away. Deploying the flow to another environment starts with no
  credentials: pick them again there. The choice survives *Update version* as long as the
  step still exists.
- A step's selector lists only credentials **of its own service**, Global ones, and
  **Credential Lookup**.

## Creating one

1. **Service**: *Built-in* (HTTP), *Vendor*, or a native integration with its version
   (enable the integration in Integrations first). Some services offer **connection
   templates** that pre-fill fields.
2. **Credential name**: use the **same name in every environment**. It makes promotion
   obvious and Credential Lookup portable.
3. **URL**: HTTP offers only **Any** (the step keeps its own URL). Vendors offer
   **Custom** (an address) or Any. Native integrations also offer their server template
   (e.g. `https://{instance}.service-now.com` with an Instance field). With Custom or a
   template, the credential decides the base URL, so swapping the credential swaps the
   instance (sandbox vs production).
4. **Credential type**: check it — the list isn't filtered by service and keeps your
   last choice.
5. The type's fields, then **Advanced** for optional ones.

Secrets belong in credentials, never in environment variables, flow variables or step
fields (those values appear in run data). The user types the secret; it stays masked and
can't be read back. Editing a credential
without retyping the secret keeps it. **Creating a credential doesn't test it**: a wrong
secret shows up at the first run.

The credential row's menu offers **Move to** (another environment, or Global) and
Delete. **Usage** shows which flows, rooms and tools use it — check it before rotating or
deleting. The **MCP** column gives an MCP connection (URL and a per-credential token) so an
external MCP client can use the service without ever holding the vendor secret; that
token is a secret too.

## What each type sends

| Type | Effect |
|---|---|
| HTTP API Key | Query parameters and/or headers, all added (query merged with the URL's own) |
| HTTP Basic | `Authorization: Basic …` from user and password |
| HTTP Bearer | `Authorization: Bearer <token>`; the step's body is kept |
| HTTP Body Secret | **Replaces the whole request body** (even on GET); only for login-style endpoints |
| HTTP Digest | Failed in testing where other clients succeed: test against your target first |
| OAuth 2.0 Client Credentials | Fetches a token at run time, then sends it as Bearer; Scope is passed; client authentication as Basic header or in the body; a wrong secret fails before the API is called (`invalid_client`); expect a token request on each run |
| OAuth 2.0 Authorization Code | *Authorize and create* opens the provider's sign-in; register Mindflow's callback URL (shown in the form) in the provider app |
| OAuth 2.0 Resource Owner Password, Application; SMTP; SSH; LDAP; AWS (SigV4, Assume Role); Google service account; Azure; IBM; Oracle; HashiCorp Vault; vendor-specific types | Fill the fields the form asks for; not covered in detail here |

Switching a step's credential replaces everything the previous one added. A step's
Request log never shows the credential's values, **but if the API echoes the request back,
the secret lands in the Response log**: never point a real credential at an echo or test
service.

## Credential Lookup (pick the credential by name at run time)

In the credential selector, **Credential Lookup** takes a name (a constant, a pill or a
formula). At run time Mindflow looks for that name among the credentials of the same
service in the environment the run belongs to (and Global). It is offered on steps and on AI agent tools alike. Use it to switch instance or tenant per run:
- **Per environment**: an environment variable holding the credential name.
- **Per customer or tenant in the payload**: name the credentials after the payload values
  (`acme`, `globex`…) and put the **trigger pill** (e.g. `body › customer`) directly in
  the lookup name. For names that differ from the payload, use a flow variable defined in
  **Flow settings** (a map) and an fx function such as JSONata to pick the entry. Changing the variable takes effect on the next run.

Rules:
- **The variable must also be used by an ordinary step** (a Transform data that copies it
  is enough); otherwise it isn't loaded and the name comes out empty.
- It can read constants, environment variables, **flow variables defined in Flow
  settings**, and the trigger input. It **cannot** read outputs of other steps — and that
  includes variables set by a **Transform data** step: the lookup is resolved before any
  step runs.
- The name must match exactly one credential of that service; otherwise the step fails
  ("No credential named …").
- **It works only on a deployment that follows "latest".** A deployment pinned to a named
  version runs that step with no credential: choose a credential on the deployment
  instead.
- After switching a deployment to "latest", the lookup takes effect at its next
  redeploy, which happens on the first run after any edit of the flow.
- It may be switched off on some platform deployments.

## Before the first run in a new environment

A step with **no credential** — not yet chosen after a deploy, or a lookup whose name came
out empty — doesn't stop: it calls the integration's **default public address without
authentication**, sending the request data to the real vendor. So:
- choose every credential, and create every variable a lookup reads, before the flow is
  triggered in that environment;
- guard a lookup with a Condition that stops when the name is empty;
- keep the trigger unannounced until this is done; if the flow is already being called
  or has a Scheduler, undeploy it first (see
  [troubleshooting.md](troubleshooting.md#i-need-to-stop-a-deployment-from-running-while-i-fix-it)).
