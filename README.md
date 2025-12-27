# Claude Skills

Personal collection of custom Claude Code skills for enhanced productivity and specialized workflows.

## Installation

### Add this marketplace to Claude Code

```bash
/plugin marketplace add https://github.com/jmblog/claude-skills.git
```

### Install skills

```bash
# Install all skills
/plugin update

# Or install specific skills
/plugin install reviewing-typescript@claude-skills
/plugin install designing-apis@claude-skills
/plugin install planning-tests@claude-skills
```

### Update skills

To get the latest version of skills from the marketplace:

```bash
# Update a specific skill
/plugin update improving-ux@claude-skills

# Or update all plugins/skills from all marketplaces
/plugin update
```

After updating, restart Claude Code to load the changes.

## Available Skills

### TypeScript Development

- **reviewing-typescript** - Review and improve TypeScript code quality, type safety, and best practices
- **designing-apis** - Design and review REST/GraphQL APIs with best practices, consistency, and scalability
- **planning-tests** - Plan and implement comprehensive testing strategies for software projects
- **reviewing-architecture** - Review and design software architecture for scalability, maintainability, and best practices
- **designing-databases** - Design and review database schemas, queries, and data models for optimal performance and integrity

### UX & Design

- **improving-ux** - Analyze and improve user experience and growth using evidence-based UX psychology principles

## Usage

Once installed, skills will automatically trigger when you make relevant requests to Claude Code. You can also explicitly reference skills in your prompts.

## Development

### Directory Structure

```
claude-skills/
├── .claude-plugin/
│   └── marketplace.json
├── skills/
│   └── improving-ux/
│       ├── SKILL.md
│       └── references/
│           ├── checklist.md
│           └── principles.md
└── README.md
```

### Validating Changes

Before pushing changes to GitHub, validate the marketplace configuration:

```bash
# Navigate to the repository
cd /path/to/claude-skills

# Validate marketplace.json
claude plugin validate .
```

This will check:
- JSON syntax
- Required fields (name, owner, plugins)
- Plugin source paths
- Schema compliance

### Adding New Skills

1. Create a new directory under `skills/`
2. Add a `SKILL.md` file with YAML frontmatter
3. Update `.claude-plugin/marketplace.json` to add the new skill to the `plugins` array
4. **Validate the configuration**: Run `claude plugin validate .`
5. Commit and push to GitHub

### Updating Skills

When you make changes to existing skills:

1. Edit skill files in `skills/skill-name/`
2. **Validate the configuration**: Run `claude plugin validate .`
3. Commit and push to GitHub
4. On other devices, update the skills:
   ```bash
   /plugin update
   ```
5. Restart Claude Code to load the changes

## License

MIT

## Author

[@jmblog](https://github.com/jmblog)
