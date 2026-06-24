---
name: rustacean
description: Apply strict Rust engineering judgment when writing, reviewing, or refactoring Rust code. Use for Rust tasks that need correctness-first design, model completeness, mathematical or formal-logic abstractions, group-theoretic cryptography models, TLA-style distributed-system models, monadic state modeling, macro use to remove repetition, functional abstractions over OOP habits, explicit side-effect boundaries, rich tests, complete documentation with deny(missing_docs), bounded size, explicit error types, careful Clone or Copy semantics, and production code without panic paths.
---

# Rustacean

## Operating Rule

Use this skill as a set of assertions. Treat each assertion as satisfied, violated, or explicitly deferred by a user constraint. Prefer exact code facts over taste claims. Name the type, function, module, invariant, test, or side-effect boundary that makes the assertion true.

When reporting about Rust code, write in short declarative statements:

- The invariant is in the type.
- The boundary is named.
- The repetition has one source.
- The error is algebraic.
- The panic path remains.
- The model is complete.
- The clone is unjustified.

## Assertions

1. Choose the right thing, not the familiar thing. Correct design is neither needless complexity nor simple triviality. Preserve the real states of the problem. Remove only distinctions that are not semantically used.

2. Repetition is evidence. When the same structure appears twice, inspect it. When it appears three times, abstract it unless the third case proves a real difference. Prefer generic functions, traits, `macro_rules!`, or procedural macros when they remove semantic duplication. Use macros to make repetition impossible, not to hide ordinary control flow.

3. Functions compose before objects organize. Prefer data, pure functions, and small traits over OOP-shaped object graphs. Put effects at named boundaries. Core modules receive values and return values. IO, time, randomness, networking, process state, and filesystem access stay in adapter modules. For homomorphic data transformations, think like Haskell: map structure to structure, and keep effects outside the mapping.

4. Invariants are propositions. Express invariants in types first, constructors second, runtime checks third, tests fourth, and comments last. Add formal-logic comments where reasoning crosses function or module boundaries. Do not decorate obvious code with symbolic noise.

5. Quality must be witnessed. `cargo fmt` and `cargo clippy` are only the floor. Add unit tests for pure functions, table tests for boundary cases, property tests for laws, integration tests for side-effect boundaries, and regression tests for fixed bugs. A test name should state the proposition it witnesses.

6. Documentation is part of the public type. Document every public item. Put `#![deny(missing_docs)]` in crate roots such as `lib.rs` and `main.rs`. If generated or foreign code prevents this, isolate that code and document the reason at the smallest possible scope.

7. Size is a signal. Keep each source file under 1000 lines. Keep each function under 100 lines. Split by domain proposition, not by vague `utils` buckets. A large file or function is acceptable only for generated code or externally imposed format, and the exception must be explicit.

8. Errors are algebraic facts. Define explicit error enums or structs with `thiserror` or manual `Display` and `Error` implementations. Each recoverable failure has a named variant. Do not use `anyhow`, `eyre`, `Box<dyn Error>`, stringly typed errors, or context-only error chains as a substitute for domain error types.

9. Panic is a failed proof. Do not introduce `unwrap`, `expect`, `panic!`, `todo!`, `unimplemented!`, `unreachable!`, unchecked indexing, unchecked numeric casts, or other production paths that can panic. Use typed validation, checked APIs, exhaustive matching, and fallible returns. Test assertion macros are allowed in test code because they witness propositions rather than implement runtime behavior.

10. The model comes before the implementation. A good implementation has a complete model, or names the exact incompleteness it accepts. If a behavior can be expressed mathematically or in formal logic, prefer that expression before code shape. The implementation should be a witness of the model, not a substitute for it.

11. Duplication of ownership is a semantic claim. Use `Clone` cautiously. A `.clone()` that merely appeases the borrow checker is a design smell. Prefer borrowing, lifetime repair, smaller ownership scopes, or explicit state transitions. Prefer `Copy` over `Clone` only for small, immutable, identity-free value types where duplication is semantically invisible.

## Model Completeness

Before choosing modules and functions, name the mathematical object or formal model that makes the code coherent.

For cryptography:

- Prefer group-theoretic, ring-theoretic, or field-theoretic abstractions before byte-level plumbing.
- Name the carrier set, operation, identity, inverse, scalar field, encoding, decoding, and subgroup or domain-separation checks when they matter.
- Keep secret material inside types that enforce constant-time comparison, controlled serialization, and zeroization when required.
- Treat unchecked encoding, invalid curve points, missing subgroup checks, and ad hoc byte concatenation as model failures.

For distributed systems:

- Prefer TLA-style abstraction before implementation. Name state variables, initial states, actions, next-state relation, safety invariants, liveness expectations, and fairness assumptions.
- Separate protocol state from transport behavior, retries, clocks, and persistence.
- Model message loss, duplication, reordering, timeout, restart, and partial failure explicitly.
- Treat a concurrent implementation without a state-transition model as incomplete until the safety property is named and tested.

For stateful logic:

- Prefer monadic state modeling. Make the transition shape explicit: `State -> Input -> Result<(State, Output), Error>` or an equivalent typed composition.
- Use `Option`, `Result`, iterator adapters, futures, parser combinators, and state-passing functions as compositional effects.
- Keep hidden mutable state out of pure transitions. A state transition should be replayable from its input state and event.
- Treat global mutation, ambient context, and implicit caches as effects that must be named at the boundary.

## Formal Logic Comments

Use formal comments only when they clarify a proof obligation that Rust's type system does not fully express.

```rust
// Pre: start <= end <= input.len() and both bounds are UTF-8 character boundaries.
// Post: result.as_str() == &input[start..end].
fn slice_checked(input: NonEmptyUtf8, start: ByteIndex, end: ByteIndex) -> Result<Slice, SliceError> {
    Slice::from_checked_bounds(input, start, end)
}

// Invariant: forall id in pending, !completed.contains(id).
// Preservation: inserting Completed(id) removes id from pending first.
struct WorkSet {
    pending: BTreeSet<JobId>,
    completed: BTreeSet<JobId>,
}

// Law: forall x, decode(encode(x)) == Ok(x).
#[test]
fn encode_decode_round_trip_preserves_value() {
    for value in codec_fixtures() {
        assert_eq!(decode(&encode(&value)), Ok(value));
    }
}
```

Prefer these forms:

- `Pre`: the condition a caller must satisfy.
- `Post`: the condition the function guarantees.
- `Invariant`: the proposition preserved by all public operations.
- `Law`: an algebraic relation that tests must witness.
- `Preservation`: why a state transition keeps the invariant true.

## Implementation Discipline

Start from the domain.

- Introduce newtypes for validated concepts instead of passing raw `String`, `usize`, or `u64` values across the domain.
- Parse and validate at the boundary. Let the core operate on already-validated types.
- Make illegal states unrepresentable when the type system can carry the distinction.
- Use enums for finite state machines. Use exhaustive `match`, not sentinel values.
- Prefer iterator pipelines and total helper functions for pure transformations.
- Use traits when there are multiple implementations, a boundary needs injection, or a law is shared. Do not create traits merely to imitate classes.
- Use `macro_rules!` for local syntactic repetition. Use procedural macros only when the generated structure is large, law-bound, and otherwise error-prone.
- Keep `async`, threads, locks, channels, global state, and process exits at explicit outer layers.
- Avoid `Arc<Mutex<_>>` until shared mutation is part of the problem statement, not an accident of implementation.
- Avoid `.clone()` as a borrow-checker escape hatch. First try passing references, narrowing lifetimes, moving ownership at the correct boundary, or extracting a smaller owned value.
- Derive `Copy` only for scalar-like, immutable, identity-free types where implicit duplication cannot hide cost, authority, resource ownership, or state transition.
- Do not derive `Clone` or `Copy` on handles, guards, tokens, capability types, secret material, open resources, or state machines unless the duplication law is documented and tested.
- Return typed errors. Convert them to user-facing messages only at the application boundary.

## Review Protocol

When reviewing or refactoring Rust code under this skill, check these facts before claiming compliance:

```bash
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets --all-features
```

Search for forbidden or suspect constructs:

```bash
rg -n '\b(anyhow|eyre)\b|Box<dyn Error>|unwrap\(|expect\(|panic!|todo!|unimplemented!|unreachable!|\[[^]]+\]'
```

Search for ownership duplication that needs justification:

```bash
rg -n '\.clone\(|clone_from\(|derive\([^)]*(Clone|Copy)'
```

Inspect the results instead of applying a blind rule. Indexing inside tests may be harmless. Indexing in production code needs a proof, a checked access path, or a type-level bound.

Check structure:

- Crate roots deny missing docs.
- Public items have contract-focused docs.
- Error types are explicit and local to the domain they describe.
- Side effects are confined to named modules.
- The model is complete for cryptography, distributed systems, and stateful logic, or its incompleteness is explicit.
- `Clone` and `Copy` implementations have documented semantic laws.
- Tests cover laws, boundaries, and known regressions.
- No source file exceeds 1000 lines.
- No function exceeds 100 lines.

## Response Pattern

When applying the skill, report findings as assertions:

```text
The parser boundary is named in parse_config.
The domain state is algebraic in ConfigState.
The IO effect leaks into normalize_rules.
The retry error is stringly typed.
The panic path remains in load_cache.
The protocol has no TLA-style state relation.
The clone in apply_event hides ownership of state.
```

If a user asks for implementation, fix the violation directly. If a user asks only for review, stay read-only and list violations with file and line references.
