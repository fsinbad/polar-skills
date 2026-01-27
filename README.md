# Polar Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities for working with [Polar](https://polar.sh) - the open-source payment infrastructure platform.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### polar-developer-guide

Comprehensive integration guide for Polar. Covers SDK setup, checkout flows, webhooks, subscriptions, benefits, customer management, and MCP server configuration.

**Use when:**
- Setting up Polar from scratch
- Implementing checkout flows (links, embedded, API)
- Handling webhooks and signature verification
- Managing subscriptions, products, and benefits
- Integrating with frameworks (Next.js, Express, FastAPI)
- Configuring the Polar MCP server for AI agents

**Categories covered:**
- Quick Start & SDK Installation
- Framework Integration (Next.js, Express, FastAPI)
- Checkout Implementation
- Webhook Events & Handling
- Benefits & Entitlements
- Customer Management
- Polar MCP Server Configuration
- API Patterns & Best Practices

### polar-testing

Guide for testing Polar payment integrations using the sandbox environment.

**Use when:**
- Setting up the Polar sandbox for development
- Testing checkout flows without real payments
- Using Stripe test cards with Polar
- Writing integration tests for payment flows
- Testing webhooks locally with ngrok
- Mocking Polar in unit tests
- Setting up CI/CD pipelines

**Categories covered:**
- Sandbox Environment Setup
- Test Card Numbers (success, decline, 3DS)
- Local Webhook Testing with ngrok
- Integration Test Patterns
- Mocking Polar SDK
- CI/CD Configuration
- Debugging Tips

### polar-migration

Guide for migrating to Polar from other payment platforms.

**Use when:**
- Planning a migration from Stripe Billing, Paddle, or Lemon Squeezy
- Migrating customer data and subscriptions
- Running parallel systems during transition
- Mapping products and pricing between platforms
- Communicating migration to customers

**Categories covered:**
- Migration Strategies (hard cutover, gradual, new customers only)
- Stripe Billing Migration
- Paddle Migration
- Lemon Squeezy Migration
- Customer Communication Templates
- Parallel Running Patterns
- Post-Migration Verification

## Installation

### Claude Code

```bash
cp -r skills/polar-developer-guide ~/.claude/skills/
cp -r skills/polar-testing ~/.claude/skills/
cp -r skills/polar-migration ~/.claude/skills/
```

### claude.ai

Add the skill to project knowledge or paste `SKILL.md` contents into the conversation.

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**
```
Help me set up Polar in my Next.js app
```
```
How do I test Polar webhooks locally?
```
```
Help me migrate from Stripe to Polar
```
```
Set up the Polar MCP server for Claude
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent (required)
- `AGENTS.md` - Full compiled document for agents (optional)
- `references/` - Supporting documentation (optional)
- `metadata.json` - Version and organization info (optional)

## Resources

- [Polar Documentation](https://polar.sh/docs)
- [Polar GitHub](https://github.com/polarsource/polar)
- [Polar Sandbox](https://sandbox.polar.sh)
- [Polar MCP Integration](https://polar.sh/docs/integrate/mcp)

## License

MIT
