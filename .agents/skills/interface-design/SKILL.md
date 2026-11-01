---
name: interface-design
description: Lay down the interfaces, APIs, and module architecture of a system as real code, with bodies left as Vibe annotations stating what each part of the API must guarantee. Use when the user wants to design or stub out interfaces, APIs, types, or module boundaries before implementation, or to lower a .vibe design file into a compiling skeleton. For capturing design intent in prose, use vibe-design; for implementing Vibe artifacts, use vibe.
---

This skill sits between `vibe-design` and `vibe`. A `.vibe` design file says what a system is for and what was decided. This skill turns that into code: the files, modules, types, and signatures that make up the system's interfaces, with Vibe annotations recording what each piece promises. The `vibe` skill later fills in the bodies.

Write enough code to pin down the architecture and the API surface, and stop there. The output should compile and show how the system fits together. It should not do anything yet.

## Inputs

- Any `.vibe` design files in scope, plus the conversation that produced them.
- The existing codebase: its language, layout, conventions, and the interfaces already there.
- If no design exists and the shape of the API is unclear, settle it with the user before writing code. A wrong interface costs more to undo than a wrong implementation.

## What to write as real code

- The file and directory layout. Where a module lives is part of the design.
- Public types: domain models, request and response shapes, error types, identifiers.
- Interfaces, traits, protocols, or abstract classes at the seams between components.
- Signatures for every public function, method, endpoint, command, or message handler.
- The wiring that shows how components depend on each other: constructors, dependency injection points, registration, and route tables. Write just enough that a reader can trace a call from the entry point to the boundary.

Use the target language's real type system. Spell out parameter and return types, error and result types, and nullability or optionality. Leave a type loose only when it is still undecided, and mark it with a `vibe:` comment when you do.

## What to leave out

- Function bodies beyond a stub. Use the idiomatic not-implemented form for the language, such as `raise NotImplementedError`, `todo!()`, `throw new Error("not implemented")`, or `panic("not implemented")`.
- Private helpers, internal data structures, and algorithms, unless the user has already decided on them.
- Tests, migrations, and config values, unless they are part of the interface (an exposed config schema, for example).
- Speculative extension points. Every interface should have a reason to exist that came out of the discussion. Do not add an abstraction layer just because one could exist.

## Vibe annotations

The annotations carry the expectations the signatures cannot express. Write them as Vibe artifacts so the `vibe` skill picks them up:

- A single-line `// vibe:` comment for one expectation on a line or a small section.
- A `startvibe` / `endvibe` region inside a body for anything longer. Keep the signature outside the region so it stays fixed. Everything inside the region is provisional.

```python
def reserve(self, order: OrderId, sku: Sku, qty: int) -> Reservation:
    # startvibe
    # Raises OutOfStock if qty exceeds available; never reserves partially.
    # Idempotent per (order, sku): a repeat call returns the existing reservation.
    # Safe to call concurrently for the same sku.
    raise NotImplementedError
    # endvibe
```

Annotate the contract, not the steps:

- preconditions and input constraints the caller must meet
- what the caller can rely on afterward: return guarantees, state changes, side effects
- failure modes and which error type each one surfaces as
- idempotency, ordering, concurrency, and transaction boundaries
- ownership and lifetime of resources, and who closes or frees them
- performance or limit expectations, if the user raised them

Write implementation steps only when the user has already settled them. The test from `vibe-design` still applies: if an implementer would get it right without being told, leave it out. A signature that already explains itself gets no annotation.

Write a contract that should stay as permanent documentation, such as the semantics of an interface method that every implementer must follow, as an ordinary doc comment rather than a Vibe annotation. Vibe annotations are consumed and removed when implemented. Doc comments stay.

## Workflow

1. Read the design files, the conversation, and the code around the target area. Note which interfaces already exist and should be reused or extended rather than duplicated.
2. Settle any open question that would change an interface's shape before writing code: what goes on which side of a boundary, sync vs. async, how errors are represented, who owns which data.
3. Write the types and interfaces first, then the modules that implement or consume them, then the wiring.
4. Validate with the smallest existing type-check, build, or lint command that covers the changed files. The skeleton must compile with every body stubbed. If nothing can validate it, review it carefully for consistent types and imports across files.
5. Leave `.vibe` design files in place. Once the code captures a decision, do not repeat that decision in the annotations.

## Final response

Report concisely:

- the files and modules created or changed, and the architecture they set up
- the main interfaces and API surface
- any decisions you made that the user did not state explicitly
- the validation you ran
- open questions left as `vibe:` annotations
