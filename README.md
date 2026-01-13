# Claude Code Agents & Skills

A collection of custom agents and skills (slash commands) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Anthropic's official CLI tool for Claude.

## What's Included

### Agents

| Agent | Description |
|-------|-------------|
| [`qa-audit-analytics`](agents/qa-audit-analytics.md) | QA auditor for data analytics projects. Verifies calculations, cross-references claims against source data, and flags discrepancies before outputs go into presentations or reports. |

### Skills (Slash Commands)

| Skill | Description |
|-------|-------------|
| [`humanize`](commands/humanize.md) | Transform AI-generated content into natural, human-sounding text. Identifies and fixes AI patterns like overused words, uniform sentence lengths, and robotic phrasing. |

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured

### Installing Agents

Copy agent files to your Claude Code agents directory:

```bash
# Create agents directory if it doesn't exist
mkdir -p ~/.claude/agents

# Copy agent file(s)
cp agents/qa-audit-analytics.md ~/.claude/agents/
```

Agents are automatically invoked by Claude Code when relevant tasks are detected. You can also reference them explicitly in your prompts.

### Installing Skills (Commands)

Copy command files to your Claude Code commands directory:

```bash
# Create commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy command file(s)
cp commands/humanize.md ~/.claude/commands/
```

Once installed, invoke skills using the slash command syntax:

```
/humanize
```

## Usage Examples

### QA Audit Analytics Agent

The QA audit agent is automatically invoked when you have a data analysis project that needs verification. You can also explicitly request an audit:

```
Audit the analysis in results.md for accuracy before I send it to the client.
```

The agent will:
1. Discover all analysis files (scripts, data, reports)
2. Extract numerical claims from output files
3. Trace each claim back to source data
4. Verify calculations, percentages, and counts
5. Generate a discrepancy report with severity levels

### Humanize Skill

Use the `/humanize` command to transform AI-generated text:

```
/humanize

[Paste your AI-generated content here]
```

The skill analyzes text for AI patterns and rewrites it to sound more natural by:
- Varying sentence lengths (adding short punchy sentences)
- Replacing flagged vocabulary (delve, tapestry, leverage, etc.)
- Adding human elements (contractions, specifics, opinions)
- Breaking predictable structures

## Contributing

Feel free to submit issues or pull requests with improvements to existing agents/skills or suggestions for new ones.

## Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)

## License

MIT License - feel free to use and modify these agents and skills for your own projects.
