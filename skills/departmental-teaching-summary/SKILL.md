---
name: departmental-teaching-summary
description: Analyze a spreadsheet of course enrollment data for an entire department or school and produce a structured teaching summary table broken down by faculty name, title, rank, course prefix, course name, enrollment, and credit hours — with subtotals by faculty title group and a data quality notes section. Use when asked to analyze departmental teaching data, summarize course loads across a unit, produce faculty roster teaching tables, or review staffing patterns for portfolio review or academic planning. Triggers on "departmental teaching," "school teaching summary," "faculty roster analysis," "course load by title," or "who is teaching what in [dept/school]." Use this skill — not faculty-teaching-analysis — whenever the analysis covers multiple faculty and the output is a single consolidated table, not individual reports.
license: Apache-2.0
metadata:
  version: "1.0"
  companion-skill: faculty-teaching-analysis
  author: Syracuse University Academic Affairs
---

# Departmental Teaching Summary

Produce a clean, decision-ready departmental teaching summary table from a raw course enrollment spreadsheet covering multiple faculty members. This skill is designed for academic units (departments, schools, colleges) and produces a single consolidated output — not individual faculty reports.

> **Companion skill**: For in-depth single-faculty analysis (trajectory, pattern interpretation, Key Findings narrative), use the `faculty-teaching-analysis` skill instead.

## Purpose

Academic Affairs and school leadership need to see, at a glance:
- Who is teaching what
- How load distributes across faculty title categories (tenure-stream vs. teaching faculty vs. PTI)
- Total credit hours and student contact by group

## Execution Workflow

Follow these five phases **in exact order**.

### Phase 1: Data Validation

Confirm the spreadsheet contains all required fields:

| Required Field | Acceptable Column Names |
|---|---|
| Faculty name | Instructor, Faculty, Name, Instructor Name |
| Title/Rank | Title, Rank, Faculty Title, Instructor Title, Status |
| Course prefix | Subject, Prefix, Course Subject, Dept |
| Course name | Course Title, Title, Name |
| Enrollment | Students, Enrolled, Headcount, Enrollment |
| Credit hours | Credits, Credit Hours, SCH, Units |

If any required field is missing: STOP. Report which fields are absent and request clarification.

### Phase 2: Title and Rank Standardization

| Standardized Title | Maps From |
|---|---|
| Professor | Full Professor, Professor, Prof |
| Associate Professor | Assoc Prof, Associate Professor |
| Assistant Professor | Asst Prof, Assistant Professor |
| Teaching Professor | Teaching Professor, Prof of Practice |
| Senior Lecturer | Senior Lecturer, Sr. Lecturer |
| Lecturer | Lecturer |
| Part-Time Instructor | PTI, Adjunct, Part-Time, Part Time Instructor |
| Other / Unknown | Anything not matching above — flag in Data Quality Notes |

### Phase 3: Aggregation Rules

**One row per section per faculty member.** Sort by: Title group → Faculty Name → Course Prefix → Course Name.

Title group sort order: Professor → Associate Professor → Assistant Professor → Teaching Professor → Senior Lecturer → Lecturer → Part-Time Instructor → Other/Unknown

### Phase 4: Output Construction

**Main table columns:**
```
| Faculty Name | Title | Rank | Course Prefix | Course Name | Number of Students | Credit Hours |
```

**Subtotals by Title Group:**
```
| Faculty Title | Sections | Total Students | Total Credit Hours |
```
Include grand total row. Verify subtotals sum to grand total.

**Data Quality Notes** (required, even if clean):
- Source, records processed, records flagged
- Issues identified and how handled
- Fields standardized

### Phase 5: Arithmetic Verification

Before delivering, verify:
- Subtotal rows sum correctly
- Grand total = sum of all subtotals
- No rows missing from the main table
- Zero-enrollment records are present and flagged, not dropped

## Hard Constraints

- **No estimation**: Never fill in missing values
- **No fabrication**: Never create rows not in source data
- **No quality judgments**: Table documents facts, not performance
- **No PII**: No student names or IDs
- **No removal**: Flag anomalous records; never silently drop them
