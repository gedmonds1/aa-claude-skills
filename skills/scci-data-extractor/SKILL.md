---
name: scci-data-extractor
description: Extract and map curriculum data from Syracuse University SCCI (Senate Committee on Curricula and Instruction) course and program review documents into standardized Senate Review Summary spreadsheets. Use when asked to process SCCI review documents, populate curriculum tracking spreadsheets, extract proposal information from Senate review reports, or map curriculum data from Word documents to Excel tracking systems.
---

# SCCI Data Extractor

Extract curriculum proposal data from SCCI review documents and map to standardized Senate Review Summary spreadsheets for institutional tracking.

## About SCCI Reviews

Syracuse University's Senate Committee on Curricula and Instruction (SCCI) generates detailed review reports for all course and program proposals submitted through the curriculum approval process.

## Extraction Workflow

### Step 1: Document Analysis

Read each SCCI review document and identify:
1. **Proposal type**: Course vs. program, new vs. revised vs. inactive
2. **Core identifiers**: Course prefix/number/title or program name
3. **Department and school/college**
4. **Effective term** (if stated)
5. **Compliance determination**: Approved, approved with notes, revisions required, rejected

### Step 2: Field Mapping

| Spreadsheet Column | Source | Rules |
|-------------------|--------|-------|
| **Course Prefix** | Review title/body | Alpha prefix only (e.g., "MMI") |
| **Course #** | Review title/body | Numeric portion only |
| **Program Name** | Review title/body | Full program name as stated |
| **Submission Type** | Review content | "Course - New", "Course - Revised", "Course - Inactive", "Program - New", "Program - Revised", "Program - Inactive" |
| **Department** | Review body | Extract from proposal details |
| **School/College** | Review body | Extract or infer from department |
| **Effec. Term** | Review body | Format: "Fall 2025" (only if stated) |
| **Course Title** | Review title/body | Full course title |
| **Pre-Reqs/Co-Reqs** | Review body | Prerequisite/corequisite information |
| **Notes** | Review body | "APPROVED", "APPROVED WITH NOTES: [summary]", "REVISIONS REQUIRED: [issues]", "REJECTED: [reasons]" |

### Step 3: Handle Missing Information

- **Leave blank** (do not enter "N/A" or "Unknown")
- **Do NOT invent or infer** information not present in documents

### Step 4: Quality Verification

1. Verify course prefix/number exactly match review document
2. Confirm submission type classification is accurate
3. Check Notes field accurately reflects compliance determination
4. Ensure no fields contain placeholder text or assumptions

## Critical Rules

**DO:** Extract exactly as stated; use standardized terminology; leave fields blank when unavailable

**DO NOT:** Invent course numbers or titles; assume unstated information; merge multiple proposals into one row; modify spreadsheet structure

## Output Format

One row per proposal. After table, provide:
- **Extraction Notes**: Document filename, confidence level (HIGH/MEDIUM/LOW), significant ambiguities
- **Data Quality Assessment**: Total proposals processed, complete vs. partial records, most frequently missing fields
