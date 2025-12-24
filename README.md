# Claude Skills

Personal collection of custom Claude Code skills for enhanced productivity and specialized workflows.

## Installation

### Add this marketplace to Claude Code

```bash
/plugin marketplace add https://github.com/jmblog/claude-skills.git
```

### Install skills

```bash
/plugin install ux-growth@claude-skills
```

### Update skills

To get the latest version of skills from the marketplace:

```bash
# Update a specific skill
/plugin update ux-growth@claude-skills

# Or update all plugins/skills from all marketplaces
/plugin update
```

After updating, restart Claude Code to load the changes.

## Available Skills

### UX Growth

Analyze and improve user experience and growth using evidence-based UX psychology principles.

**Use cases:**
- UX analysis of websites or applications
- Growth optimization recommendations
- Conversion rate improvement strategies
- Psychological design pattern applications
- User behavior analysis
- Evidence-based design improvements

**What's included:**
- 25+ UX psychology principles with detailed explanations
- Systematic UX analysis framework with 10 evaluation categories
- Implementation patterns and anti-patterns
- Prioritization frameworks for improvements
- A/B testing recommendations

**Trigger examples:**
- "Analyze the UX of this landing page"
- "How can I improve conversion rates using psychology?"
- "What psychological principles should I apply to this design?"
- "Help me increase user engagement"

## Usage

Once installed, skills will automatically trigger when you make relevant requests to Claude Code. You can also explicitly reference skills in your prompts.

## Development

### Directory Structure

```
claude-skills/
├── .claude-plugin/
│   └── marketplace.json
├── skills/
│   └── ux-growth/
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
4. On other devices, update the skill:
   ```bash
   /plugin update ux-growth@claude-skills
   ```
5. Restart Claude Code to load the changes

## License

MIT

## Author

[@jmblog](https://github.com/jmblog)
