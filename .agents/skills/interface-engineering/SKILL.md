---
name: interface-engineering
description: Design and encode only the minimal interfaces, architecture, and executable interface constraints needed in a codebase, without implementing behavior. Use when the user invokes /interface-engineering or asks for interface engineering, which is an API-driven, test-first interface design using language interfaces, schemas, protobufs, or signatures, with unit tests defining the contract. Keep interface constraints separate from later implementation constraints.
---

Interface engineering focuses exclusively on the smallest amount of code and architecture needed to express an intended interface and constrain its behavior. It is inspired by API-driven design and TDD: design the interface from its consumers' needs, encode expectations as unit tests, and stop before implementing the behavior.

The output is a reviewable, executable contract, not a working feature. Tests specify what a future implementation must satisfy; they are not permission to implement it.

## Two kinds of constraints

### Interface constraints

Interface constraints define what consumers may rely on, regardless of how the interface is implemented.

- Encode the structural contract in the appropriate native form: function signatures, language interfaces, traits, protocols, public types, schemas, protobuf service and message definitions, or equivalent repository conventions.
- Keep semantics that types cannot express in durable contract documentation as well as focused assertions. Tests are 

These tests define the contract rather than a preferred implementation. Do not assert private state, internal helper calls, storage layouts, or algorithms unless that detail is explicitly part of the intended interface.

### Implementation constraints

Implementation constraints validate properties of a particular implementation that are not required by the interface constraints. 

Examples include using a particular internal cache, limiting calls to a private dependency, choosing a storage strategy, or maintaining an internal invariant. An alternative conforming implementation may legitimately fail these tests.

Implementation constraints supplement interface constraints. They must not weaken, replace, or silently redefine the public contract. Changing a public expectation is an explicit interface-design decision, not a way to make an implementation pass.

## Distinguish interfaces from implementations

Establish a clear, consistent codebase pattern that makes interface declarations distinguishable from their implementations through names, locations, and dependency direction. A reader should be able to find the contract without inspecting a provider's internals. Follow an existing clear convention. If none exists, work with the user to establish one
