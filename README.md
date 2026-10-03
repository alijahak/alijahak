### Alijah Konikowski

I build **production-ready MCP servers** and AI agent integrations for Claude and ChatGPT: tool design that matches what each tool really does, OAuth done to the spec, and a test for every tool.

**Open source**

- [**github-review-queue-mcp**](https://github.com/alijahak/github-review-queue-mcp): an MCP server for your GitHub review queue. It runs locally, or remotely as its own OAuth 2.1 authorization server: PKCE, dynamic registration, a consent screen per client, audience-bound tokens, refresh rotation. 27 tests prove the flow and tenant isolation.
- [**mcp-readiness**](https://github.com/alijahak/mcp-readiness): checks an MCP server against the Claude Connectors Directory and ChatGPT app directory rules before you submit. I used it to [scan 200 servers from the official MCP Registry](https://github.com/alijahak/mcp-readiness/blob/main/scans/2026-10/README.md): only 31% of the public ones had no directory-blocking finding.

**Case studies**

- [A multi-agent platform that writes, reviews and merges its own code](case-studies/multi-agent-platform.md): Claude, OpenAI Codex and Gemini agents behind exact-commit independent review; 280+ merged PRs.
- [A brokerage MCP integration that survives OAuth expiry, hangs and accidental writes](case-studies/brokerage-mcp-oauth.md): recoverable token lifecycle, fail-fast timeouts, read-only by construction.

**Work with me:** fixed-price MCP server builds and pre-submission readiness checks.
