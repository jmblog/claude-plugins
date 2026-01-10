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

### marketplace.json Structure

The marketplace.json uses a **parent plugin with skills array** pattern:

```json
{
  "plugins": [
    {
      "name": "jmblog-plugins",
      "description": "Description of the plugin...",
      "source": "./",
      "strict": false,
      "skills": [
        "./skills/improving-ux",
        "./skills/reviewing-typescript",
        ...
      ]
    }
  ]
}
```

**Important**: Each skill is listed in the `skills` array, NOT as a separate plugin entry.

### Current Skills

- **improving-ux**: UX analysis using evidence-based psychology principles
- **reviewing-typescript**: TypeScript code review and best practices
- **designing-apis**: REST/GraphQL API design and review
- **planning-tests**: Testing strategy planning and implementation
- **reviewing-architecture**: Software architecture review
- **designing-databases**: Database schema and query design
- **designing-with-shig**: UI design using Sociomedia's Human Interface Guidelines

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
/plugin marketplace add /Users/jimbo/Projects/jmblog/claude-plugins

# Install a skill
/plugin install ux-growth@claude-plugins
```

## Development Workflow

### Adding New Skills

1. Create directory: `skills/skill-name/`
2. Create `skills/skill-name/SKILL.md` with YAML frontmatter
3. Add supporting files in `skills/skill-name/references/` if needed
4. Update `.claude-plugin/marketplace.json`:
   - Add path to the `skills` array: `"./skills/skill-name"`
5. **Always validate**: `claude plugin validate .`
6. Commit and push to GitHub

### Modifying Existing Skills

1. Edit files in `skills/skill-name/`
2. **Always validate**: `claude plugin validate .`
3. Commit and push to GitHub
4. Users update via: `/plugin update skill-name@claude-plugins` and restart Claude Code

### Critical Rules

**marketplace.json structure**:
- Use a single parent plugin with a `skills` array (NOT separate plugin entries per skill)
- Plugin `source` MUST be `"./"` (root directory)
- Skill paths in the array MUST start with `./` (e.g., `"./skills/improving-ux"`)
- Always run `claude plugin validate .` before committing

**Skill YAML frontmatter**:
- `name` must match the directory name and marketplace.json entry
- `description` should explain when the skill triggers and what it does
- Description format: "Use when users request: (1) X, (2) Y, (3) Z..."
- Do NOT include `user-invocable: true` field (it causes skills to not appear in slash commands)

**Documentation structure**:
- Keep SKILL.md focused on workflows and quick reference
- Move comprehensive details to `references/` files
- Reference supporting docs explicitly in SKILL.md
