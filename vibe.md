---
name: vibe
description: Convert Vibe pseudocode in .vibe files, startvibe/endvibe source blocks, and source comments containing *vibe* into working code. Use when the user invokes /vibe, asks to implement or resolve Vibe code, supplies a .vibe file, or references actual Vibe markers. Do not activate for incidental prose or documentation that merely discusses Vibe.
---

Vibe is a grammarless pseudocode metalanguage for expressing implementation intent inside a real codebase. Its contents may be valid target-language code, malformed approximations, prose, omissions, or any mixture of them. Treat Vibe syntax as evidence of intent, not as a literal implementation contract.

Transform the Vibe artifacts in the requested scope into working target-language code, then remove only the artifacts whose intent has been fully implemented.

## Scope

- Default to the entire repository.
- If the user explicitly names a path or component, limit the run to that scope and name the scope in the final result.
- Search tracked and unignored working-tree source, including untracked non-ignored files.
- Exclude generated output, dependencies, vendor code, caches, submodule internals, and similar derived or third-party content.
- If the declared scope contains no Vibe artifacts, make no changes and report that none were found.

## Recognizing Vibe

There are three artifact types:

1. Any file whose name ends in `.vibe`. One file may describe one target file or an entire component spanning source, tests, migrations, and configuration. The file's existence and location are themselves intent: they signal the code structure the user wants (for example, an intended folder layout, module placement, or component boundary), so honor that structure when producing real files.
2. An inline region in a plausible target-language source file, delimited by `startvibe` and `endvibe`.
3. A single-line comment in a plausible target-language source file containing the literal marker `*vibe*`. This comment draws attention to a specific line or nearby section of code. Treat its remaining text as the complete statement of intent and use its location to identify the code it targets.

Inline delimiters must:

- be standalone comment tokens valid for the containing file's language;
- match case-insensitively;
- allow whitespace around the token; and
- contain no trailing prose.

For example, `# startvibe`, `// STARTVIBE`, and `/* endvibe */` are markers when valid for the containing language. Do not treat marker examples in documentation or data files as artifacts unless the user explicitly includes those files.

Multiple non-overlapping inline blocks in one file are valid. Nested, unmatched, or reversed markers are malformed. Report malformed markers with their locations and do not edit that file until their boundaries are clarified or corrected.

A single-line Vibe comment must be a valid comment for the containing language and may appear on its own line or after code. For example, `// *vibe* handle an expired session here` and `value = load() # *vibe* avoid doing this twice` are artifacts. Do not treat `*vibe*` in strings, documentation, data files, or multi-line comments as an artifact unless the user explicitly includes them.

## Workflow

### 1. Discover the complete inventory

Find every in-scope `.vibe` file, valid inline block, and single-line Vibe comment before editing. Bring the full inventory and enough surrounding repository context into the analysis so related artifacts are considered together.

For each artifact, identify:

- the behavior or structure it requests;
- its likely target language and destination;
- affected contracts, callers, tests, data, configuration, and external effects;
- dependencies on other Vibe artifacts or existing code; and
- ambiguities or contradictions that could materially change the result.

Infer a standalone `.vibe` file's target from, in order:

1. an explicit header or user instruction;
2. its path and filename;
3. nearby modules, imports, symbols, and tests; and
4. build manifests and repository conventions.

Build a lightweight dependency graph and plan to implement foundations before consumers.

### 2. Establish sufficient clarity

Do not begin a target implementation unless you are sufficiently clear on that component's objective.

Use this authority order when evidence conflicts:

1. explicit user instructions;
2. observable tests and contracts;
3. established repository architecture and conventions;
4. literal details in the Vibe artifact.

Infer reversible implementation details when repository evidence is strong. In the event of ambiguity, clarify with the user before editing rather than guessing — especially when uncertainty affects public APIs, persistence or schemas, authorization or security, destructive behavior, external side effects, materially different user-visible outcomes, or another semantic contract.

Treat unresolved contradictions between Vibe artifacts as material conflicts. Present the plausible interpretations together, recommend one, and block the affected dependency component until the user decides.

Batch all currently answerable questions. Independent components whose objectives are sufficiently clear may proceed while unrelated components remain blocked. Do not edit a component that depends on unresolved intent.

### 3. Implement in dependency order

- Treat an entire inline Vibe region as provisional intent, including lines that already look like valid production code.
- Treat a single-line Vibe comment as an instruction about the line or nearby section it identifies. Use the comment text and local code context to determine its target without requiring a larger Vibe block.
- Replace each understood artifact with a complete implementation that follows the repository's existing patterns and types.
- A `.vibe` file may require creating or updating multiple real files.
- Preserve and integrate with existing working-tree changes. Never revert unrelated user work. If safe integration is unclear, block that component and ask.
- Keep inline marker lines, single-line Vibe comments, and each `.vibe` file in place until their corresponding implementation is ready to validate.

### 4. Validate before removing intent

Use the smallest existing syntax, formatting, type-check, build, or test commands that cover each changed component. Do not install or invent tooling merely to clear a Vibe artifact.

If applicable validation fails, fix the implementation and rerun it. If failure exposes unresolved intent or correctness cannot be established, preserve the Vibe artifact and clarify the ambiguity with the user before proceeding.

When no applicable automated validation exists, strong contextual and structural confidence may suffice. Inspect the result for syntactic and structural coherence and disclose the lack of automated validation.

Only after all outputs represented by an artifact are complete and sufficiently supported:

- remove the `startvibe` and `endvibe` marker lines from a successfully implemented inline block; or
- remove the entire successfully implemented single-line Vibe comment while preserving any code on the same line; or
- delete the successfully implemented `.vibe` file.

Never delete unresolved intent.

### 5. Enforce the completion gate

Repeat discovery in the declared scope after cleanup.

The run is fully successful only when no in-scope `.vibe` files, valid inline blocks, single-line Vibe comments, or malformed Vibe markers remain. Otherwise, describe the result as partial and list every residual artifact with its blocker. When the user narrowed the scope, do not claim that the whole repository is Vibe-free.

## Final response

Report concisely:

- the components implemented;
- the `.vibe` files deleted, inline blocks removed, and single-line Vibe comments removed;
- the validation performed, or that no applicable automation existed; and
- any residual artifacts and why they remain.
