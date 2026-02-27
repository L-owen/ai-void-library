# AI Void Library

A comprehensive collection of AI agent resources including skills, MCP servers, rules, and prompts for Claude Code and other AI agents.

## Overview

This repository serves as a centralized library for reusable AI agent components designed to enhance agent capabilities with specialized knowledge, workflows, and integrations.

## Structure

```
ai-void-library/
├── skills/              # Claude Code skills
│   ├── codehub-review/      # Code review workflow skill
│   └── java-code-review/    # Java code review skill
├── mcp/                 # MCP (Model Context Protocol) servers
├── rules/               # Agent rules and guidelines
├── prompts/             # Reusable prompt templates
└── tools/               # Utility scripts and tools
```

## Contents

### Skills

Specialized workflows and knowledge packages for Claude Code.

#### codehub-review

Systematic code review workflow for merge requests using codehub-review-mcp.

- **Focus**: Review process and workflow
- **Features**:
  - Fetch MR information via MCP
  - Complexity-based review (simple/medium/complex)
  - Local patch application for complex changes
  - Integration with coding standards skills

See [skills/codehub-review/](skills/codehub-review/) for details.

#### java-code-review

Comprehensive Java code review skill with architecture, security, and performance guidelines.

- **Focus**: Java-specific code quality and best practices
- **Features**:
  - Architecture principles review
  - Security checklist
  - Performance patterns
  - Stability checks
  - IntelliJ best practices

See [skills/java-code-review/](skills/java-code-review/) for details.

### MCP Servers

Model Context Protocol servers for extending agent capabilities with external tools and data sources.

*Coming soon...*

### Rules

Agent behavior rules, coding standards, and review guidelines.

*Coming soon...*

### Prompts

Reusable prompt templates for common AI agent tasks.

*Coming soon...*

### Tools

Utility scripts and tools to support AI agent workflows.

*Coming soon...*

## Usage

### Using Skills

To use skills from this repository with Claude Code:

1. Copy the skill directory to your Claude Code skills folder
2. Restart Claude Code
3. The skill will be automatically available

Example:
```bash
cp -r skills/codehub-review ~/.claude/skills/
```

### Using MCP Servers

To use MCP servers:

1. Install the MCP server dependencies
2. Configure the server in your Claude Code settings
3. Restart Claude Code

See individual MCP server documentation for setup instructions.

## Development

This is an active collection that will be continuously updated with:

- ✅ **Skills** - Domain-specific workflows and knowledge
- 🚧 **MCP Servers** - External integrations and tools
- 🚧 **Rules** - Agent behavior guidelines
- 🚧 **Prompts** - Reusable prompt templates
- 🚧 **Tools** - Supporting utilities

## Contributing

Suggestions and improvements are welcome through issues and pull requests.

## License

Please refer to individual components for specific licensing information.

## Contact

Maintained by [L-owen](https://github.com/L-owen)
