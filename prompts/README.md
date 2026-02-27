# Prompt Templates

This directory contains reusable prompt templates for common AI agent tasks.

## Directory Structure

```
prompts/
├── code-generation/     # Code generation prompts
│   ├── api-endpoint.md
│   └── unit-test.md
├── code-review/        # Code review prompts
│   └── standard-review.md
├── documentation/      # Documentation generation prompts
│   ├── api-docs.md
│   └── readme.md
├── debugging/          # Debugging assistance prompts
│   └── error-analysis.md
├── refactoring/        # Code refactoring prompts
│   └── cleanup.md
└── README.md          # This file
```

## Prompt Categories

### Code Generation
Templates for generating various types of code:
- API endpoints
- Unit tests
- Data models
- Utility functions

### Code Review
Structured prompts for conducting code reviews:
- Standard review流程
- Security-focused review
- Performance review
- Architecture review

### Documentation
Templates for generating documentation:
- API documentation
- README files
- Inline code comments
- Architecture diagrams

### Debugging
Prompts to assist with debugging:
- Error analysis
- Log interpretation
- Root cause analysis
- Troubleshooting guides

### Refactoring
Templates for code refactoring:
- Code cleanup
- Performance optimization
- Design pattern application
- Legacy code modernization

## Adding New Prompts

1. Create the appropriate category directory if it doesn't exist
2. Add prompt templates in Markdown format
3. Include:
   - Clear description of when to use the prompt
   - Required parameters/variables
   - Example usage
   - Expected output format
4. Update this README with the new prompt's description

## Prompt Template Format

Each prompt should follow this structure:

```markdown
# Prompt Name

## Description
Brief description of what this prompt does and when to use it.

## Parameters
- `param1`: Description
- `param2`: Description

## Template
\`\`\`
The actual prompt template with {{parameters}}
\`\`\`

## Example
\`\`\`
Example usage with actual values
\`\`\`

## Expected Output
Description of what the agent should produce.
\`\`\`

## Existing Prompts

Coming soon...
