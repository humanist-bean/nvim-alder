---
name: nexus-development-debugging
description: Use for all Nexus development and debugging tasks across NextGen-API, NextGen-UI, and Nexus-HeavyLifter-API, including pricing, order, release, SQL, repository, and integration issues.
---

# Nexus Development and Debugging

This skill is the default operating guide for all Nexus development and debugging work.

## Scope

Use this skill for any task in the Nexus workspace, including:
- API development in `NextGen-API`
- UI development in `NextGen-UI`
- service/integration work in `Nexus-HeavyLifter-API`
- data, pricing, order, release, and stored-procedure investigations

## Core Architecture Rule

Instead of modifying or working around stored procedures, rebuild them as methods in the appropriate Repository class in the API. If no such repository exists, create one.

Implementation expectations:
- Prefer repository methods + service logic over adding new SP dependency.
- Keep DB access encapsulated in repository classes.
- Keep service layer focused on business logic, orchestration, and validation.
- For risky legacy SP behavior, migrate logic incrementally and preserve external behavior.

## Database Access Policy

The agent may use database access only for safe investigation and validation.

Required DB access model:
- Dedicated read-only account (for example: `nexus_ai_reader`)
- `CONNECT`
- `VIEW DEFINITION` (database-wide or explicit object grants)
- Controlled `EXECUTE` only on approved/allowlisted procedures used for diagnostics

Must not be granted:
- `INSERT`, `UPDATE`, `DELETE`, `MERGE`
- `ALTER`, `CREATE`, `DROP`, `CONTROL`, `TAKE OWNERSHIP`
- elevated roles such as `db_owner`, `db_ddladmin`, `db_datawriter`

Operational rules:
- Never execute DDL/DML from agent workflows.
- Never change schema, procs, functions, triggers, or data directly in DB.
- If a DB object change is required, output a SQL patch script for DBA execution.
- Never store credentials in repo files, skill files, or source-controlled config.
- Use environment variables or local secret manager (for example: `NEXUS_DB_READONLY_CONN`).

## Debugging Workflow

Use this loop for Nexus bugs:

1. Reproduce and trace
- Identify user-visible symptom.
- Trace UI -> API controller -> service -> repository/SP path.

2. Inspect implementation
- Read relevant TS/C# and repository logic.
- If needed, read stored procedure/function definitions via read-only access.

3. Find root cause
- Identify exact branching/data condition causing failure.
- Confirm whether the issue is in app code, query shape, mapping, or legacy SP logic.

4. Implement fix in codebase
- Prefer repository/service implementation over SP changes.
- Add a new repository when no appropriate repository exists.
- Keep changes minimal, scoped, and convention-aligned.

5. Validate
- Build and run targeted tests where possible.
- For DB-related behavior, use read-only verification queries and allowlisted proc execution.

6. Report clearly
- State root cause, files changed, and why approach is safe.
- If DB patch needed, provide script and rollback notes for DBA.

## Output Requirements for DB-Related Bugs

When the issue involves SQL/SP behavior, include:
- impacted UI/API path
- relevant procedure/query location
- root cause condition
- repository/service implementation plan (or code changes)
- verification steps and expected outcomes
- optional DBA SQL patch script only when unavoidable

## Safety and Change Discipline

- Keep edits surgical: touch only files needed for requested outcome.
- Do not perform unrelated refactors.
- Preserve existing conventions unless user asks otherwise.
- Do not commit or push unless explicitly requested.
