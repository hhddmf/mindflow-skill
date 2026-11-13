# Excel Workbook Format Spec

## Brand Colors
- Header background: Mindflow blue `#1A56DB`
- Header text: white, bold
- Alternating rows: light gray `#F3F4F6` / white
- Font: Calibri 10pt body, 11pt headers

## Global Rules
- Freeze row 1 (headers) on all data tabs
- Auto-fit column widths, minimum readable width
- Wrap text enabled, vertical alignment top
- Thin light gray grid lines throughout (`#D1D5DB`)

---

## 2-Sheet Workbook Structure

| Sheet | Name | Audience |
|-------|------|----------|
| 1 | Executive Summary | Sponsors, non-technical stakeholders |
| 2 | Use Cases & Flows | Business + technical (layered via row grouping) |

---

## Tab 1: Executive Summary

Formatted text sheet. Two columns: A (labels, width 26) and B (values, width 88).

### Layout (top to bottom)

**Title block (rows 1–3)**
- Row 1: "MINDFLOW" — Calibri 22pt bold, brand blue `#1A56DB`
- Row 2: "Automation Business Case — 24 Month Plan" — Calibri 13pt, color `#374151`
- Row 3: "Prepared for: [Client] | [Date]" — Calibri 10pt bold

**Value Story (rows 5–6)**
- Row 5: Section header "Value Story" — 11pt bold brand blue, background `#EFF6FF`
- Row 6: Merged A:B, background `#EFF6FF`, height ~110px
  - 4 sentences structured as:
    1. Where the team is today — maturity level + 1 key challenge in their own words
    2. What Mindflow enables — 2–3 exact tool names + outcomes unlocked
    3. What that's worth — quantified, tied to Q3B volumes and Q4B KPIs (use "x" where data is missing)
    4. How the phasing aligns to their timeline or initiative
  - **Banned words:** transformative, cutting-edge, end-to-end, seamlessly, robust, holistic, game-changing, leverage, synergy, paradigm, unlock, empower. If a sentence could apply to any company, rewrite it.

**Executive Summary (rows 8–9)**
- Row 8: Section header "Executive Summary" — 11pt bold brand blue
- Row 9: Merged A:B, height ~130px
  - 3 paragraphs:
    1. Scope statement — number of Use Cases and flows
    2. Context — what's driving the initiative and what's changing in their stack
    3. Phase arc — brief description of each phase and what it delivers

**Discovery Inputs (rows 11–17)**
- Row 11: Section header "Discovery Inputs" — 11pt bold brand blue
- Rows 12–17: 2-column key/value rows (label bold in col A, value in col B), height 30px each:
  - Areas of focus
  - Tools in scope (exact product names)
  - Automation maturity
  - Team & scale
  - Key challenges
  - Driving initiative

**Priority Order (rows 19–31)**
- Row 19: Section header "Priority Order — What to Build First" — background `#F0FDF4`
- Row 20: Column headers — white on brand blue: UC# | Use Case | Phase | Why first
- Rows 21–30: One row per Use Case (10 rows), colour-coded by phase:
  - Phase 0 — POS: `#FEE2E2`
  - Phase 1 — Foundation: `#D1FAE5`
  - Phase 2 — Scale: `#FEF3C7`
  - Phase 3 — Intelligence: `#DBEAFE`
- Column widths for Priority Order: A=8, B=50, C=24, D=56

**Recommended Next Step (rows 32–33)**
- Row 32: Section header
- Row 33: Merged A:B, height ~70px — 2 paragraphs:
  1. Specific first action calibrated to maturity (which Use Cases to start with and why)
  2. Close: *"Mindflow's 4,000+ native integrations and autonomous AI agents mean this roadmap plugs into your current stack as-is — no rip-and-replace, no rebuilding what already works."*

---

## Tab 2: Use Cases & Flows

### Purpose
One sheet serving two audiences via Excel row grouping (outline):
- **Default view (collapsed):** 10–12 UC rows — clean business conversation
- **Expanded view:** Flow sub-rows visible below each UC — technical detail on demand

### Column Structure (8 columns, same for UC rows and flow sub-rows)

| Col | Width | UC Row Content | Flow Sub-Row Content |
|-----|-------|---------------|---------------------|
| A | 9 | UC-01, UC-02… | FL-01, FL-02… |
| B | 50 | Use Case (outcome-framed name) | Flow Name (technical, step-by-step) |
| C | 12 | Category (Q1 area) | Category |
| D | 42 | Business Challenge (client's own words from Q4A) | Trigger (what initiates the flow) |
| E | 40 | Benefits (what life looks like after — 2 sentences max, no jargon) | Orchestrated Tools (all tools, exact names, comma-separated) |
| F | 38 | Projected ROI (specific, tied to Q3B; use "x" for unknowns) | Flow Description (2–3 sentences, full chain) |
| G | 36 | KPI Impacted (from Q4B) | Inputs (what data enters the flow) |
| H | 22 | Phase | Outputs (what the flow produces) |

### UC Row Styling
- Background: phase colour (Phase 0 = `#FEE2E2`, Phase 1 = `#D1FAE5`, Phase 2 = `#FEF3C7`, Phase 3 = `#DBEAFE`)
- Font: Calibri 10pt; column A and B bold
- Border: thin `#D1D5DB`
- Row height: ~52px

### Flow Sub-Header Row (inserted between UC row and its flow rows)
- One row immediately after the UC row, before the first flow
- Background: `#E0E7FF` (indigo-100)
- Content: blank | Flow Name | Trigger | Orchestrated Tools | Description | Inputs | Outputs | Phase
- Font: Calibri 9pt bold, color `#374151`
- Row height: 16px
- **Grouped and collapsed with the flow rows** (same outline level 1, hidden=True by default)

### Flow Sub-Row Styling
- Background: alternating `#F8FAFF` / `#EEF2FF`
- Font: Calibri 9pt, color `#374151`
- Border: thin `#E5E7EB`
- Row height: ~70px
- **Grouped at outline level 1, hidden=True by default** (collapsed)

### Row Grouping Setup
```python
ws.sheet_format.outlineLevelRow = 1
ws.sheet_properties.outlinePr.summaryBelow = False  # expand button ABOVE the group
```
For each UC, after writing the UC row:
1. Write the flow sub-header row
2. Write all flow rows for that UC
3. Set outline_level = 1 and hidden = True on the sub-header row AND all flow rows

### Header Row (row 1)
- Headers: # | Use Case | Category | Business Challenge | Benefits | Projected ROI | KPI Impacted | Phase
- White text on Mindflow blue `#1A56DB`, Calibri 10pt bold, centered
- Freeze row 1: `ws.freeze_panes = "A2"`

### Conditional Formatting (Phase column H)
Applied across all rows (UC + flow sub-rows):
- "Phase 0" in H → background `#FEE2E2`
- "Phase 1" in H → background `#D1FAE5`
- "Phase 2" in H → background `#FEF3C7`
- "Phase 3" in H → background `#DBEAFE`

---

## Content Generation Rules

### Use Cases (10–12 total)
- Each UC is a **business outcome**, not a workflow — column B must be outcome-framed
  - ✅ "Cut phishing investigation time from 30 min to under 5 min per report"
  - ❌ "Automated phishing triage flow"
- Benefits (col E): 1–2 sentences. What does the analyst/operator experience after this is running? No jargon. No "the system will automatically..." — frame from the human perspective.
- Projected ROI (col F): Use "x" where volumes from Q3B are unknown. Always show the formula: "x alerts/month × y min saved". Never invent numbers.
- Every Q4A challenge must be addressed by at least one UC
- Phase distribution: 2 in Phase 0, 4 in Phase 1, 3–4 in Phase 2, 1–2 in Phase 3 (adjust for maturity)

### Flows (20–30 total, 2–4 per UC)
- Flow Name (col B): technical, step-by-step — tool chain readable from the name alone
- Trigger (col D): specific — "ZIIVE webhook on new alert creation", not "when alert arrives"
- Orchestrated Tools (col E): ALL tools in the full chain, exact product names, comma-separated
- Flow Description (col F): 2–3 sentences describing every step. Full chain: source → enrichment → action → notification
- Inputs (col G): specific data fields — "ZIIVE alert payload (title, severity, affected user)", not "alert data"
- Outputs (col H): concrete artifacts — "AI verdict + confidence score written to ZIIVE ticket comment", not "results"
- Every Q2 tool must appear in at least one flow
- ≥30% of flows must use Mindflow AI Task or Mindflow Rooms

### Exact product names everywhere
Never use vendor names alone. Examples:
- "CrowdStrike Falcon" not "CrowdStrike"
- "Proofpoint TAP" not "Proofpoint"
- "Cortex (StrangeBee)" not "Cortex"
- "Google Workspace (Gmail)" not "Google" or "Gmail"
- "Microsoft Defender for Office 365" not "Defender"
Apply in every cell — UC rows, flow rows, and the Executive Summary tab.

---

## Flow Technical Precision

Before writing any flow row, read `references/mindflow-capabilities.md`. Flow descriptions must reference actual Mindflow primitives — not generic descriptions.

### Required for every flow row

**Trigger (col D):** Must name the exact Mindflow trigger type plus the source event. Examples:
- ✅ "Webhook trigger (ZIIVE new alert creation event)"
- ✅ "Scheduler trigger (every 5 min poll)"
- ✅ "Chathook trigger (analyst message in Mindflow Rooms)"
- ❌ "When ZIIVE fires" / "On alert arrival" / "Triggered by the system"

**Flow Description (col F):** Must name Mindflow steps in the chain. The reader should be able to build this flow from the description. Examples:
- ✅ "A **Condition step** checks alert severity against the threshold. If above, a **Webhook response step** acknowledges. If below, an **AI task step** generates a Teams message; a **Send email step** is used for the fallback after a **Wait step** (3,600 s)."
- ❌ "The system checks severity, then AI generates a message and sends it."

### Step selection rules

| Scenario | Use |
|----------|-----|
| Single AI call (classify, extract, draft, assess) | **AI task step** |
| Multi-tool reasoning, pivoting across systems | **Agent step** + Agentic flow tools |
| Analyst interactive chat in Rooms | **Chathook trigger** + **Agent step** |
| Reusable logic called from multiple flows | **Flowhook trigger** + **Call a flow step** |
| Loop over a list (IOCs, alerts, mailboxes) | **For each step** |
| Timed pause (escalation wait, rate limiting) | **Wait step** |
| Branching logic (severity, confidence, tool type) | **Condition step** |
| Data reshaping between steps | **Transform data step** |
| API without catalog integration | **HTTP request step** (Built in) |
| Persist data between runs | **Platform Artifacts** (put/get artifact steps) |
