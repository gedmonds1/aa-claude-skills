---
name: aa-syllabus
description: Create Syracuse University course syllabi following official Office of Academic Affairs requirements. Use when asked to create, generate, format, or review a Syracuse University syllabus, course template, or course outline. Handles mandatory institutional policy statements (academic integrity, AI policy, disability accommodations, discrimination/harassment, faith tradition observances) and optional sections (Turnitin, attendance, Blackboard LMS). Also use when faculty ask about AI policy options for syllabi, syllabus compliance, or required course policy language.
---

# Syracuse University Syllabus Generator

Create syllabi compliant with Syracuse University Office of Academic Affairs requirements. **Output as .docx using the docx skill.**

**Authoritative Source:** https://academicaffairs.syracuse.edu/important-syllabus-reminders/

All reference files in this skill are derived from the official Academic Affairs website and are kept synchronized with university policy.

## Workflow

### 1. Gather Course Information
Collect: course number/title/credits, meeting times/location, instructor name/contact/office hours, prerequisites.

### 2. AI Policy Selection (Required)
**IMPORTANT:** Always prompt the user to select their AI policy option. Do not assume or choose for them.

Present the three options and ask which one they prefer:

| Option | Use When | Key Feature |
|--------|----------|-------------|
| **Option 1: Zero Tolerance** | AI undermines learning objectives (writing courses, exams) | All AI prohibited |
| **Option 2: Some Use** | Some assignments benefit from AI | Instructor specifies which assignments |
| **Option 3: Open Use** | AI literacy is a learning goal | Permitted with disclosure/citation |

Faculty must choose exactly one option—custom AI statements are not permitted.

See `ai-policy-options.md` for exact required language.

### 3. Optional Sections
**Ask the user** which optional sections to include:
- Turnitin plagiarism detection
- Attendance policy (faculty customizes)
- Blackboard LMS information

See `optional-statements.md` for language.

### 4. Generate Syllabus
Assemble in this order:
1. Course information header
2. Course description and learning outcomes
3. Required materials
4. Grading and assignments
5. Course schedule
6. **Required institutional statements** (verbatim from `required-statements.md`):
   - Academic Integrity Statement
   - AI Policy (selected option)
   - Disability Statement
   - Discrimination/Harassment Statement
   - Faith Tradition Observances
7. Selected optional statements

## Critical Rules

1. **Required statements must be verbatim**—do not modify institutional language
2. **AI policy is mandatory**—cases cannot be investigated without a syllabus AI statement
3. **Faculty cannot write custom AI statements**—must use one of three official options
4. **Note**: The sentence "Any established violation in this course may result in course failure regardless of violation level" is already included in the required Academic Integrity statement

## Output

Use the **docx skill** to generate the final syllabus as a .docx file. Apply professional formatting with clear section headers.

## Reference Files

| File | Contents |
|------|----------|
| `required-statements.md` | Mandatory institutional statements (verbatim) |
| `ai-policy-options.md` | Three AI policy options with exact language |
| `optional-statements.md` | Turnitin, Attendance, Blackboard statements |

## Resources

- Academic Affairs Syllabus Reminders: https://academicaffairs.syracuse.edu/important-syllabus-reminders/
- Academic Integrity Office: aio@syr.edu
- Center for Disability Resources: https://disabilityresources.syr.edu
