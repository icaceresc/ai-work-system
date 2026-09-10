# Environment Adapter — Example

> **Scope:** environment-specific routing and restrictions for one working environment.
> **Principle:** keep this file small. It points to approved environment owners instead of duplicating their contents.

This file is a template. Copy it outside this repository, into your own environment, and adapt it. The Worker Core (`worker/worker_core.md`) treats it as the entry point for environment-specific configuration.

An Environment Adapter is **optional**. A simple environment may not need one.

It defines a **logical contract, not a loading mechanism**. How its content reaches the Worker depends on the product: it may be referenced as a separate file, or pasted inline into the same instructions file as the Worker Core. The README documents what has been verified for each executor.

## Contract

```text
Worker Core
    ↓
Environment Adapter (this file)
    ↓
router/config local (optional)
    ↓
tools/helpers/references owned by this environment
```

The Worker Core stays generic and portable. This adapter carries whatever is specific to one environment: which systems and tools exist, which local sources of truth are authoritative, which access is read-only, which paths are readable or writable, which restrictions apply, and how to route toward private resources.

It exists so the Worker does not guess at infrastructure, does not rediscover the environment ad hoc on every task, and does not push private configuration into the portable Core.

## 1. Environment router (optional)

If this environment has an index of tools, data sources or systems, point to it here instead of duplicating its contents:

`<path or reference to your router, if one exists>`

Read it on demand, not at session start. Read it before the first action that touches a system it owns.

If no router exists, list the relevant systems directly in this file. A router only pays off when the environment is complex enough that listing everything here would bloat the adapter. For that case a structured index — a small JSON file mapping each system to its approved reference or helper — has worked well in practice: the adapter stays short, and the Worker follows one route per system instead of exploring.

If the router is missing or unreadable, report the path and stop. Do not invent a replacement.

## 2. Routing discipline

Use the router (if any) only as an index.

Follow the references it provides instead of copying:
- endpoints;
- paths;
- connection coordinates;
- dictionary/schema contents.

Do not explore infrastructure ad hoc when an approved route already exists.

## 3. Data access (if applicable)

For any data system available in this environment:
- state whether access is read-only or read-write;
- list allowed operations explicitly (e.g. `SELECT` only);
- if a write is required but not allowed, draft it for human execution instead of running it;
- verify schema/field meaning through an approved source before relying on it.

## 4. Secrets and credentials

Do not read or copy credential files or secrets unless a task explicitly authorizes the exact source and doing so is necessary.

Use environment variables or an approved local mechanism when credentials are required.

Non-secret connection coordinates belong to this adapter or its router, not to generated code.

## 5. Filesystem

List any paths outside the working directory that are authorized for reading, and under what conditions:
- `<path>` — `<why it's readable>`

Writing outside the current working directory remains prohibited unless explicitly authorized.

## 6. Execution environment

State what this environment does and does not allow by default, for example:
- deploying to remote/internal systems;
- opening remote sessions (SSH or equivalent);
- starting local replicas of external services.

Unless explicitly authorized by a task, none of the above.

## 7. Project precedence

A project may add stricter or more specific rules through its own local project configuration.

Project configuration must not duplicate this adapter unless the project owns a genuinely project-specific exception.
