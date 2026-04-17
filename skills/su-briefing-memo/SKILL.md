---
name: su-briefing-memo
description: Generate chancellor-level briefing memos as formatted Word (.docx) files that exactly match the official Syracuse University Briefing Memo Template — including the Block S logo, Verdana font, SU navy MEMORANDUM heading, inline divider line, and correct bullet formatting. Use this skill whenever asked to produce a briefing memo, chancellor briefing, meeting brief, or any memo using the SU template. Also trigger when the user says "write a memo," "draft a briefing," "prepare a brief for the Chancellor," or provides To/From/Date/Subject content and asks for a formatted document. Even if the user just says "make a memo for the Chancellor's meeting with X" — use this skill. Do NOT attempt to build the memo from scratch using docx.js or any other code generator; always clone the real template file.
---

# SU Briefing Memo Skill

Produces chancellor-level briefing memos as `.docx` files that are **pixel-perfect matches** to the official SU Briefing Memo Template. The template contains a floating Block S WMF image, Verdana font, SU navy `#132654` MEMORANDUM heading, and an inline connector line — none of which can be reproduced by building from scratch. The only correct approach is to clone the template and fill it via XML editing.

## Bundled Assets

| File | Purpose |
|------|---------|
| `assets/Briefing_Memo_Template.docx` | The real template — clone this every time |
| `assets/fill_memo_reference.py` | Reference implementation of the fill script |

## Workflow

### Step 1: Collect memo content

Gather: To, From, Date, Subject, Cc (optional), Summary (3–4 bullets), Logistical Details (optional), Background (5–6 bullets), Recommendations (4+ bullets), Enclosures (optional).

### Step 2: Set up the working copy

```bash
cp <skill_dir>/assets/Briefing_Memo_Template.docx /home/claude/memo_working.docx
python /mnt/skills/public/docx/scripts/office/unpack.py \
    /home/claude/memo_working.docx /home/claude/memo_out/
```

### Step 3: Write the fill script

Create `/home/claude/fill_memo.py` using the pattern in `assets/fill_memo_reference.py`. Populate the `memo = { ... }` data object with user content.

### Step 4: Run the fill script

```bash
python3 /home/claude/fill_memo.py
```

### Step 5: Pack the docx

```bash
python /mnt/skills/public/docx/scripts/office/pack.py \
    /home/claude/memo_out/ /home/claude/Briefing_Memo.docx \
    --original /home/claude/memo_working.docx --validate false
```

### Step 6: Fix namespace and settings issues (always required)

**Fix 1 — Strip broken external template reference:**
```bash
python3 -c "
import re
f = '/home/claude/memo_out/word/settings.xml'
content = open(f).read()
content = re.sub(r'\s*<w:attachedTemplate[^/]*/>', '', content)
open(f, 'w').write(content)
f2 = '/home/claude/memo_out/word/_rels/settings.xml.rels'
content2 = open(f2).read()
content2 = re.sub(r'\s*<Relationship[^>]*attachedTemplate[^>]*/>', '', content2)
open(f2, 'w').write(content2)
"
```

**Fix 2 — Patch mc:Ignorable in the packed docx:**
```python
import zipfile, re, os
docx_path = '/home/claude/Briefing_Memo.docx'
tmp_path  = '/home/claude/tmp_fixed.docx'
with zipfile.ZipFile(docx_path, 'r') as zin, \
     zipfile.ZipFile(tmp_path, 'w', zipfile.ZIP_DEFLATED) as zout:
    for item in zin.infolist():
        data = zin.read(item.filename)
        if item.filename == 'word/document.xml':
            content = data.decode('utf-8')
            declared = set(re.findall(r'xmlns:(\w+)=', content))
            def fix_ignorable(m):
                kept = [p for p in m.group(1).split() if p in declared]
                return f'mc:Ignorable="{" ".join(kept)}"'
            content = re.sub(r'mc:Ignorable="([^"]+)"', fix_ignorable, content)
            data = content.encode('utf-8')
        zout.writestr(item, data)
os.replace(tmp_path, docx_path)
```

### Step 7: Validate

```bash
python /mnt/skills/public/docx/scripts/office/validate.py /home/claude/Briefing_Memo.docx
```

### Step 8: Deliver

```bash
cp /home/claude/Briefing_Memo.docx /mnt/user-data/outputs/Briefing_Memo.docx
```

## Template Elements (Do Not Recreate)

| Element | Details |
|---------|---------|
| Block S logo | Floating WMF image, anchored alongside MEMORANDUM |
| MEMORANDUM text | Bold, color #132654 (SU navy), Verdana |
| Divider line | Inline wps:wsp straight connector drawing shape |
| Font | Verdana body, Arial complex script |
| Page size | US Letter, 1" top/left/right, 0.5" bottom |

## Quick Checklist Before Delivering

- ✅ Validation passes with zero errors
- ✅ All five sections present in correct order
- ✅ All user content appears verbatim
- ✅ File named descriptively
