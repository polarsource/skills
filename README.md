# Polar Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities for working with [Polar](https://polar.sh) - the open-source payment infrastructure platform.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### setup-polar

Interactive onboarding wizard to set up Polar payments from scratch — installs the Polar MCP server, creates an organization and first product, then hands off to `polar-integration` to generate framework-specific endpoints.

**Use when:**
- Starting fresh with Polar in a project
- "How do I get started with Polar?"
- Adding payments/subscriptions/checkout to an app for the first time

### polar-integration

Canonical guide for wiring up Polar's three core HTTP endpoints — Checkout, Customer Portal, and Webhooks — using `@polar-sh/sdk` directly. Recipes are framework-agnostic (Web Standards `Request`/`Response`) with per-framework adjustments.

**Use when:**
- Adding a Checkout, Customer Portal, or Webhooks endpoint
- Working in any TS/JS framework: Next.js, Express, Hono, Astro, SvelteKit, Remix, TanStack Start, Nuxt, Fastify, Elysia, Deno, Supabase Edge Functions, Cloudflare Workers, Bun
- Verifying webhook signatures with `validateEvent`

### polar-testing

Guide for testing Polar payment integrations using the sandbox environment.

**Use when:**
- Setting up the Polar sandbox for development
- Testing checkout flows without real payments
- Using Stripe test cards with Polar
- Writing integration tests for payment flows
- Testing webhooks locally with the Polar CLI (`polar listen`, `polar trigger`)
- Generating webhook fixtures for handler tests
- Mocking Polar in unit tests
- Setting up CI/CD pipelines

### polar-migration

Guide for migrating to Polar from other payment platforms.

**Use when:**
- Planning a migration from Stripe Billing, Paddle, or Lemon Squeezy
- Migrating customer data and subscriptions
- Running parallel systems during transition
- Mapping products and pricing between platforms
- Communicating migration to customers

## Installation

### Claude Code

```bash
cp -r skills/setup-polar ~/.claude/skills/
cp -r skills/polar-integration ~/.claude/skills/
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
Write tests for my Polar webhook handler
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
- `references/` - Supporting documentation (optional)
- `scripts/` - Executable scripts (optional)

## Resources

- [Polar Documentation](https://polar.sh/docs)
- [Polar GitHub](https://github.com/polarsource/polar)
- [Polar Sandbox](https://sandbox.polar.sh)
- [Polar MCP Integration](https://polar.sh/docs/integrate/mcp)

## License

MIT
