# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code skills marketplace repository that provides custom skills for enhanced productivity. The repository is structured to be distributed via Claude Code's plugin system, allowing users to install skills that extend Claude's capabilities with specialized knowledge and workflows.

## Architecture

### Marketplace Configuration

The repository follows Claude Code's marketplace specification:
- `.claude-plugin/marketplace.json` - Defines the marketplace metadata and available plugins/skills
- Each skill is a separate subdirectory under `skills/`
- Skills contain a `SKILL.md` with YAML frontmatter and optional `references/` directory

### Skill Structure

Skills consist of:
1. **SKILL.md**: Main skill definition with YAML frontmatter containing:
   - `name`: Skill identifier
   - `description`: When and how the skill should be triggered
   - Markdown content with skill instructions, workflows, and guidelines

2. **references/** (optional): Supporting documentation files that provide:
   - Comprehensive reference materials
   - Checklists and frameworks
   - Examples and patterns
   - These are referenced from the main SKILL.md

### Current Skills

**ux-growth**: UX analysis and optimization using evidence-based psychology principles
- Provides 25+ UX psychology principles
- Systematic analysis framework with 10 evaluation categories
- Implementation patterns and anti-patterns
- Located in: `skills/ux-growth/`

## Common Commands

### Validation

Always validate before committing changes:
```bash
claude plugin validate .
```

This validates:
- JSON syntax in marketplace.json
- Required fields (name, owner, plugins)
- Plugin source paths (must start with `./`)
- Schema compliance

### Testing Installation Locally

```bash
# Add the marketplace
/plugin marketplace add /Users/jimbo/Projects/jmblog/claude-skills

# Install a skill
/plugin install ux-growth@claude-skills
```

## Development Workflow

### Adding New Skills

1. Create directory: `skills/skill-name/`
2. Create `skills/skill-name/SKILL.md` with YAML frontmatter
3. Add supporting files in `skills/skill-name/references/` if needed
4. Update `.claude-plugin/marketplace.json`:
   - Add entry to `plugins` array with `name`, `source` (must use `./` prefix), and `description`
5. **Always validate**: `claude plugin validate .`
6. Commit and push to GitHub

### Modifying Existing Skills

1. Edit files in `skills/skill-name/`
2. **Always validate**: `claude plugin validate .`
3. Commit and push to GitHub
4. Users update via: `/plugin update skill-name@claude-skills` and restart Claude Code

### Critical Rules

**marketplace.json validation**:
- `source` paths MUST start with `./` (e.g., `./skills/ux-growth` not `skills/ux-growth`)
- Always run `claude plugin validate .` before committing
- The plugins array contains plugin definitions, not skill definitions

**Skill YAML frontmatter**:
- `name` must match the directory name and marketplace.json entry
- `description` should explain when the skill triggers and what it does
- Description format: "Use when users request: (1) X, (2) Y, (3) Z..."

**Documentation structure**:
- Keep SKILL.md focused on workflows and quick reference
- Move comprehensive details to `references/` files
- Reference supporting docs explicitly in SKILL.md
