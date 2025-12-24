# Claude Skills

Personal collection of custom Claude Code skills for enhanced productivity and specialized workflows.

## Installation

### Add this marketplace to Claude Code

```bash
/plugin marketplace add https://github.com/jmblog/claude-skills.git
```

### Install skills

```bash
# Install UX Growth skill
/plugin install ux-growth@claude-skills
```

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

### Adding New Skills

1. Create a new directory under `skills/`
2. Add a `SKILL.md` file with YAML frontmatter
3. Update `.claude-plugin/marketplace.json`
4. Commit and push to GitHub

## License

MIT

## Author

[@jmblog](https://github.com/jmblog)
