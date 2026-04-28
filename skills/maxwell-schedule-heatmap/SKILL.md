---
name: maxwell-schedule-heatmap
description: >
  Generate a fully interactive HTML scheduling heat map for the Maxwell School
  of Citizenship and Public Affairs at Syracuse University. Takes a
  PeopleSoft/SIS course schedule Excel export and produces a self-contained
  HTML artifact with a department filter, day × time-block heat map grid,
  hover tooltips showing course lists, enrollment color-coding, summary stats,
  and a sortable course data table. Use this skill whenever asked to: build a
  scheduling heat map, visualize Maxwell course density, show instructional
  load by day and time, analyze scheduling patterns across departments, produce
  a chair-ready scheduling overview, or explore when Maxwell courses meet. Even
  if the user just says "make a heat map of our schedule" or "show me the
  scheduling density" — use this skill.
license: Apache-2.0
---

# Maxwell Scheduling Heat Map

Produces a single, fully interactive HTML artifact — a scheduling heat map —
that a department chair can use live in Claude to explore instructional density
across days and time blocks, filtered by department.

## Inputs

- A Maxwell School course schedule Excel export from IDR or PeopleSoft
  (.xlsx). The user attaches this file.
- Optionally: a target semester label (e.g., "Spring 2026") for the artifact
  title.

## SU Standard Time Blocks

Map every section's meeting time to the nearest block below. Sections that do
not fit any block go in a **Non-Standard** catch-all row — never drop them.

| Pattern | Blocks |
|---------|--------|
| MW | 8:00–9:20, 9:30–10:50, 11:00–12:20, 12:30–1:50, 2:00–3:20, 3:30–4:50, 5:00–6:20, 6:30–7:50 |
| TTh | 8:00–9:20, 9:30–10:50, 11:00–12:20, 12:30–1:50, 2:00–3:20, 3:30–4:50, 5:00–6:20, 6:30–7:50 |
| MWF | 8:00–8:55, 9:05–10:00, 10:10–11:05, 11:15–12:10, 12:20–1:15, 1:25–2:20, 2:30–3:25 |

## Step-by-Step Instructions

### Step 1 — Parse the Excel file

Read the .xlsx with pandas. Identify and map columns to:

- Department code (e.g., PAI, PSC, SOC, ECN, GEO)
- Course number and section
- Days of week (M, T, W, Th, F)
- Start time / End time (or time block code)
- Enrolled students (actual enrollment, not capacity)
- Cross-listing indicator or cross-listed course codes (if present)
- Class Nbr (used for deduplication)

**Deduplication**: Rows sharing a Class Nbr represent co-instructors or split
meeting times — not independent sections. Deduplicate on Class Nbr before
analysis, keeping one row per unique section. Rows with null Class Nbr are
continuation rows; drop them.

**Exclusions**: Filter out Independent Study, Dissertation, and Thesis
sections by keyword-matching on Class Description (case-insensitive: "independent
study", "dissertation", "thesis", "ind study").

### Step 2 — Normalize time blocks

Map each section's meeting days + start time to the closest SU standard time
block. Use start time as the anchor; ignore end time for block assignment.
Flag any section that does not map cleanly → place in **Non-Standard** row.

### Step 3 — Handle cross-listed courses

If a section is cross-listed (appears under multiple department codes):
- Include it in each department's filtered view.
- In the Maxwell-wide ("All Maxwell") view, count it **once** under the home
  department — do not double-count enrollment totals.

### Step 4 — Aggregate by cell

For each (day, time-block) cell compute:
- **Section count**: number of unique sections meeting in that slot
- **Total enrollment**: sum of enrolled students

### Step 5 — Build the HTML artifact

Embed all data as a JSON blob inside a `<script>` tag. No external API calls.
CDN libraries (e.g., from cdnjs.cloudflare.com) are allowed.

**Required components:**

| Component | Spec |
|-----------|------|
| Filter bar | Dropdown: "All Maxwell" + each dept code found in data. Updates everything instantly on change. |
| Heat map grid | Rows = time blocks (Non-Standard last); Columns = M T W Th F. Color: white (0) → #F76900 (max density). Each cell shows `[N sections] / [E enrolled]`. |
| Hover tooltip | List of course numbers + titles meeting in that cell for the active filter. |
| Summary stats bar | Total sections, total enrolled, busiest slot, emptiest non-zero slot — for current filter. |
| Color scale legend | Min → max range. |
| Data table toggle | Collapsible table of all courses in current filter. Sortable by day/time, enrollment, dept. Enrollment color-coded to MMCE thresholds: 🔴 < 4, 🟡 4–9, 🟢 ≥ 10. |

**Design system:**

- Primary orange: `#F76900`
- Dark navy: `#002147`
- Background gray: `#F4F4F4`
- Font: IBM Plex Sans (via CDN) or system sans-serif fallback
- Fully self-contained — no external dependencies beyond CDN-hosted libraries

### Step 6 — Verify before rendering

Before writing the final HTML:
- ✅ All departments in the data appear in the dropdown
- ✅ Cross-listed courses appear in both relevant department filters
- ✅ No sections silently dropped (non-standard times → Non-Standard row)
- ✅ Enrollment totals in summary bar match deduplicated section count for the
  selected filter
- ✅ MMCE enrollment color-coding applied in data table

## Output

1. **One self-contained HTML artifact** — fully functional, no further input
   needed.
2. **Plain-text Data Notes** (outside the artifact) listing:
   - Time blocks that could not be mapped (Non-Standard)
   - Departments found in the data
   - Cross-listed course count
   - Any data quality flags (missing enrollment, unmapped columns, etc.)

## Quality Bar

The artifact meets the bar if:
- Filtering by department updates the heat map instantly with no page reload
- Hovering a cell shows the actual course list for that slot
- Cross-listed courses appear in both departments' filtered views
- Non-standard meeting times are visible, not dropped
- Summary stats are accurate for the selected filter
- A department chair with no technical training can interpret it in under
  60 seconds

## Reference Files

- `references/su-time-blocks.md` — Full SU time block reference with edge
  cases and mapping notes. Read this if you encounter ambiguous meeting times.
- `references/mmce-thresholds.md` — MMCE enrollment threshold rules for
  color-coding in the data table.
