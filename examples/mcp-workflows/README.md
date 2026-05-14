# MCP workflow examples

Examples of engineering workflows using Model Context Protocol (MCP) integrations with GitHub, Jira, and DataDog. MCP enables AI tools to query these systems directly rather than requiring the engineer to copy-paste context manually.

---

## What MCP changes

Without MCP, the engineer copies relevant context (PR diffs, ticket content, log excerpts) and pastes it into the AI session. This adds friction and creates a gap between the live system state and what the model sees.

With MCP, the AI tool can query GitHub, Jira, or DataDog directly within the session, reducing context preparation overhead and enabling analysis of more complete data (e.g., the full PR comment history, a complete log range, all linked tickets in a sprint).

**What MCP does not change:** The workflow logic, validation gates, and quality criteria are identical with or without MCP. MCP changes the mechanics of context gathering — not the engineering process or the human review requirements.

---

## Setup assumption

These examples assume MCP server connections are already configured in the AI tool. MCP server setup is outside the scope of this repository. Refer to:

- [Anthropic Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code) for Claude Code MCP configuration
- The [MCP specification](https://modelcontextprotocol.io) for building custom MCP servers
- Your organization's internal MCP setup guide if one exists

---

## Index

| Example | Integration | Workflow | Description |
|---|---|---|---|
| [GitHub PR review](github-integration.md) | GitHub | [PR Review](../../workflows/engineering/pr-review.md) | PR review using GitHub MCP — diff, description, and comments fetched directly |
| [Jira ticket refinement](jira-integration.md) | Jira | [Ticket Refinement](../../workflows/delivery/ticket-refinement.md) | Ticket refinement using Jira MCP — ticket content and linked issues fetched directly |
| [DataDog investigation](datadog-integration.md) | DataDog | [DataDog Investigation](../../workflows/operations/datadog-investigation.md) | Alert investigation using DataDog MCP — metrics and logs queried within the session |
