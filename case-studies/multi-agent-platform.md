# Case study: a multi-agent platform that writes, reviews and merges its own code

**Role:** sole developer and operator
**Stack:** Python, pytest, GitHub, Claude, OpenAI Codex and Gemini agents, MCP
**Period:** April 2026 to now; runs every day
**Code:** private repository. This page describes it and gives its numbers.

## The problem

I wanted coding agents from three vendors to work on one codebase on a schedule: write fixes, review each other's work, and merge. That had to happen without a human in every loop, and without any agent approving its own changes.

## What I built

- **A scheduled fleet.** 16 Claude roles, plus OpenAI Codex and Gemini automations. Each has one job, such as building, reviewing, integrating or auditing.
- **An independent-review gate.** A pull request merges only when an agent that didn't write it has published a verdict tied to the exact commit SHA, with a matching receipt file. A negative verdict on that commit always wins. A new commit needs a new review.
- **An integration lock and an admission check.** Every merge runs under both, so two agents never merge into a moving target.
- **A tool-permission layer.** It decides which agent may call which MCP tool, by the tool's safety class.
- **Fail-closed defaults.** Wherever a safety decision depends on state that might not be readable, unreadable means "stop". For example, the kill switch treats a state file it can't read as "killed".

## By the numbers

- 2,352 commits on the main branch since April 12, 2026
- 280+ merged pull requests
- About 1,000 Python modules and 1,400 test files

## Three problems worth knowing about

**1. A process that looks dead may not be.**
On Windows, asking for a process handle can fail with "access denied" on a live, protected process. Treating every failed lookup as "dead" let a lock be taken from a holder that was still running. The fix: only "invalid parameter" (error 87) proves a process ID is gone. Anything else means alive or unknown, and a lock is reclaimed only on proven death.

**2. A heartbeat isn't progress.**
A lock holder can be alive and parked for hours, waiting on a human prompt. Checks that ask "does the process exist?" pass the whole time. A lease or a heartbeat proves existence, not progress, and a recovery plan has to treat those as different questions.

**3. A safety stop has to fail closed.**
A kill switch that answered "not killed" when its state file couldn't be read would keep the fleet running at the exact moment an operator tried to stop it. It now answers "killed" and logs the error.

## What this means for your project

Client work gets the same habits:

- a test for every MCP tool;
- permission decisions that fail closed;
- review evidence tied to the exact revision it covers.
