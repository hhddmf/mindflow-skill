# Missing, null and empty values, and large data

Measured in a lab flow: each case below was run through the Studio's pills, fx functions
and Compare tests.

## What a pill gives you

With the pill's **Required** off:

| The data is… | The pill gives | *Is empty* | Test that works |
|---|---|---|---|
| A key that is **missing** | empty | ✅ true | *Is empty* |
| An empty string `""` | empty | ✅ true | *Is empty* |
| **`null`** (also a nested null) | the text **`null`** | ❌ false | *Is equal to* `null` |
| `0` | `0` | ❌ false | *Is equal to* `0` |
| `false` | `false` | ❌ false | *Is equal to* `false` |
| An empty list `[]` | `[]` | ❌ false | *Is equal to* `[]` (or *Array length* = 0) |
| An empty object `{}` | `{}` | ❌ false | *Is equal to* `{}` |

**Is empty only catches missing keys and empty strings.** To treat "missing or null" the
same way, put two tests in the branch (*Is empty* and *Is equal to* `null`) joined with
**Or**.

## Required on or off

- **Required on** fails the run only when the key is **missing**: "Data referenced by
  **x** was not present in the webhook payload" (or "…was not found" for step data). A
  key that is present with a `null` value **passes**, and the step receives the text
  `null`.
- **Required off** lets the run continue with an empty value. Use it whenever absence is
  a normal case, and branch on it explicitly:
  - optional fields in a payload;
  - a failed or timed-out step's `code` (with On failure = Continue);
  - GraphQL `errors` (absent on success);
  - an approval nobody answered;
  - a sub-flow output when the sub-flow failed;
  - anything read by a Condition that must route the "nothing there" case.
- Keep Required **on** where missing data is a real error and should stop the run with a
  clear message, rather than flowing on as an empty value.

## fx functions on missing or null data

| Function | Situation | Result |
|---|---|---|
| **JSON Path**, Wrap **off** | the path matches nothing | ⚠️ **the step crashes**: "Failed to evaluate function JSON Path… Cannot read properties of undefined (reading 'startsWith')" |
| JSON Path, Wrap **on** | the path matches nothing | `[]` → test *Is equal to* `[]` |
| JSON Path | the path points at a `null` value | the text `null` |
| **JSONata** | the expression matches nothing | the text **`undefined`** (*Is empty* is false) |
| JSONata | the expression points at `null` | the text `null` |
| JSONata with a default | `$exists(x) ? x : ''` | `''` → *Is empty* works |

Rules of thumb:
- **If a path may be absent, use JSONata with a default** (below): one function, no
  crash, and *Is empty* works on the result.
- If you prefer JSON Path, turn **Wrap on** and test the result with *Is equal to* `[]`
  first. Only extract the value (a second JSON Path `$[0]`) in the branch where the
  result is **not** `[]`: `$[0]` on `[]` matches nothing and crashes again.
- **Give JSONata a default** so the result is predictable: `$exists(field) ? field : ''`
  for "empty if absent", or `field ? field : 'none'` to also replace null, 0 and false.
- If you don't set a default, test JSONata results with *Is equal to* `undefined`.
- The run log shows the error "Cannot read properties of undefined (reading
  'startsWith')" when a JSON Path didn't match: check the path against the real data
  (e.g. a field that exists in one API version but not another).

## Large data (hundreds of KB and more)

**At run time:** JSONata, JSON Path, Array length and Array slice all worked on inputs of
**1.3 MB** (9,000 items), and JSONata produced outputs of about 440 KB. Large data passed between steps is handled automatically; flows don't need to do
anything special.

**In the Studio's fx preview:** above the preview limit, **JSONata is skipped**: "This
function was skipped. JSONata could not be evaluated because the input data exceeds the
preview limit. The output displayed is from the previous step." The preview then shows
the input instead of the result, and you can't pick fields from the result.

Workaround while designing:
1. Put **Array slice** (Start 0, End 100, for example) in front of JSONata in the
   pipeline. The preview now evaluates on a small sample.
2. Write and check the JSONata query; pick the fields you need downstream.
3. Remove the Array slice (or widen it) so real runs process everything.
   For a large **object** rather than a list, reduce it first with a JSON Path to the
   part you need.

Other options: narrow the data upstream (ask the API for fewer fields or pages), or do
heavy transformations in a **Run script**, which isn't affected by the preview limit.

Only JSONata is skipped in the preview; the other fx functions preview normally on large
inputs.
