---
name: assignment-conflict-detector
description: >
  Detect and resolve dates where students face too many concurrent major graded events across courses in a department or program, then recommend minimal-disruption adjustments to distribute workload more evenly across the term. Use this skill whenever an instructor, department chair, or curriculum coordinator wants to analyze syllabus scheduling conflicts, check for assignment clustering, identify midterm or deadline pileups, review workload distribution across a course sequence, or reduce DFW risk from concurrent high-stakes deadlines. Trigger this skill for any request involving: comparing deadlines across multiple syllabi, scheduling fairness for students, exam conflict analysis, assignment calendar coordination, or workload balancing across a semester. Even if the user simply says "look at these syllabi" or "do our deadlines cluster?" — use this skill.
license: Apache-2.0
metadata:
  author: Jerry Edmonds, Syracuse University
  domain: curriculum coordination, student success, academic scheduling
---

# Assignment & Exam Conflict Detector

Detect and resolve dates where students face too many concurrent major graded events, then recommend minimal-disruption adjustments to distribute workload more evenly across the term.

## Why This Matters

Students following a standard course sequence within a department or program often share the same set of required courses. When multiple instructors independently set major deadlines, clustering emerges—not from any single syllabus being unreasonable, but from the aggregate effect across courses. Research consistently shows that assignment clustering increases student stress, reduces learning quality, and contributes to DFW rates, particularly for first-generation and underrepresented students who may have fewer support structures to absorb the impact.

## What Counts as a "Major Graded Event"

Include any deliverable worth **10% or more** of the course grade:
- Exams (midterms, finals, unit exams)
- Major papers and essays
- Major projects and presentations
- Lab practicals
- Portfolio submissions

Exclude routine homework, reading responses, discussion posts, low-stakes quizzes, and participation-based grades.

## Conflict Threshold

| Events on Same Day | Classification | Action |
|--------------------|----------------|--------|
| 1–2 | Normal | No action needed |
| 3 | Watch date | Monitor; flag for awareness |
| 4+ | **Conflict date** | Requires resolution recommendation |

These thresholds assume students following the standard course progression for the program. If the user provides a specific co-enrollment pattern, adjust accordingly.

---

## Analysis Workflow

Work through the analysis in **three sequential passes**. Complete each pass fully before moving to the next.

### Pass 1 — Inventory & Map

For each course provided:
- Extract every major graded event with its scheduled date, assignment type, and weight (% of grade if available)
- Build a **consolidated calendar** listing all events by date across all courses, sorted chronologically
- If weight information is missing, note the gap but still include events that clearly meet the threshold (e.g., anything labeled "midterm exam" or "final project" qualifies regardless)

### Pass 2 — Conflict Detection

- Scan the consolidated calendar and identify every date with **4 or more** overlapping major graded events (conflict dates)
- For each conflict date: list the specific courses, assignment types, and weights involved
- Assign a **severity note**: the composition matters. Four paper deadlines are stressful but manageable differently than two exams plus two major projects. Note the mix.
- Flag dates with exactly **3 events** as watch dates—not violations, but worth monitoring, especially if the composition is heavy (e.g., 3 exams)

### Pass 3 — Resolution Recommendations

For each conflict date:
- Propose specific date changes for one or more assignments to bring the count to 3 or fewer
- Apply the **minimal disruption principle**: prefer shifting by 1–3 calendar days rather than full weeks
- For each proposed change, document:
  - Which course and assignment would move
  - The proposed new date
  - Whether the move creates or worsens any other conflict
  - Any sequencing risk (e.g., moving an exam before the content has been taught)

**Priority for moving assignments** (most to least flexible):
1. Paper and project deadlines
2. Presentations
3. Exams tied to content modules
4. Final exams (usually fixed by registrar — note as unresolvable if immovable)

After all recommendations, verify the final calendar has **zero dates with 4+ major graded events**. If a conflict proves unresolvable, note it explicitly with explanation.

---

## Output Format

Return the analysis in three clearly labeled sections:

### Section 1: Consolidated Assignment Calendar

```
| Date | Course | Assignment Type | Weight (% of grade) |
```
Sorted chronologically. Use "—" for unknown weights.

### Section 2: Conflict Report

For each **conflict date** (4+ events):
- Date
- Number of overlapping events
- List of courses and assignments involved
- Severity note (composition description)

Include a **Watch List** subsection for dates with exactly 3 events.

### Section 3: Recommended Adjustments

For each proposed change:
- Original date → Proposed date
- Course and assignment affected
- Rationale for the move
- Impact check: Does this create new conflicts? Any sequencing risk?

End with: **"X conflict dates resolved, Y remain unresolvable, Z watch dates in final calendar."**

---

## Common Patterns to Watch For

Flag these recurring clustering patterns explicitly when they appear:

- **Midterm week pileup**: Multiple courses independently targeting weeks 7–8 for midterms
- **Pre-break cramming**: Assignment clustering in the days before fall/spring break
- **End-of-semester convergence**: Final papers, projects, and presentations all landing in the last week of classes before finals
- **Monday/Wednesday bias**: Heavier assignment clustering on MWF course meeting days

When these patterns appear, note them — they often indicate systemic scheduling habits that a department can address through coordination norms rather than one-off fixes.

---

## Adaptation Guidance

**Single department analysis (default):** Analyze courses within one department assuming shared enrollment in the standard major sequence.

**Cross-departmental analysis:** When the user provides courses spanning multiple departments, ask: "Which courses do students in [program] typically co-enroll in during this semester?" Focus conflict detection on the co-enrolled subset.

**Mid-semester adjustment:** If syllabi have already shifted, the user can provide updated dates. Run the same three-pass analysis on the revised calendar. Flag any new conflicts that emerged from earlier changes.

**Partial information:** If the user provides only some syllabi, run the analysis on what's available but clearly note which courses are missing and that additional conflicts may exist.

---

## Quality Verification

Before presenting results, verify:
- Every conflict date (4+ events) has at least one proposed resolution
- No proposed change creates a new 4+ event conflict
- Recommendations respect content sequencing—flag any uncertainty rather than silently creating a pedagogical problem
- The final calendar shows zero dates with 4+ major graded events (or explicitly notes unresolvable cases)
- Watch dates are tracked so the user has early warning if conditions shift
