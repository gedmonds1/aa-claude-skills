---
name: scci-template
description: Create pixel-perfect Syracuse University Senate Committee on Curricula and Instruction (SCCI) course proposal review documents. Use when asked to create SCCI review templates, course proposal evaluations, syllabus compliance reviews, or any document that must match the exact SCCI template specifications (fonts, colors, spacing, table structure, checkboxes). This skill ensures precise adherence to typography (Verdana 11pt/16pt), color palette (#E7E6E6 headers, #CCCCCC borders), table structure (2-column with specific widths), checkbox formatting (☐/☒ in Segoe UI Symbol), and all spacing/margin requirements.
license: Apache-2.0
---

# SCCI Template Generator

Create pixel-perfect Syracuse University SCCI course proposal review documents matching official template specifications.

## Quick Start

When asked to create an SCCI template or course proposal review document:

1. Read `references/template_spec.md` for complete specifications
2. Use `scripts/create_template.py` to generate the base template
3. Apply validation checks from `references/validation_checklist.md`

## Core Workflow

### Step 1: Generate Base Template

```bash
cd /home/claude
python scripts/create_template.py
```

Creates `SCCI_Template_Generated.docx` with all specifications applied.

### Step 2: Verify Critical Specifications

**Typography:**
- Title: Verdana, 16pt, Bold, Left-aligned
- Headings: Verdana, 11pt, Bold (NOT 14pt default)
- Body: Verdana, 11pt, Regular

**Colors:**
- Table header cells: #E7E6E6 (light gray)
- Table borders: #CCCCCC (medium gray)
- All text: #000000 (black)

**Table Structure:**
- 2 columns: 1.625" + 4.75" = 6.375" total
- Left column: Bold, gray background
- Right column: Regular, white background

**Checkboxes:**
- Font: Segoe UI Symbol
- Empty: ☐ (U+2610)
- Checked: ☒ (U+2612)

## Document Structure (Must Follow This Order)

1. Title: "SCCI COURSE PROPOSAL REVIEW"
2. Course Information Table (4 rows × 2 columns)
3. Executive Summary
4. Compliance Status (with checkbox options)
5. Critical Compliance Issues (Must Fix)
6. Technical/Data Quality Issues (Must Fix)
7. Recommendations (Optional Improvements)
8. Approval Clearance Checklist (8 items)
9. Summary and Next Steps
10. Resources and Support
11. Contact Information

## Common Pitfalls

❌ **DON'T:** Center the title; use 14pt for Heading 1; use theme colors; use standard checkbox characters

✅ **DO:** Left-align title; keep all headings at 11pt bold; use exact RGB color codes; test checkbox display

## Quality Assurance

Before presenting any SCCI document:
1. Verify title is Verdana 16pt, bold, left-aligned
2. Check table has gray header cells (#E7E6E6)
3. Confirm all text is Verdana 11pt
4. Ensure checkboxes are ☐ in Segoe UI Symbol
5. Validate heading structure (H1 = 11pt, not 14pt)
