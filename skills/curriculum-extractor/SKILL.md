---
name: curriculum-extractor
description: Extract and analyze course information from Syracuse University course catalogs to build comprehensive tables for degree program analysis, curriculum mapping, and institutional research. Use when asked to extract curriculum data, map prerequisites, analyze degree requirements, or build course tables from SU catalog pages. Supports both undergraduate and graduate programs.
license: Apache-2.0
---

# Curriculum Extractor

Extract structured course data from Syracuse University course catalogs for curriculum analysis and institutional research.

## Workflow

1. **Get catalog URL** - If not provided, ask for program name and level (undergrad/grad)
   - Undergraduate: `https://coursecatalog.syracuse.edu/undergraduate/`
   - Graduate: `https://coursecatalog.syracuse.edu/graduate/`

2. **Fetch and extract** - Use `web_fetch`, then extract:
   - Program metadata (name, degree type, total credits, special requirements)
   - All required courses with: code, title, credits, prerequisites, corequisites, requirement type, restrictions
   - Elective categories and options

3. **Map prerequisites** - Document dependency chains and alternative paths
   - Notation: `ARC 407 ← ARC 308 ← ARC 207` for sequential
   - Use `OR`/`AND` for logical relationships: `(ARC 207 AND ARC 208) OR permission`

4. **Verify and flag issues** - Check for:
   - Missing course definitions referenced in prerequisites
   - Circular dependencies
   - Ambiguous requirements → tag with `[UNCERTAIN]`

5. **Output** - Generate both markdown tables and Excel workbook.

## Critical Rules

**Extract exactly as written** - Never invent, assume, or simplify. Preserve:
- Variable credits as ranges (e.g., "1-6" not just "6")
- Cross-listed courses with both prefixes (e.g., "ARC 308 / CRS 308")
- All footnotes, restrictions, and special conditions
- Complete elective groupings (e.g., "Select 6 credits from: ARC 401, 402, 403")

**Flag uncertainties** - When information is ambiguous or incomplete:
- `[UNCERTAIN - prerequisites not listed]`
- `[WARNING - circular prerequisite detected]`
- List all referenced-but-undefined courses in Data Quality Notes

## Edge Cases

| Situation | Handling |
|-----------|----------|
| Course referenced but not defined | Add to "Missing Information" in quality notes |
| Prerequisite alternatives | Document as: `Course A OR Course B` |
| Conditional prerequisites | Full notation: `(A AND B) OR permission of instructor` |
| Cross-college requirements | Note home college/school in Notes column |
