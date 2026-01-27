# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

A collection of skills for Claude.ai and Claude Code for working with Polar - the open-source payment infrastructure platform. Skills are packaged instructions and scripts that extend Claude's capabilities.

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    AGENTS.md             # Optional: full compiled document
    references/           # Optional: supporting documentation
    scripts/              # Optional: executable scripts
  {skill-name}.zip        # Optional: packaged for distribution
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `polar-developer-guide`, `polar-best-practices`)
- **SKILL.md**: Always uppercase, always this exact filename
- **References**: `kebab-case.md` (e.g., `sdk-reference.md`, `webhook-events.md`)
- **Zip file**: Must match directory name exactly: `{skill-name}.zip`

### SKILL.md Format

```markdown
---
name: {skill-name}
description: |
  {One paragraph describing what the skill does and when to use it.
  Include trigger phrases and specific scenarios that activate the skill.}
---

# {Skill Title}

{Brief description of what the skill does.}

## Quick Start

{Essential setup steps}

## Core Features

{Main functionality with code examples}

## References

- [Reference Name](references/file.md) - Description
```

### Best Practices for Context Efficiency

Skills are loaded on-demand — only the skill name and description are loaded at startup. The full `SKILL.md` loads into context only when the agent decides the skill is relevant. To minimize context usage:

- **Keep SKILL.md under 500 lines** — put detailed reference material in separate files
- **Write specific descriptions** — helps the agent know exactly when to activate the skill
- **Use progressive disclosure** — reference supporting files that get read only when needed
- **File references work one level deep** — link directly from SKILL.md to supporting files

### Creating the Zip Package

After creating or updating a skill:

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### End-User Installation

Document these two installation methods for users:

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
Add the skill to project knowledge or paste SKILL.md contents into the conversation.

## Polar-Specific Guidelines

When creating skills for Polar integration:

### SDK Patterns

Always show both TypeScript and Python examples where applicable:

```typescript
// TypeScript
import { Polar } from "@polar-sh/sdk";
const polar = new Polar({ accessToken: process.env.POLAR_ACCESS_TOKEN });
```

```python
# Python
from polar_sdk import Polar
polar = Polar(access_token=os.environ["POLAR_ACCESS_TOKEN"])
```

### Environment Configuration

Always include both sandbox and production configurations:

```bash
# Development
POLAR_ACCESS_TOKEN=pat_sandbox_xxx
POLAR_SERVER=sandbox

# Production
POLAR_ACCESS_TOKEN=pat_xxx
POLAR_SERVER=production
```

### Webhook Handling

Always emphasize signature verification:

```typescript
import { validateEvent, WebhookVerificationError } from "@polar-sh/sdk/webhooks";

try {
  const event = validateEvent(payload, headers, secret);
} catch (e) {
  if (e instanceof WebhookVerificationError) {
    // Reject request
  }
}
```

### Framework Adapters

Reference the official adapters when available:
- `@polar-sh/nextjs` - Next.js
- `@polar-sh/express` - Express.js
- `polar-sdk` - Python/FastAPI

### MCP Server

Include MCP configuration for AI agent integration:
- Production: `https://mcp.polar.sh/mcp/polar-mcp`
- Sandbox: `https://mcp.polar.sh/mcp/polar-sandbox`
