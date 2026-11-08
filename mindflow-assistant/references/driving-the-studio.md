# Operating the Studio in a browser

Read this only when you can control a browser and the user has asked you to act in the
Studio for them. Otherwise, guide them with click paths.

## Ground rules

- Work only in the environments and flows the user names. Confirm before any run,
  deploy, delete, restore, or change in a shared or production environment.
- Never type, read back or copy secrets. Prepare a credential (service, name, URL, type,
  header name) and let the user enter the secret and click Create.
- Don't change members or permissions.
- Keep a list of everything you create (flows, credentials, variables), give throwaway
  objects a recognisable prefix, and offer to delete them at the end. Never delete what
  you only opened.
- If sign-in works but the Studio shows "Current user not found" or a no-organization
  page, the browser is probably being blocked by the site's bot protection: stop and
  tell the user.
- Tell the user what you changed, step by step, at the end.

## Clicking reliably

- **Target elements, not pixels.** Prefer role, label or visible text, then
  `data-testid` (named `Component-Element`; e.g. `button-autolayout`,
  `button-zoom-to-fit`, `StepViewContainer-Widgets-AddStepButton` — the add button of
  one step, shown on hover), then coordinates as a last resort.
- **Wait for a settled screen** before acting on or checking something that just
  changed: no spinner on step cards, no grey skeleton placeholders, no half-open panel or
  dialog. After typing in a field, give it about half a second to save before moving on.
  A fixed pause alone isn't enough; check the loading indicators are gone.
- **Lay out the designer first**: collapse the app sidebar, run auto-layout, then
  zoom-to-fit once. Don't keep zooming out: tiny labels cause misclicks. Unlock the flow
  if the toolbar shows Locked.
- Take a fresh screenshot before each coordinate click: the canvas re-centres, the open
  step panel shifts it, and drags move further than expected and close the panel.

## Editing fields

- Default text such as "property 1" is real text: select it before typing.
- Confirm a key name with **Tab** before clicking fx, or the key is lost.
- Some rich-text fields ignore select-all: move to the end (cmd/ctrl + Down) and delete.
- Number fields: triple-click, delete, type, Tab.
- Branch names are multi-line: Return adds a new line.
- After inserting a pill, press Escape. A `/` typed at the end of a value opens the
  picker and is swallowed.
- **Re-open the step after editing and check every field**: values can be lost without
  warning. If you can read the flow through Mindflow's API, compare it with what you
  set.

## When the page misbehaves

- "Failed to do Create step", unresponsive buttons, results that don't refresh: reload
  the designer and retry. Check the list before retrying a Create that seemed to do
  nothing — it may have worked.
- A version just created is missing from a dialog: reload.
- After *Restore this version*, reload before editing.
- "data is undefined" when viewing a step's run: go back and reopen it.
- In Rooms, the `@` mention menu may need a cursor move after typing `@name`.
