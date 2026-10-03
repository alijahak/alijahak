# Case study: making a brokerage MCP integration survive OAuth expiry, hangs and accidental writes

**Role:** sole developer
**Stack:** Python, MCP client, OAuth 2.0 (RFC 6749 refresh), pytest
**Context:** a market-data research pipeline that reads from a brokerage's production MCP server. It runs in shadow mode: it never places orders.
**Code:** private repository. This page describes it and gives its numbers.

## The problem

The pipeline has to collect data in a fixed window around the market open. Three things kept costing that window:

1. **Expired sign-ins found too late.** The stored token looked fine locally, but the server answered `initialize` with HTTP 401. The pipeline found out during the market window, with no automated way to recover.
2. **Silent hangs.** When the secret store or the auth chain stalled, the run just sat there instead of failing.
3. **Write risk.** A brokerage MCP server exposes order tools next to data tools. A data collector must never be able to reach them.

## What I built

**Sign-in as a recoverable state machine**

- Lifecycle states form a closed set. A live 401 overrides a locally "OK" token: the state becomes "refresh needed", never "ready".
- A standard refresh-token grant (RFC 6749) runs against the token endpoint found by discovery, not a hard-coded one.
- If refresh can't recover, the pipeline hands off to an interactive re-login. A successful sign-in produces an "auth-ready" certificate, and the pipeline checks that certificate *before* the market window instead of finding out during it.
- Diagnostics show presence, status and expiry only. They never print a token value.

**Fail fast instead of hanging**

- Every stage has an explicit timeout: secret resolution 5 s, config validation 2 s, MCP handshake 10 s, auth readiness 10 s.
- Any failure writes a diagnostic JSON and exits non-zero, with the exact manual step to take next.

**Read-only by construction**

- The collector may call allowlisted MCP tools only.
- The fallback data adapter has a mutation guard that blocks 17 order-placing functions. A test proves the guard holds.

**No made-up data**

- With no authenticated source, the collector records `AUTH_REQUIRED` and writes zero rows. It never fills gaps with estimates.

## Results

- The recurring "operator must fix auth" dead end became a recoverable pre-market step.
- A hung run now fails within seconds, with a reason.
- The read-only certification run passed 99 targeted tests. The repository has 37 MCP-focused test files.

## What this means for your project

If your MCP server or agent touches anything that can move money or change records:

- write access should be impossible by construction, not by convention;
- auth failures should be states with a recovery path, not crashes;
- every failure should say what to do next.
