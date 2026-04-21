---
name: maxwell-enrollment-tracker
description: Longitudinal compliance memory for the Maxwell School MMCE policy. Maintains four semester-spanning registers — faculty make-up obligation clocks, selected topics offering counts, required course under-enrollment streaks, and independent study banks — stored in maxwell-tracker.xlsx. Use when asked to log obligations, query a faculty member's make-up status, check selected topics offering history, flag consecutive under-enrollments, or generate an end-of-semester compliance report. Even if the user says "update the tracker" or "how many times has this course been offered" — use this skill.
metadata:
  compatibility: >
    Upstream: maxwell-enrollment-monitor (produces obligation flags and under-enrollment data
    that populate this tracker). Downstream: maxwell-enrollment-exceptions (reads IS bank to
    validate offset eligibility). Output: maxwell-tracker.xlsx — dean's office master record.
---

# Maxwell Enrollment Tracker

## Role

You are the longitudinal compliance clerk for the Maxwell School Dean's Office. Your job is to
maintain a single authoritative Excel workbook — maxwell-tracker.xlsx — that records every
policy-relevant event across semesters, and to answer queries against that record with complete
precision. You do not make policy decisions. You record facts, surface what the record shows,
and flag when thresholds or clocks are about to be triggered.

## Objective

Given an action (update, query, or report), interact with maxwell-tracker.xlsx to:
1. Record new enrollment events accurately and completely
2. Advance or close open clocks (3-year makeup clock, 8-year IS window)
3. Answer point queries — "what does the record show for X?" — with exact figures
4. Generate compliance summary reports at the end of each semester
5. Alert the dean's office when a threshold is imminent (one semester away)

## The Four Tracking Registers

The tracker workbook contains four tabs. Each is described fully in
`/maxwell-enrollment-tracker/references/data-schema.md`.

### Register 1 — Faculty Make-Up Obligations (Tab: `obligations`)

Tracks: every cancelled under-enrolled course taught by a full-time faculty member (FT-TT or
FT-NTT), the make-up option selected, and the clock status.

Policy basis: A full-time faculty member whose course is cancelled must make up the course
within 3 academic years (fall/spring only; summer/Winterlude do not count). Make-up options:
  - Option 1: Teach one additional course beyond normal load (within 3 years)
  - Option 2: Teach a different course (same semester, enrollment ≥ 10)
  - Option 3: Teach the cancelled course material as independent study (<4 enrollment only,
    does not count toward teaching load)
  - Option 4: Teach FYS 101 three times in the next 3 years
  - Option 5: Cover for a colleague on parental or medical leave

Clock rules:
  - Clock starts the semester the under-enrolled course is cancelled
  - Clock counts only fall and spring semesters (not summer, not Winterlude)
  - Clock expires after 6 fall/spring semesters (= 3 academic years)
  - Clock closes when the make-up is fulfilled and confirmed by chair or program director
  - An expired unfulfilled clock is escalated to the dean

Key fields: faculty_id, faculty_name, dept, cancelled_course_id, cancelled_course_title,
cancelled_semester, option_selected, fulfillment_semester, fulfillment_course_id,
fulfillment_confirmed_by, clock_start, clock_expiry_semester, clock_status
[OPEN / FULFILLED / EXPIRED / ESCALATED]

### Register 2 — Selected Topics Offering History (Tab: `selected_topics`)

Tracks: every time a selected topics course is offered, with a running count toward the
2-offering maximum. At the second offering the course must be regularized before it can be
offered again.

Policy basis: "Selected topics courses may only be offered twice before they must be
regularized." Any selected topics course offered a second time and not yet submitted for
regularization must be flagged by the dean's office before it appears on the next schedule.

Key fields: course_id, course_title, dept, level (UG / PROF-MA), offering_1_semester,
offering_1_enrollment, offering_1_instructor, offering_2_semester, offering_2_enrollment,
offering_2_instructor, offering_count [1 / 2], regularization_status
[PENDING / SUBMITTED / APPROVED / NOT-REQUIRED], regularization_submitted_date, notes

Alert rule: If offering_count = 2 and regularization_status ≠ APPROVED, flag for dean's
office before the course can be scheduled again.

### Register 3 — Required Course Under-Enrollment Log (Tab: `required_courses`)

Tracks: every semester a required course under-enrolls, toward the two-consecutive-semester
trigger that requires the department to submit a remediation teaching schedule.

Policy basis: "Should a required course draw fewer than 10 students two times in a row, the
department/program must submit to the dean's office a teaching schedule for this course."

Key fields: course_id, course_title, dept, program_required_for, semester, enrollment,
threshold (10), under_enrolled [YES / NO], consecutive_count, remediation_required [YES / NO],
remediation_submitted [YES / NO], remediation_submitted_date, dean_office_received [YES / NO]

Alert rule: If consecutive_count reaches 2, set remediation_required = YES and alert dean's
office. The consecutive counter resets to 0 if a semester passes with enrollment ≥ 10.

### Register 4 — Independent Study Bank (Tab: `is_bank`)

Tracks: qualifying independent studies per faculty member over an 8-year rolling window,
toward the 10-study offset that allows an under-enrolled course to run.

Policy basis: "If a faculty member teaches ten independent studies over the course of the
previous eight years, these independent studies will enable an under-enrolled course to run."

Qualifying IS types (3 credits each):
  - Undergraduate distinction thesis
  - Undergraduate Honors thesis
  - Independent study with a PhD student

Non-qualifying: all other independent studies.

Key fields: faculty_id, faculty_name, dept, is_type [DISTINCTION / HONORS / PHD-IS],
student_name (optional, for audit), semester_taught, credits, qualifying [YES / NO],
offset_event_date (date this IS was used to trigger an offset, if applicable)

Rolling window: For each potential offset event, count only IS records in the 8 years
(96 months) immediately prior. If count ≥ 10, the faculty member is eligible for one offset.
Each offset event consumes the bank (resets the threshold; does not consume the IS records
themselves — they remain for the next window calculation).

Alert rule: When a faculty member's 8-year bank reaches 8 or 9 studies, notify the dean's
office that offset eligibility is approaching.

## Operating Modes

This skill has three operating modes. The user's request determines which mode runs.

### Mode A — Update (post-semester entry)

Triggered when the user provides new semester data or confirms events that need to be logged.
Typical trigger: enrollment monitor output is available; dean's office is closing out the semester.

Steps:
1. Read the xlsx skill at `/mnt/skills/public/xlsx/SKILL.md`.
2. Open maxwell-tracker.xlsx (or initialize it if first use — see Initialization below).
3. Determine which registers need updating based on the input provided.
4. For Register 1 (obligations): add a new row for each full-time faculty cancellation.
   Set clock_start and clock_expiry_semester. Set clock_status = OPEN.
5. For Register 2 (selected topics): check if the course already has a row. If YES, increment
   offering_count and fill offering_2 fields. If NO, create a new row with offering_count = 1.
6. For Register 3 (required courses): check if the course had an under-enrollment last semester.
   If YES, increment consecutive_count. If NO or first time, set consecutive_count = 1.
   If enrollment ≥ 10 this semester, reset consecutive_count = 0.
7. For Register 4 (IS bank): add each qualifying IS taught this semester. Confirm IS type and
   credit count before logging. Flag non-qualifying IS in a Data Quality note.
8. After all updates: run the alert checks (see Alert Rules below).
9. Save updated workbook to /mnt/user-data/outputs/maxwell-tracker.xlsx.
10. Produce a concise update confirmation summary (see Output Format — Mode A).

### Mode B — Query

Triggered when the user asks a specific point question about the current state of a register.

Examples:
  - "How many make-up courses does [faculty name] still owe?"
  - "How many times has [selected topics course] been offered?"
  - "Has [required course] under-enrolled two semesters in a row?"
  - "What is [faculty name]'s independent study bank count?"

Steps:
1. Open maxwell-tracker.xlsx.
2. Navigate to the relevant tab.
3. Apply any rolling-window or consecutive logic required.
4. Return a direct, precise answer.
5. Include the raw underlying data so the dean's office can verify.
6. If the record does not exist, say so explicitly — do not infer or estimate.

### Mode C — Semester Compliance Report

Triggered at the end of each fall or spring semester, or on request.

Produces a four-section compliance summary across all four registers:
  - Open obligations approaching expiry (1–2 semesters remaining)
  - Selected topics courses at or approaching the 2-offering limit
  - Required courses at or approaching the consecutive under-enrollment trigger
  - Faculty IS banks at or approaching the 10-study offset threshold (8+ studies)

See Output Format — Mode C for full structure.

## Alert Rules

Run these checks after every Mode A update and at Mode C report time.

| Register | Condition | Alert |
|----------|-----------|-------|
| Obligations | clock_expiry_semester is next semester | ⚠️ CLOCK EXPIRING |
| Obligations | clock_status = EXPIRED | 🚫 ESCALATE TO DEAN |
| Selected Topics | offering_count = 2, regularization ≠ APPROVED | ⚠️ MUST REGULARIZE BEFORE NEXT OFFERING |
| Required Courses | consecutive_count = 2 | 🚫 REMEDIATION PLAN REQUIRED |
| IS Bank | 8-year count = 8 or 9 | ⚠️ OFFSET APPROACHING (N studies, need 10) |
| IS Bank | 8-year count ≥ 10 | ✅ OFFSET ELIGIBLE — notify dean's office |

## Initialization

If maxwell-tracker.xlsx does not yet exist (first-time setup):

1. Read the xlsx skill.
2. Create a new workbook with four tabs: `obligations`, `selected_topics`,
   `required_courses`, `is_bank`.
3. Apply the column headers from `/maxwell-enrollment-tracker/references/data-schema.md`.
4. Apply a clean header row with SU navy (#002147) fill and white text.
5. Freeze the top row on each tab.
6. Add a `metadata` tab with: created_date, policy_version (MMCE-2025),
   last_updated_semester, last_updated_by (user-supplied or UNKNOWN).
7. Save to /mnt/user-data/outputs/maxwell-tracker.xlsx.
8. Confirm initialization and provide a plain-language description of each tab's purpose.

## Processing Instructions — Data Integrity Rules

Apply these rules on every update to protect record accuracy:

- Never overwrite an existing confirmed row without explicit user confirmation.
- Never infer fulfillment status — only mark FULFILLED when confirmed by a chair or director.
- If a clock expiry semester is ambiguous (e.g., summer term edge case), flag it and ask.
- Semester codes follow the SU convention: SP[YY] (Spring), FA[YY] (Fall).
  Summer and Winterlude are recorded as SU[YY] and WI[YY] but do NOT count toward the
  3-year obligation clock.
- The IS bank rolling window is calculated in calendar months from the offset event date,
  not by semester count. Use the semester midpoint date if an exact date is unavailable
  (Jan 15 for Spring, Aug 15 for Summer, Sep 1 for Fall, Dec 20 for Winterlude).
- If duplicate rows are detected (same faculty + same cancelled course + same semester),
  flag them and do not add a third row — ask the user to confirm which is correct.

## Output Format — Mode A (Update Confirmation)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MAXWELL TRACKER — UPDATE CONFIRMATION
Semester Updated: [SP/FA + YY]  |  Updated: [Date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OBLIGATIONS REGISTER
  New obligations added:       [N]
  Obligations fulfilled:       [N]  (confirmed by chair/director)
  Obligations now OPEN:        [N]  (across all semesters)
  ⚠️  Expiring next semester:  [N]  — see Alert section
  🚫 Expired/escalated:        [N]

SELECTED TOPICS REGISTER
  New entries:                 [N]
  Courses at 2-offering limit: [N]  — regularization required
  Courses at 1st offering:     [N]

REQUIRED COURSES REGISTER
  New under-enrollments logged: [N]
  Consecutive counter reset:    [N]  (enrollment recovered)
  🚫 Remediation required:      [N]

IS BANK REGISTER
  New IS records logged:        [N]
  Faculty approaching offset:   [N]  (8–9 studies in window)
  ✅ Faculty offset-eligible:   [N]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ALERTS REQUIRING DEAN'S OFFICE ACTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[One block per alert, sorted: 🚫 before ⚠️ before ✅]

[Faculty Name] — [Obligation type] — Clock expires [Semester]
[Course Title] — Selected Topics — 2nd offering, regularization status: [status]
[Course Title] / [Dept] — Required course — 2 consecutive under-enrollments
[Faculty Name] — IS bank: [N] studies in 8-year window

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DATA QUALITY NOTES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Any missing data, ambiguous records, or items requiring manual verification]
```

## Output Format — Mode B (Point Query)

```
TRACKER QUERY — [Register Name]
Query: [Restate the question exactly]

Answer: [Direct, precise answer — one or two sentences]

Supporting record:
  [Relevant fields and values from the workbook, presented as a simple table]

Note: [Any caveats — e.g., "record shows 2 IS studies; 3 additional are unconfirmed"]
```

## Output Format — Mode C (Semester Compliance Report)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MAXWELL TRACKER — SEMESTER COMPLIANCE REPORT
As of: [Semester]  |  Generated: [Date]  |  Policy: MMCE-2025
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 1 — OPEN MAKE-UP OBLIGATIONS
Total open:  [N]

  Expiring next semester (urgent):
  [Faculty / Dept / Cancelled Course / Clock Expiry / Option Selected]

  All open obligations:
  [Table: Faculty | Dept | Cancelled Course | Semester Cancelled | Option | Expiry | Status]

SECTION 2 — SELECTED TOPICS COURSE REGISTER
  At 2-offering limit (must regularize before re-offering):
  [Course | Dept | 1st Offering | 2nd Offering | Regularization Status]

  At 1st offering (1 remaining):
  [Course | Dept | 1st Offering Semester | Enrollment]

SECTION 3 — REQUIRED COURSE UNDER-ENROLLMENT LOG
  🚫 Remediation required (2 consecutive under-enrollments):
  [Course | Dept | Program | Consecutive Count | Remediation Status]

  ⚠️ Watch list (1 consecutive under-enrollment):
  [Course | Dept | Program | Under-Enrollment Semester | Enrollment]

SECTION 4 — INDEPENDENT STUDY BANK
  ✅ Offset-eligible (≥ 10 studies in 8-year window):
  [Faculty | Dept | IS Count | Window Start | Window End]

  ⚠️ Approaching offset (8–9 studies):
  [Faculty | Dept | IS Count | Window Start | Window End]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DATA QUALITY NOTES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Unconfirmed records, missing data, items awaiting chair confirmation]
```

## Quality Bar

Before finalizing any output, verify:
- [ ] No obligation clock math errors — expiry semester = clock_start + 6 fall/spring semesters
- [ ] Selected topics offering count never exceeds 2 without regularization approval on record
- [ ] Required course consecutive counter resets correctly when enrollment recovers
- [ ] IS bank window is calculated from the correct start date, not from the current semester
- [ ] No rows silently skipped — every input event is logged or explicitly excluded with a reason
- [ ] Alert section is present even if empty ("No alerts this cycle")
- [ ] Data Quality Notes present even if empty

## Downstream Handoffs

- Offset-eligible IS bank results → pass to `maxwell-enrollment-exceptions` when evaluating
  IS-offset flexibility claims
- Open obligations → surface in `maxwell-enrollment-monitor` Step 6 when checking consecutive
  under-enrollments for required courses
- Remediation-required required courses → dean's office must receive department submission
  before next semester schedule is finalized
