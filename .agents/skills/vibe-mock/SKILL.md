---
name: vibe-mock
description: Lower Vibe pseudocode into mock implementations — real, callable code with synthetic data and stub logic that preserves the original vibe narratives for later production lowering via /vibe. Use when the user invokes /vibe-mock or asks to mock out vibe code into something executable.
---

vibe-mock sits between vibe-along and vibe in the development lifecycle. Where vibe-along captures intent as pseudocode and vibe lowers that pseudocode into production implementations, vibe-mock produces an intermediate executable layer: real, callable code backed by mock data and stub logic that behaves as the vibe artifacts intend — without requiring real infrastructure, services, or data stores.

The mock code is designed to be temporary scaffolding. Every mock implementation preserves the original vibe narrative so that `/vibe` can later replace it with a real implementation.

## Activation

Activate on `/vibe-mock`. Process the requested scope in a single pass.

## Scope

- Default to the entire repository.
- If the user explicitly names a path or component, limit the run to that scope.
- Search tracked and unignored working-tree source, including untracked non-ignored files.
- If available, use the current git diff to understand vibe-scoped modifications for further clarity of user intent.
- Exclude generated output, dependencies, vendor code, caches, submodule internals, and similar derived or third-party content from vibe parsing.
- If the declared scope contains no Vibe artifacts, make no changes and report that none were found.

## Recognizing Vibe

Use the same artifact recognition as the vibe skill:

1. Files ending in `.vibe`.
2. Inline regions delimited by `startvibe` / `endvibe`.
3. Single-line comments containing `vibe:`.

## Workflow

### 1. Discover the complete inventory

Find every in-scope Vibe artifact before editing. For each artifact, identify:

- the behavior or structure it requests;
- its likely target language and destination;
- the data shapes, return types, and call signatures implied;
- dependencies on other Vibe artifacts or existing code; and
- ambiguities that would change the mock's observable behavior.

Build a lightweight dependency graph and plan to implement foundations before consumers.

### 2. Establish sufficient clarity

If a vibe artifact is too ambiguous to produce a useful mock — for example, a stub that says `# vibe: do the thing` with no surrounding context — ask the user before guessing. A mock that returns the wrong shape is worse than no mock.

### 3. Generate mock implementations

For each vibe artifact, produce real, callable code that:

- **Matches the intended interface** — function signatures, class shapes, module exports, and type annotations should reflect the vibe intent as faithfully as possible.
- **Returns plausible synthetic data** — use hardcoded or deterministically generated values that match the expected data shapes. Prefer realistic-looking values over obvious placeholders (e.g., `"jane.doe@example.com"` over `"string"`).
- **Simulates expected behavior** — if the vibe describes conditional logic, branching, error cases, or state transitions, the mock should approximate those paths well enough that callers can exercise them.
- **Is immediately executable** — the mock code must run without external dependencies that aren't already in the project. No real database connections, no real API calls, no real authentication. Import only what the project already has or what the standard library provides.

#### Mock markers

Every mock implementation must be clearly identified. Use two mechanisms:

1. **A `VIBE-MOCK` banner comment** at the top of each generated or modified file:
   ```
   # ┌─────────────────────────────────────────────────┐
   # │ VIBE-MOCK: This file contains mock              │
   # │ implementations generated from vibe pseudocode. │
   # │ Run /vibe to lower to real implementations.     │
   # └─────────────────────────────────────────────────┘
   ```
   Adapt the comment syntax to the target language.

2. **Inline `vibe-mock:` annotations** on each mock function, class, or block, describing what it stubs and referencing the original vibe intent:
   ```python
   def get_user(user_id: str) -> dict:
       # vibe-mock: returns synthetic user data
       # vibe: fetch user record from database by id
       return {
           "id": user_id,
           "name": "Jane Doe",
           "email": "jane.doe@example.com",
           "created_at": "2025-01-15T09:30:00Z",
       }
   ```

The `vibe-mock:` comment describes what the mock does. The original `vibe:` comment (or equivalent narrative) is preserved directly below it to document the real intent.

### 4. Preserve vibe narratives

This is critical. The original vibe intent must survive the mock lowering so that `/vibe` can later replace the mock with a real implementation.

- **For `.vibe` files**: Do not delete the `.vibe` file. Create or update the target source file(s) with mock implementations, but leave the `.vibe` file intact.
- **For inline `startvibe`/`endvibe` blocks**: Replace the block contents with mock code, but convert the original vibe narrative into `vibe:` comments within the mock body. Remove the `startvibe`/`endvibe` delimiters and replace them with `startvibe-mock`/`endvibe-mock` delimiters so the region is identifiable as mocked:
  ```python
  # startvibe-mock
  # vibe-mock: returns cached config with synthetic defaults
  # vibe: load config from remote config service, cache locally, refresh on TTL
  def get_config() -> dict:
      return {"feature_flags": {"dark_mode": True}, "ttl": 300}
  # endvibe-mock
  ```
- **For single-line `vibe:` comments**: Add a `vibe-mock:` comment and mock code directly adjacent, keeping the original `vibe:` comment in place:
  ```python
  # vibe-mock: using hardcoded rate limit of 100 req/min
  # vibe: enforce per-user rate limiting with sliding window
  rate_limit = 100
  ```

### 5. Handle data dependencies

When multiple vibe artifacts describe related data (e.g., a user service and an order service that references users), ensure the mock data is internally consistent:

- Use the same synthetic identifiers across mocks (e.g., if a mock user has `id: "usr_001"`, a mock order should reference `user_id: "usr_001"`).
- If a vibe artifact describes a data store, create a simple in-memory dictionary or list that other mocks can reference.
- Prefer a single shared mock data module when the project has several interrelated mocks, rather than scattering hardcoded values.

### 6. Validate

The mock code must be syntactically valid and, where tooling exists, pass basic checks:

- Run available linting, type-checking, or syntax validation.
- If the project has a build step, confirm the mock code does not break it.
- If existing tests reference the mocked interfaces, run them to confirm the mocks satisfy the expected contracts.

Fix any validation failures before reporting completion.

## What not to mock

- **Existing production code** — only mock vibe artifacts. Do not replace working implementations with mocks.
- **Build configuration and infrastructure** — do not mock `Dockerfile`, CI configs, `package.json` scripts, etc., even if they contain vibe comments. Flag these for the user's attention instead.
- **Test files** — if vibe artifacts appear in test files, implement them as real test logic (assertions, fixtures), not mocks of mocks.

## Final response

Report concisely:

- the mock implementations generated, with their file locations;
- the vibe narratives preserved and where they can be found;
- any shared mock data modules created;
- validation performed and results;
- any vibe artifacts that could not be mocked and why;
- a reminder that `/vibe` can be used to lower the mocks to real implementations.
