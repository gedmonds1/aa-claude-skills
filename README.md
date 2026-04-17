# AA Claude Skills

Custom Claude skill library developed by the Office of Academic Affairs at Syracuse University. These skills extend Claude's capabilities for higher education administration workflows — curriculum review, syllabus compliance, instructional analysis, and more.

---

## Installation

Each skill is a folder containing a `SKILL.md` file. Claude reads this file at runtime to know how to behave for that task.

### Prerequisites

- Access to [Claude.ai](https://claude.ai) (Pro, Team, or Enterprise plan)
- Skills feature enabled in your Claude account

### Step-by-step

**1. Download the skill folder**

Clone the entire repository or download just the skill(s) you need:

```bash
# Clone the full repo
git clone https://github.com/gedmonds1/aa-claude-skills.git

# Or download a single skill folder via GitHub:
# Navigate to skills/<skill-name>/ → click SKILL.md → Download raw file
```

**2. Open Claude Settings**

Go to [claude.ai](https://claude.ai) → click your profile icon → **Settings** → **Skills**.

**3. Add a new skill**

Click **+ Add Skill** and upload the `SKILL.md` file from the skill folder you downloaded. Give the skill a name matching the folder name (e.g., `senate-syllabus-review`).

**4. Trigger the skill**

Once installed, Claude will automatically invoke the skill when your request matches the skill's description. You can also reference it explicitly:

> *"Use the senate-syllabus-review skill to evaluate this course proposal."*

---

## Available Skills

### Curriculum & Instruction

| Skill | Description |
|---|---|
| [`aa-syllabus`](skills/aa-syllabus/) | Generate SU course syllabi following Academic Affairs requirements, including AI policy selection and all mandatory institutional statements |
| [`senate-syllabus-review`](skills/senate-syllabus-review/) | Systematic compliance review of syllabi and course proposals against SCCI standards, Academic Affairs requirements, and federal regulations |
| [`scci-template`](skills/scci-template/) | Create pixel-perfect SCCI course proposal review documents matching official SU template specs (Verdana, #E7E6E6 headers, Segoe UI Symbol checkboxes) |
| [`scci-data-extractor`](skills/scci-data-extractor/) | Extract and map curriculum data from SCCI review documents into Senate Review Summary spreadsheets |
| [`curriculum-extractor`](skills/curriculum-extractor/) | Extract structured course data from SU course catalog pages for degree program analysis and curriculum mapping |
| [`curricular-complexity`](skills/curricular-complexity/) | Analyze prerequisite structures and program complexity scores to identify barriers to student progression and timely degree completion |
| [`assignment-conflict-detector`](skills/assignment-conflict-detector/) | Detect and resolve deadline clustering across courses in a department or program to reduce student workload pileups and DFW risk |

### Faculty & Teaching Analysis

| Skill | Description |
|---|---|
| [`departmental-teaching-summary`](skills/departmental-teaching-summary/) | Produce a dean-ready teaching summary table from course enrollment data, with subtotals by faculty title group and arithmetic verification |

### Syllabus Design

| Skill | Description |
|---|---|
| [`su-syllabus-design`](skills/su-syllabus-design/) | Evidence-based guidance for designing or improving syllabi using backward design, learning science, and UDL frameworks |

### Administrative & Communications

| Skill | Description |
|---|---|
| [`su-briefing-memo`](skills/su-briefing-memo/) | Generate chancellor-level briefing memos as .docx files that exactly match the official SU Briefing Memo Template |
| [`qualtrics-qsf`](skills/qualtrics-qsf/) | Generate valid Qualtrics Survey Files (QSF) for import into Syracuse University's Qualtrics tenant |

### Prompt Engineering

| Skill | Description |
|---|---|
| [`prompt-engineering`](skills/prompt-engineering/) | Create and improve Claude prompts using the Prompt Spine framework, with Claude 4.x-specific guidance on effort control and anti-patterns |

---

## Skill Structure

Each skill follows this layout:

```
skills/
└── skill-name/
    ├── SKILL.md          ← Main skill file (required)
    └── references/       ← Supporting reference files (optional)
```

The `SKILL.md` file contains:
- A YAML frontmatter block with `name` and `description`
- Step-by-step workflow instructions for Claude
- Output format specifications
- Critical rules and quality standards

---

## Notes

- Skills with `references/` subfolders reference local files that are not included in this repository (they contain institution-specific content). The `SKILL.md` files are self-contained for most use cases.
- Some skills are designed specifically for Syracuse University workflows and reference SU systems (Qualtrics tenant, SCCI process, SU catalog URLs). Adapt field names and URLs as needed for your institution.
- Skills are tested against Claude Sonnet 4.6 and Opus 4.6.

---

## Contributing

Issues and pull requests welcome. If you adapt a skill for another institution or identify an error, please open an issue.

---

## License

MIT — free to use and adapt with attribution. See [LICENSE](LICENSE) for details.

---

*Maintained by the Office of Academic Affairs, Syracuse University.*
