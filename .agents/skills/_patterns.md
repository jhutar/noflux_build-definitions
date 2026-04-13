# Agent Skills Common Patterns

This document contains reusable patterns referenced by skills to promote consistency and reduce duplication.

## 1. Tool Permission Pattern

All skills MUST declare tool permissions via `allowed-tools` in YAML frontmatter to implement the principle of least privilege.

### Available Tools
- **Read**: Read files from filesystem
- **Write**: Create new files
- **Edit**: Modify existing files
- **Glob**: Pattern-based file searching
- **Grep**: Content searching within files
- **Bash**: Execute bash commands (can be scoped, e.g., `Bash(git:*)`)

### Adding Permissions
Determine required tools and apply the minimum necessary set:
```yaml
---
name: skill-name
description: ...
allowed-tools: Read,Write,Grep
---
```

## 2. Progressive Disclosure Pattern

Organize complex skills into a modular structure: high-level workflow + detailed references + templates.

### Pattern Structure
```text
skill-name/
├── SKILL.md                    # High-level workflow (always loaded)
├── references/                 # Detailed docs (loaded on-demand)
│   ├── detailed-process.md     # Step-by-step procedures
└── assets/                     # Templates and static files
    └── template.md             # Ready-to-use templates
```

### SKILL.md (Base Workflow)
Keep concise (500-1000 words). Include YAML frontmatter, overview, high-level workflow steps, and clear references to detailed docs/templates.

## 3. Input Validation & Sanitization

### Filename Sanitization Pattern
Use when converting user input to filenames:
- Remove path traversal: `..`, `/`, `\`
- Remove dangerous characters: null bytes, control characters
- Convert to lowercase kebab-case (spaces → hyphens, remove special characters except hyphens and alphanumerics)
- Limit length: max 80-100 characters

### Version Number Validation Pattern
- Format: X.Y.Z (e.g., 1.2.3)
- Allow only: digits (0-9) and dots (.)
- Convert dots to hyphens for filenames: 1.2.3 → 1-2-3

### Specialist Role Validation Pattern
- Allow: letters, numbers, spaces, hyphens
- Convert to title case for display, kebab-case for filenames

## 4. Error Handling

### Framework Not Set Up
```markdown
The AI Software Architect framework is not set up yet.
To get started: "Setup ai-software-architect"
```

### File Not Found
```markdown
Could not find [file/directory name].
Expected location: [path]
```

### Permission Error
```markdown
Permission denied accessing [file/path].
```

## 5. File Loading

### Load Configuration
1. Check if `.architecture/config.yml` exists
2. If missing: Use default configuration
3. If exists: Parse YAML and extract settings
4. Handle errors gracefully

### Load Members
1. Check if `.architecture/members.yml` exists
2. Parse YAML and extract member information
3. Validate structure and return array of member objects

### Load ADR List
1. List files matching: `ADR-[0-9]+-*.md` in `.architecture/decisions/adrs/`
2. Extract ADR numbers and titles, sort by number

## 6. Reporting Format

### Success Report Pattern
```markdown
[Skill Action] Complete: [Target]

Location: [file path]
Key Points:
- [Point 1]
- [Point 2]

Next Steps:
- [Action 1]
```

### Review Report Pattern
```markdown
# [Review Type]: [Target]

**Reviewer**: [Name/Role]
**Assessment**: Excellent | Good | Adequate | Needs Improvement | Critical Issues

## Executive Summary
## [Detailed Analysis Sections]

## Recommendations
### Immediate (0-2 weeks)
### Short-term (2-8 weeks)
### Long-term (2-6 months)
```

## 7. Skill Workflow Template

```markdown
---
name: skill-name
description: Clear description with trigger phrases.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Skill Title

## Process
### 1. [First Step]
### 2. [Second Step]
### N. Report to User

## Related Skills
**Before This Skill**: "[Related skill]"
**After This Skill**: "[Related skill]"
```

## 8. Directory Structure Validation Pattern

When skills need specific directory structures:
1. Check `.architecture/` exists
2. Check/create required subdirectories (`decisions/adrs/`, `reviews/`, `templates/`, etc.)
3. Verify key files exist (`members.yml`, `principles.md`, `config.yml`)
