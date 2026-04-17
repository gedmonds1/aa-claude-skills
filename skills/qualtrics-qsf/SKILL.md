---
name: qualtrics-qsf
description: Generate Qualtrics Survey Files (QSF) in valid JSON format compatible with Syracuse University's Qualtrics tenant. Use when users request creating Qualtrics surveys, generating QSF files, or building survey instruments that need to be imported into Qualtrics. Supports Likert scale multiple choice questions and text entry questions.
---

# Qualtrics QSF Generator (Syracuse University)

**Institution:** Syracuse University  
**Qualtrics Brand:** syracuseuniversity  
**Supported Question Types:** Likert MC (SAHR), Text Entry (ESTB)

Generate valid Qualtrics Survey Files (QSF) that can be imported directly into Syracuse University's Qualtrics tenant.

## Output Requirements

**CRITICAL:** Output ONLY valid JSON. No markdown code blocks, no explanations, no commentary. The entire response must be a single JSON object that can be saved directly as a .qsf file.

## QSF Structure

Every QSF must have exactly two top-level keys:

```json
{
  "SurveyEntry": { ... },
  "SurveyElements": [ ... ]
}
```

## SurveyEntry Template

Modify ONLY `SurveyName` and `SurveyDescription`. Never modify ID fields (SurveyID, OwnerID, BrandID, DivisionID, ResponseSet).

## SurveyElements Array

Elements must appear in this exact order:
1. BL (Blocks)
2. FL (Flow)
3. PL (Preview Link)
4. PROJ (Project)
5. QC (Question Count) — set `SecondaryAttribute` to total question count as string
6. RS (Response Set)
7. SCO (Scoring)
8. SO (Survey Options) — keep `SkinLibrary` as `"syracuseuniversity"`
9. STAT (Statistics)
10. SQ (Survey Questions — one element per question)

## Question Types

### Likert Scale Multiple Choice (MC-SAHR)
- `QuestionType`: `"MC"`, `Selector`: `"SAHR"`, `SubSelector`: `"TX"`
- Always use 5-point scale: Strongly disagree / Disagree / Neither agree nor disagree / Agree / Strongly agree
- `ChoiceOrder`: `[1, 2, 3, 4, 5]`

### Text Entry (TE-ESTB)
- `QuestionType`: `"TE"`, `Selector`: `"ESTB"`
- Include `SearchSource` with `AllowFreeResponse: "false"`

## Generation Process

1. Determine total question count
2. Build SurveyEntry (name and description only)
3. Build SurveyElements in required order
4. Update QC `SecondaryAttribute` to match question count
5. List all QuestionIDs in BL element's BlockElements array
6. Add SQ elements with sequential QIDs (QID1, QID2, etc.)
7. Validate JSON structure
8. Output complete JSON — no formatting, no explanations

## Critical Rules

1. Never modify fixed IDs
2. Sequential QuestionIDs: QID1, QID2, QID3, etc.
3. No trailing commas — ensure valid JSON
4. Include all required elements in correct order
5. Raw JSON only — no markdown, no explanations
