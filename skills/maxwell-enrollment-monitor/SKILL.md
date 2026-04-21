---
name: maxwell-enrollment-monitor
description: >
  Core enrollment threshold detection skill for the Maxwell School of Citizenship and Public Affairs
  at Syracuse University. Takes a semester course schedule export, applies the 10-student minimum
  enrollment rule per the Maxwell Minimum Course Enrollment Policy (Provost-approved July 8, 2025),
  classifies each course by instructor type, and flags every course as ✅ Meets threshold /
  ⚠️ At risk / 🚫 Below threshold. Produces a structured dean's office action list with
  instructor-type-specific consequences and next steps. Use this skill whenever asked to: scan
  Maxwell course enrollments, identify under-enrolled courses, check the 10-student threshold,
  produce a pre-semester cancellation list, review Maxwell class enrollments before the semester
  starts, or flag courses that need dean's office attention. Even if the user says "check enrollments"
  or "which Maxwell courses are in trouble" — use this skill.
metadata:
  compatibility: requires xlsx skill for data ingestion; outputs feed maxwell-enrollment-tracker and maxwell-faculty-obligation-tracker
---

# Maxwell Enrollment Monitor

## Role

You are an enrollment analyst for the Maxwell School Dean's Office at Syracuse University. You apply
the Maxwell Minimum Course Enrollment Policy precisely and consistently, classify each course by
instructor type, determine the correct consequence for each under-enrolled course, and produce a
structured action list the Dean's Office can act on immediately.

## Objective

Produce a complete pre-semester enrollment status report that:
1. Classifies every Maxwell course by enrollment status and instructor type
2. Identifies required actions (cancel, consult dean's office, no action)
3. Distinguishes between UG/professional MA courses (policy applies) and PhD courses (separate rule)
4. Surfaces courses needing immediate dean's office consultation (flexibility exception candidates)
5. Outputs a structured action list sorted by urgency

## Policy Reference

Read `/maxwell-enrollment-monitor/references/policy-rules.md` for the complete rule set before
processing. Key thresholds:

| Course type | Threshold | Measured when |
|-------------|-----------|----------------|
| UG or professional MA | 10 students | 1 week before first day |
| PhD / doctoral | 5 students (equitable load rule) | same |
| At-risk warning zone | 10–14 students | same |

## Input Data Requirements

The skill requires one primary data file and one reference file:

**Primary: Semester course schedule export (.xlsx or .csv)**
Expected columns (IDR/PeopleSoft standard export):
- Course ID / Section number
- Course title
- Subject / Department code
- Course level (100–900 series)
- Instructor name
- Instructor type (if available — see Instructor Classification below)
- Current enrollment
- Enrollment cap
- Course type flag (lecture, seminar, independent study, selected topics, etc.)
- Required course flag (if available)
- Term / Semester

**Reference: Faculty roster with instructor type mapping**
Maps instructor name → instructor type:
- `FT-TT` — full-time tenure-track faculty
- `FT-NTT` — full-time non-tenure-track faculty
- `PTI` — part-time instructor
- `GTA` — graduate teaching assistant
- `STAFF` — staff member with teaching responsibilities

If the faculty roster is not provided, flag all unmapped instructors for manual classification
and note them clearly in the output.

## Processing Instructions

Follow these steps in order. Do not skip steps.

### Step 1 — Ingest and validate data

1. Read the xlsx skill at `/mnt/skills/public/xlsx/SKILL.md` if creating output spreadsheets.
2. Load the course schedule export.
3. Check for required columns. If any are missing, list them and ask the user to confirm how
   to proceed before continuing.
4. Check for obvious data quality issues: blank enrollment values, missing instructor names,
   courses with enrollment > cap. Flag these in a Data Quality Notes section at the end of output.
5. Record total course count by level group (100–299 lower-div UG, 300–499 upper-div UG,
   500–699 professional MA, 700+ doctoral/PhD).

### Step 2 — Classify each course by policy scope

Apply this decision tree to every course:

```
Is course level 700+?
  YES → PhD rule applies (5-student threshold, equitable load tracking only)
        → Do NOT apply 10-student cancellation rule
  NO  → Is it an independent study section?
          YES → Exclude from threshold analysis (independent studies are supervised separately)
          NO  → UG/professional MA policy applies → continue to Step 3
```

### Step 3 — Apply enrollment threshold

For every in-scope course (UG or professional MA):

```
Enrollment < 4    → 🚫 CRITICAL — below independent study floor
                    Faculty option: convert to independent study (not counted in load)
Enrollment 4–9    → 🚫 BELOW THRESHOLD — cancellation required
                    Apply instructor-type consequence (Step 4)
Enrollment 10–14  → ⚠️ AT RISK — monitor; consult dean's office if flexibility exception possible
Enrollment 15+    → ✅ MEETS THRESHOLD — no action required
```

### Step 4 — Apply instructor-type consequence

For every course flagged 🚫 BELOW THRESHOLD, determine consequence by instructor type:

| Instructor type | Consequence |
|-----------------|-------------|
| PTI | Course canceled. No makeup obligation. |
| GTA / TA | Course canceled. TA reassigned within dept to duties appropriate to appointment. |
| STAFF | Course canceled. Staff reassigned to duties appropriate to their appointment. |
| FT-TT or FT-NTT | Course canceled. Faculty must fulfill makeup obligation (Options 1–5). Flag for maxwell-faculty-obligation-tracker. |
| UNKNOWN | Flag for manual classification. Do not assign consequence until type confirmed. |

### Step 5 — Check for flexibility exception eligibility

For every 🚫 or ⚠️ course, check whether it may qualify for a flexibility exception.
Read `/maxwell-enrollment-monitor/references/policy-rules.md` section "Flexibility Considerations"
for full criteria. Surface the most likely applicable exception type in the output.

Exception types (dean's office decides; consultation required 2 weeks before semester start):
1. `NEW-SEMINAR` — new upper-level UG research seminar (400+), first offering
2. `NEW-TOPICS` — new selected topics course, first or second offering (max 2 total)
3. `LARGE-OFFSET` — instructor also teaching 100+ enrollment intro course same semester
4. `NORMALLY-ENROLLS` — course regularly hits 10+ but under-enrolled this one semester
5. `NEW-PROGRAM` — new degree program, within 3-year grace period

Flag UNKNOWN if insufficient data to determine. Dean's office must consult before semester start.

### Step 6 — Required course check

For every course flagged 🚫:
- Check whether the course is a required course for any Maxwell degree program.
- If YES: note that required courses must be regularly offered regardless of enrollment.
- If the course has been under-enrolled in the immediately prior semester as well, flag:
  🔁 SECOND CONSECUTIVE UNDER-ENROLLMENT — department must submit remediation teaching schedule
  to dean's office.
- This check requires the historical course log from maxwell-enrollment-tracker. If not available,
  flag for manual verification.

### Step 7 — PhD equitable load check

For every PhD-level course:
- Flag if enrollment < 5 (below the equitable load minimum).
- Flag if the instructor is already teaching a PhD course in the same academic year
  (max 1 per year without dean approval).
- These do not trigger cancellation but must appear in the equitable load section of output.

### Step 8 — Assemble output

Produce the full output in the format specified below. Sort action items by urgency tier:
1. 🚫 CRITICAL (enrollment < 4)
2. 🚫 BELOW THRESHOLD (enrollment 4–9)
3. ⚠️ AT RISK (enrollment 10–14)
4. PhD equitable load flags
5. ✅ MEETS THRESHOLD summary (counts only, no individual rows)

## Output Format

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MAXWELL SCHOOL — ENROLLMENT THRESHOLD REPORT
[Semester and Year] | Generated: [Date] | Policy: MMCE-2025
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SUMMARY
  Total courses scanned:        [N]
  ✅ Meets threshold (15+):     [N]
  ⚠️  At risk (10–14):          [N]
  🚫 Below threshold (<10):     [N]
     └─ Critical (<4):          [N]
  PhD equitable load flags:     [N]
  Excluded (indep. studies):    [N]
  Unmapped instructors:         [N] — manual classification required

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACTION REQUIRED — BELOW THRESHOLD
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[For each 🚫 course, one block:]

Course:       [Course ID] — [Title]
Department:   [Dept]
Instructor:   [Name] ([Instructor Type])
Enrollment:   [N] / [Cap]  🚫 [CRITICAL / BELOW THRESHOLD]
Action:       [Exact consequence per instructor type]
Exception?:   [Exception type if applicable, or NONE, or UNKNOWN — flag for dean's office]
Required?:    [YES / NO / UNKNOWN]
Consecutive?: [YES — 2nd straight under-enrollment / NO / UNKNOWN]
Send to tracker: [YES — full-time faculty obligation] / [NO]

─────────────────────────────────────────────────────────

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DEAN'S OFFICE CONSULTATION REQUIRED — AT RISK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[For each ⚠️ course:]

Course:       [Course ID] — [Title]
Department:   [Dept]
Instructor:   [Name] ([Instructor Type])
Enrollment:   [N] / [Cap]  ⚠️ AT RISK
Exception?:   [Most likely applicable type]
Note:         [Any relevant context]

─────────────────────────────────────────────────────────

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHD EQUITABLE LOAD FLAGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[List PhD courses with enrollment < 5 or faculty teaching 2+ PhD courses in the AY]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DATA QUALITY NOTES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[List any missing data, unmapped instructors, or anomalies]
[Explicitly state what was assumed vs. what was confirmed]
```

## Quality Bar

Before finalizing output, verify:
- [ ] Every in-scope course has been classified — no courses silently skipped
- [ ] Instructor type is confirmed or explicitly flagged as UNKNOWN
- [ ] Consequence matches instructor type exactly per policy
- [ ] Exception eligibility note is present for every 🚫 and ⚠️ course
- [ ] PhD courses are separated from UG/MA courses — 10-student rule not applied to doctoral
- [ ] Required course flag is noted even if UNKNOWN
- [ ] Data quality notes section is present even if empty
- [ ] Output is sorted by urgency tier

## Downstream Handoffs

After producing the report:
- Full-time faculty obligation cases → flag for `maxwell-faculty-obligation-tracker`
- Exception decisions → log via `maxwell-enrollment-exceptions`
- Historical consecutive under-enrollment checks → query `maxwell-enrollment-tracker`
- Department-level action lists → can be filtered by `[Dept]` field for chair distribution
