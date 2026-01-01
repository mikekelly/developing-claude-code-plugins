<overview>
Reference for creating Agent Skills in Claude Code plugins. Skills are model-invoked capabilities that Claude automatically applies when relevant to the user's request.
</overview>

<skill_basics>
## Skill File Structure

Location: `skills/skill-name/SKILL.md` at plugin root

```yaml
---
name: skill-name
description: What this skill does. Use when [trigger conditions].
---

<objective>
What this skill accomplishes.
</objective>

<quick_start>
Immediate guidance.
</quick_start>

<success_criteria>
How to know it worked.
</success_criteria>
```
</skill_basics>

<frontmatter_fields>
## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase letters, numbers, hyphens (max 64 chars) |
| `description` | Yes | What it does AND when to use (max 1024 chars) |
| `allowed-tools` | No | Restrict to specific tools |
| `model` | No | Model to use when skill is active |

### Name Requirements

- Lowercase letters, numbers, hyphens only
- Maximum 64 characters
- Must match directory name
- No reserved words: "anthropic", "claude"

### Description Best Practices

The description is how Claude decides to use your skill. Include:
1. What the skill does (capabilities)
2. When to use it (trigger conditions)
3. Keywords users would mention

Good:
```yaml
description: "Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction."
```

Bad:
```yaml
description: "Helps with documents"
```
</frontmatter_fields>

<allowed_tools>
## Tool Restrictions

Limit which tools Claude can use when skill is active:

```yaml
---
name: safe-reader
description: Read files without making changes.
allowed-tools: Read, Grep, Glob
---
```

When active, Claude can only use listed tools without permission prompts.

If omitted, skill doesn't restrict tools.
</allowed_tools>

<skill_structure>
## SKILL.md Body Structure

Use pure XML tags (no markdown headings):

```xml
<objective>
What the skill accomplishes. 1-3 paragraphs.
</objective>

<quick_start>
Immediate actionable guidance. Minimal working example.
</quick_start>

<process>
Step-by-step workflow.
</process>

<success_criteria>
How to verify the skill worked.
</success_criteria>
```

### Optional Tags

- `<context>` - Background information
- `<advanced_features>` - Deep-dive topics
- `<validation>` - Output verification
- `<examples>` - Usage examples
- `<anti_patterns>` - What NOT to do
</skill_structure>

<progressive_disclosure>
## Progressive Disclosure

Keep SKILL.md under 500 lines. Put details in reference files:

```
skills/pdf-processing/
├── SKILL.md           # Overview and navigation
├── FORMS.md           # Form filling details
├── REFERENCE.md       # API reference
└── scripts/
    └── validate.py    # Utility script
```

In SKILL.md:
```markdown
For form filling, see [FORMS.md](FORMS.md).
For API reference, see [REFERENCE.md](REFERENCE.md).
```

Claude loads reference files only when needed.
</progressive_disclosure>

<bundled_scripts>
## Bundled Scripts

Scripts can be executed without loading into context:

```markdown
Run the validation script:
```bash
python scripts/validate.py input.pdf
```

Only script output consumes tokens, not the script content.
</bundled_scripts>

<examples>
## Complete Examples

### Simple Skill

```yaml
---
name: commit-helper
description: "Generate commit messages from git diffs. Use when writing commit messages or reviewing staged changes."
---

<objective>
Generate clear, conventional commit messages by analyzing staged changes.
</objective>

<quick_start>
1. Run `git diff --staged` to see changes
2. Analyze the nature of changes
3. Generate commit message with:
   - Summary under 50 characters
   - Detailed description if needed
   - Affected components
</quick_start>

<success_criteria>
- Message follows conventional commits format
- Summary is concise and descriptive
- Explanation covers what and why
</success_criteria>
```

### Skill with Tool Restrictions

```yaml
---
name: code-analyzer
description: "Analyze code quality without modifications. Use for code review, security audit, or architecture analysis."
allowed-tools: Read, Grep, Glob
---

<objective>
Provide read-only code analysis focusing on quality, security, and architecture.
</objective>

<quick_start>
Use Read to examine files, Grep to search patterns, Glob to find files.
Never modify files - analysis only.
</quick_start>

<success_criteria>
- Identified issues categorized by severity
- No files modified
- Actionable recommendations provided
</success_criteria>
```

### Complex Skill with References

```yaml
---
name: pdf-processing
description: "Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction."
allowed-tools: Read, Bash(python:*)
---

<objective>
Process PDF files with extraction, form filling, and merging capabilities.
</objective>

<quick_start>
Extract text:
```python
import pdfplumber
with pdfplumber.open("doc.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For form filling, see [FORMS.md](FORMS.md).
</quick_start>

<success_criteria>
- PDF processed without errors
- Content extracted accurately
- Forms filled correctly
</success_criteria>
```
</examples>

<skills_vs_commands>
## Skills vs Slash Commands

| Aspect | Skills | Commands |
|--------|--------|----------|
| Discovery | Automatic (context-based) | Explicit (`/command`) |
| Structure | Directory with SKILL.md | Single .md file |
| Complexity | Complex capabilities | Simple prompts |
| Files | Multiple files, scripts | One file only |

Use Skills when:
- Claude should discover capability automatically
- Multiple files or scripts needed
- Complex workflows

Use Commands when:
- Explicit invocation preferred
- Simple prompt template
- Single file sufficient
</skills_vs_commands>
