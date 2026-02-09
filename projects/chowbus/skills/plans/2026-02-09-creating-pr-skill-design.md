# Creating PR Skill - Design Document

**Date:** 2026-02-09
**Status:** Approved

## Summary

Enhance the `creating-pull-requests` skill with auto-detection, category confirmation, and layered team conventions.

## Goals

1. **Consistency** - Standardized template across all Chowbus repos
2. **Completeness** - Auto-detect sections needed based on diff
3. **Automation** - Diff analysis + category-level confirmation
4. **Team conventions** - Layered defaults (skill → CLAUDE.md → user)
5. **Portability** - Tool-agnostic commands for Claude Code, Cursor, etc.

## Skill Structure

```
~/.claude/skills/creating-pull-requests/
├── SKILL.md              # Main skill (tool-agnostic)
└── chowbus-defaults.md   # Chowbus-specific conventions
```

## Three-Phase Workflow

### Phase 1: Analyze Changes

Run in parallel:
- `git status` - uncommitted changes
- `git log main..HEAD --oneline` - commits to include
- `git diff main...HEAD` - full diff for analysis

**Auto-Detection Rules:**

| Category | Detection Pattern | Section Triggered |
|----------|-------------------|-------------------|
| Env vars | `${VAR_NAME}` or `env:` in yml/properties | Environment Variables table |
| API endpoints | `@GetMapping`, `@PostMapping`, `router.GET`, `r.Handle` | API Documentation |
| Migrations | Files in `*/migrations/*`, `db/migrate/*`, `flyway/*` | Database Changes section |
| Breaking changes | Removed public methods, changed signatures | Breaking Changes warning |
| Config files | `application*.yml`, `*.properties` | Config Changes note |

**Output Format:**
```
┌─ Detected Changes ─────────────────────────┐
│ ✓ Env vars (2): ATLASNOVA_KEY, AI_SOCIAL_* │
│ ✓ Endpoints (1): GET /api/v1/ai-social/... │
│ ✗ Migrations: none                         │
│ ✗ Breaking changes: none                   │
│ ✓ Config files (3): staging-01, staging-02 │
└────────────────────────────────────────────┘
```

### Phase 2: Confirm Sections

For each detected category, prompt user:

1. **Env vars:** "Found 2 env vars. Include Environment Variables section? (yes/skip/edit)"
2. **Endpoints:** "Found endpoint. Include API Documentation? (yes/skip/edit)"
3. **Config changes:** "Found 3 profiles. Include Defaults by Environment table? (yes/skip)"
4. **Breaking changes:** "⚠️ Potential breaking change. Add warning? (yes/skip)"
5. **Migrations:** "Found migration. Include Database Changes? (yes/skip)"

Skip categories with no detections.

### Phase 3: Generate PR

**Title Generation:**
- Extract ticket from branch name or commits
- Format: `[TICKET] <summary from commits>`

**Body Assembly:**
- Always: Summary, Jira Story, Rollback Plan
- Conditional: Sections confirmed in Phase 2

**Pre-populate detected values** with placeholders for user to complete.

## Conventions (Layered Defaults)

### Layer 1: Skill Defaults
- Bilingual format: English / 中文
- Jira URL: `https://chowbus.atlassian.net/browse/`
- Standard sections: Summary, Key Changes, Jira, Rollback
- Reference-style links at bottom

### Layer 2: Project CLAUDE.md
- Required sections
- Ticket prefix
- Required reviewers
- Extra sections

### Layer 3: User ~/.claude/CLAUDE.md
- Personal preferences
- Language preferences

### Fallback Behavior
- No conventions → Layer 1 defaults
- Partial CLAUDE.md → merge with defaults

## Platform Compatibility

Tool-agnostic design works with:
- Claude Code
- Cursor
- Windsurf
- Any AI assistant with git access

## Quick Reference

| Phase | Action | Command |
|-------|--------|---------|
| Analyze | Check uncommitted | `git status` |
| Analyze | View commits | `git log main..HEAD --oneline` |
| Analyze | View diff | `git diff main...HEAD` |
| Generate | Push branch | `git push -u origin <branch>` |
| Generate | Create PR | `gh pr create` or platform UI |

## Next Steps

1. Implement updated SKILL.md with 3-phase workflow
2. Create chowbus-defaults.md with team conventions
3. Test with real PR creation scenarios
