# Environments, versions and promotion

## The model

A **flow** can have one **deployment** per environment. Each deployment runs either
**latest** (the saved draft) or a **named version** (a frozen snapshot). Each deployment
has its own trigger URLs, which **don't change when its version changes**. Credentials
are chosen per deployment; environment variables are read at run time. The **Global**
environment isn't a deploy target: it holds credentials and variables shared by all
environments.

## Where things are

- **Versions**: designer toolbar › *Browse versions* (clock icon). A timeline of named
  versions with a read-only preview (structure only; step settings can't be opened).
  **Create a new version** asks for a name and description. *Browse updates* shows the
  history of saves.
- **Deployments**: click the `ENVIRONMENT — version` badge in the breadcrumb: each
  deployment with **Update version** and undeploy, plus **Deploy on a new environment**.
  The deployment switcher in the toolbar jumps between a flow's deployments.
- **Variables**: Environments › the environment › **Variables** (key / value). **Not for
  secrets**: their values appear in run data; keep keys and passwords in credentials. May be
  switched off on some platform deployments.
- A deployment pinned to a named version opens **Locked**: only latest can be edited.
  Unlocking a pinned deployment only lets you change its credentials.
- If a version you just created is missing from a dialog, reload the page.

## Promoting a flow to another environment

1. Put everything that differs between environments, or that can't be edited once
   frozen, in **environment variables**: base URLs, labels, recipients, thresholds,
   credential names for lookups. Create the same keys in the target **first**.
2. **Create a named version** of the flow.
3. **Deploy on a new environment** (or *Update version* on the existing target deployment)
   with that version — **not latest**: a target on latest picks up every edit made in the
   source at its next trigger.
4. **Choose each step's credential** on the target deployment before the first run (see
   [credentials.md](credentials.md#before-the-first-run-in-a-new-environment)).
5. Trigger it once and check the run.

To ship a change: new version, then *Update version*. To roll back: *Update version* to
the previous one, or *Restore this version* (below).

**A missing variable fails late and quietly**: its pill comes out empty (shown greyed
with "Failed to evaluate pill" in the designer) and a later step fails with an unrelated
error. Add a guard for critical variables. Changing a variable needs no redeploy.

## Restore this version

- Runs at once, **without confirmation**. It first saves the current draft as a version
  called **Autosave**, so you can go back.
- Keeps step identities and redeploys at once.
- **Doesn't restore credentials**: the deployment keeps its current ones, which may not
  suit the restored steps. It also resets the flow's email notifications. Re-check both.
- Reload the page before editing again.

## Lock

The toolbar's Locked / Unlocked switch prevents accidental edits in the Studio: every
field is greyed out. It doesn't stop runs, edits through the API, or the redeploy of
"latest". To protect what runs, deploy a named version.

## Copy, paste, duplicate

- **Duplicate** (flow card menu) copies a flow under a new name.
- **Copy** (designer toolbar) puts the flow on the clipboard as JSON (without
  credentials). **Paste**: Flows list › ⋮ › *Paste flow…*, with a name and a default
  environment. The new flow has new step identities, no credentials and no version
  history, deploys itself within about half a minute, and opens Locked. This is how to
  clone a flow into another environment; then choose credentials.
- The add-step menu can also paste copied steps into an existing flow.
- **Templates** (sidebar) opens Mindflow's template gallery in a new tab; nothing is
  installed until you import it.

## Flow permissions

The avatar button in the designer toolbar: **owner** (edit, delete, manage permissions),
**editor** (edit; can't delete, manage permissions or change credentials), **viewer**
(view and read logs).

## Stored data (Artifacts)

Each environment has its own Artifacts store, so a flow promoted to another environment
starts with an empty one. See [patterns.md](patterns.md#storing-data-artifacts).
