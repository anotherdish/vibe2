---
name: interface-engineering
description: Design and encode only the minimal interfaces, architecture, and executable interface constraints needed in a codebase, without implementing behavior. Use when the user invokes /interface-engineering or asks for interface engineering: API-driven, test-first interface design using language interfaces, schemas, protobufs, or signatures, with unit tests defining the contract. Keep interface constraints separate from later implementation constraints.
---

Interface engineering focuses exclusively on the smallest amount of code and architecture needed to express an intended interface and constrain its behavior. It is inspired by API-driven design and TDD: design the interface from its consumers' needs, encode expectations as unit tests, and stop before implementing the behavior.

The output is a reviewable, executable contract, not a working feature. Tests specify what a future implementation must satisfy; they are not permission to implement it.

## Two kinds of constraints

### Interface constraints

Interface constraints define what consumers may rely on, regardless of how the interface is implemented.

- Encode the structural contract in the appropriate native form: function signatures, language interfaces, traits, protocols, public types, schemas, protobuf service and message definitions, or equivalent repository conventions.
- Encode behavioral expectations as unit tests against that public surface: results, errors, observable state transitions and side effects, input boundaries, and any agreed ordering, ownership, or retry guarantees.
- Keep semantics that types cannot express in durable contract documentation as well as focused assertions. Tests are executable constraints, not a reason to omit useful API documentation.
- Every conforming implementation must satisfy the same interface constraints without changing their assertions. Setup or provider construction may vary.

These tests define the contract rather than a preferred implementation. Do not assert private state, internal helper calls, storage layouts, or algorithms unless that detail is explicitly part of the intended interface.

### Implementation constraints

Implementation constraints validate properties of a particular implementation that are not required by the interface constraints. They can also be expressed as unit tests, but belong to the later implementation phase.

Examples include using a particular internal cache, limiting calls to a private dependency, choosing a storage strategy, or maintaining an internal invariant. An alternative conforming implementation may legitimately fail these tests.

Classify a constraint by its meaning, not its test framework or file extension. A property such as ordering or resource usage is an interface constraint if consumers are explicitly promised it; otherwise it may be an implementation constraint.

Implementation constraints supplement interface constraints. They must not weaken, replace, or silently redefine the public contract. Changing a public expectation is an explicit interface-design decision, not a way to make an implementation pass.

## Distinguish interfaces from implementations

Establish a clear, consistent codebase pattern that makes interface declarations distinguishable from their implementations through names, locations, and dependency direction. A reader should be able to find the contract without inspecting a provider's internals.

- Follow an existing clear convention. If none exists, establish the smallest language-appropriate convention for the boundary being designed and apply it consistently within scope; do not reorganize unrelated code.
- Give contracts consumer-facing names and implementations provider-specific names, such as `Catalog` and `SqlCatalog`. Use separate files or modules when appropriate, such as `catalog/contract` and `catalog/providers/sql`, or the language's native declaration/implementation separation. These are examples, not mandatory folder names or a requirement to introduce a class for every function.
- Keep public request, response, and error types with the contract, not hidden inside a concrete provider. Contracts must not depend on concrete implementations or expose their private types. Implementations depend on and satisfy contracts.
- Have consumers use the contract surface. Keep concrete provider selection in the smallest existing construction or composition boundary, rather than spreading provider-specific imports through consumer logic. Do not add a dependency-injection framework to achieve this.
- Mirror the distinction in tests: identify interface-constraint suites by the contract and later implementation-constraint suites by the provider. Isolate provider construction from shared contract assertions so an implementation can change without rewriting interface expectations.
- Record the chosen convention briefly in an existing relevant documentation location or beside the boundary. Make the distinction structural, not solely a comment saying "interface" or "implementation."

Establish only the contract side and any necessary explicit stubs during this phase. Naming a future implementation location does not require creating empty provider directories or authorizing implementation.

## Scope boundary

- Create only the interfaces, public data shapes, minimal module boundaries, necessary stubs, and focused interface-constraint tests for the agreed scenarios.
- Every new file, type, operation, and dependency seam must be necessary to express or exercise an agreed contract. Reuse existing boundaries before adding new ones.
- Do not introduce internal architecture, algorithms, private helpers, persistence, deployment configuration, frameworks, or speculative extension points.
- Do not require diagrams, design documents, a new test harness, or a new abstraction layer when source declarations and tests are sufficient. Add a visual only when requested or needed to settle a boundary.
- Do not implement even a small amount of feature behavior to turn tests green. Where a concrete body is required, use the repository's explicit not-implemented mechanism, never plausible production defaults or fake success.
- Do not replace working implementations with stubs or connect unfinished providers to live production paths.
- Record implementation suggestions as deferred, not as interface requirements or implementation-constraint tests.
- Stop at the design boundary. Contract approval does not authorize implementation; explicit implementation authorization ends this phase.

## Workflow

### 1. Ground the interface in actual consumers

Read the relevant code, callers, schemas, and nearby tests. Identify the language and test conventions, how contracts are distinguished from implementations, existing contracts, and the smallest consumer scenario that motivates the change. Preserve unrelated work.

Establish who calls the interface, what they need to express, what they can observe, and which component owns the contract. An interface can be a local function or module boundary; do not assume a service or network API is needed.

Ask focused questions when unresolved choices materially affect signatures, outcomes, compatibility, or ownership. Use the host's question tool when available. Do not encode guesses as settled constraints; independent agreed parts may proceed.

### 2. Define the smallest sufficient contract

Work from consumer usage toward declarations. Settle only the applicable details:

- operation names, inputs, outputs, errors, and sync or async behavior;
- public data shapes, required fields, optionality, and valid values;
- success and failure semantics, including observable effects and effects that must not occur;
- meaningful boundary cases and explicitly required guarantees;
- compatibility with existing consumers.

Remove operations and abstractions that no agreed scenario needs. Do not design a general-purpose framework around one use case.

Choose the representation already used at the boundary. For example, use native types for an in-process interface or the existing schema language for a protocol boundary. Do not duplicate the same contract in hand-maintained parallel representations. Use existing generation tooling when generated bindings are required; do not hand-edit generated files.

### 3. Encode declarations and interface constraints together

Write agreed declarations into their intended source locations and behavioral constraints into the existing unit-test layout. Iterate between caller examples, declarations, and tests as each exposes missing or awkward parts of the contract.

Keep new source syntactically valid and type-consistent. Add only the minimum stub or construction seam needed for tests to reach the intended provider. Do not build provider selection infrastructure when an existing fixture or a simple factory is sufficient.

Write real assertions for the important success path, relevant failure paths, and meaningful edge cases. Each test should express an observable guarantee, not restate a signature or test the language itself.

For interface-constraint tests:

- Invoke the actual intended public API. When it is still a stub, behavioral failures are expected.
- Use existing fixtures, fakes, or spies only for dependencies and controlled inputs. Do not substitute a fake implementation of the interface under test and claim its passing tests validate the future provider.
- Keep setup separate from assertions so the same constraints can exercise a future or alternative provider.
- Assert dependency interactions only when those interactions are themselves an agreed observable contract, not merely an imagined implementation plan.
- Do not add empty passing tests, blanket skips, expected-failure markers, or weakened assertions to conceal missing behavior.
- Keep interface constraints distinguishable from implementation-specific tests using the repository's existing organization or a minimal descriptive naming convention. Do not introduce a taxonomy framework.

Use durable comments for contract semantics. If the repository uses Vibe markers for pending bodies, keep them limited to unimplemented intent and do not automatically invoke an implementation skill. The presence of markers is not authorization to implement.

For an existing interface, preserve working behavior and callers. Add compatible declarations and constraint tests where possible. If the proposed shape cannot coexist safely, resolve the migration boundary with the user before changing signatures; do not turn an interface-design task into a production migration.

### 4. Review the constraints, not an imagined implementation

Walk the consumer scenarios through the declarations and tests. Check that the caller can express the request and distinguish success from failure without relying on undocumented behavior. Confirm that contract declarations and provider code are distinguishable, dependencies point toward the contract, and interface-constraint assertions do not import provider internals.

For every assertion, ask: could a different implementation satisfy the agreed interface and still fail this assertion? If yes, remove the implementation assumption or explicitly resolve whether the property belongs in the public contract.

For example, a lookup contract might require a missing key to produce a documented error and a stored key to return its value. Requiring a particular hash table or checking that a private cache was hit would instead constrain an implementation, unless those details were explicitly exposed as part of the contract.

Ensure each important behavioral promise has an executable assertion or a clearly stated validation gap. Do not claim unit tests establish properties they cannot demonstrate, such as real distributed atomicity or production latency. Record necessary later integration, concurrency, or performance validation without building that infrastructure in this phase.

### 5. Validate without implementing

Use the smallest applicable existing formatting, schema, syntax, type, test-collection, and targeted unit-test checks. Do not install new tooling solely for a design draft.

Distinguish expected behavioral failures at deliberate stubs from syntax, import, collection, type, or unrelated failures. Fix design-induced structural errors without implementing behavior. If a check is blocked, report the specific missing prerequisite rather than presenting the contract as validated.

Do not make new behavior pass by implementing it, disabling checks, or weakening the contract. Existing behavior must remain intact. Green tests against an existing provider are acceptable; manufacturing a red test is not a goal.

### 6. Stop with an implementation-ready contract

Leave the minimal declarations, stubs, and interface-constraint tests in the codebase. State the public surface established, the expectations encoded, and any unresolved decisions or expected failing or blocked checks. Be explicit that feature implementation remains absent.

Do not begin implementation or add implementation constraints in this skill. After explicit authorization for a subsequent implementation phase, preserve the agreed interface constraints and add implementation-specific tests separately as needed. A conforming implementation must satisfy both sets; implementation-specific checks do not substitute for the contract suite.
