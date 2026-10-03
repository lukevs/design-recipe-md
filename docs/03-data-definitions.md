# Data definitions: representation and interpretation

A data definition names a class of values, identifies its members, and explains their meaning in the problem domain. A machine type usually describes only part of this contract. HtDP develops these ideas in [§§3.1, 5.7, and Chapter 8](https://htdp.org/2024-11-6/Book/index.html).

## The main forms

| Form | Example definition | Design consequence |
|---|---|---|
| Atomic | `Distance = nonnegative finite number`, interpreted as meters | State units, encoding, and constraints |
| Interval | `GradePercent = number in [0, 100]` | Make endpoint inclusion explicit |
| Enumeration | `TrafficLight = Red \| Yellow \| Green` | Account for each permitted value |
| Itemization | `SearchResult = NotFound \| Found(Index)` | Distinguish alternatives reliably |
| Structure | `Coordinate = {x: Number, y: Number}` | Interpret fields and their relationship |
| Recursive | `List<X> = Empty \| Cons(X, List<X>)` | Identify base cases and finite recursive construction |
| Mutually referential | `Entry = File \| Directory(List<Entry>)` | Follow references between definitions and traversals |

These are reasoning forms, not prescriptions for a particular language. A list may be implemented as a linked list, vector, iterator, or other collection with different costs and guarantees.

## Constraints are part of the definition

Record information that changes behavior: units, ranges, order, uniqueness, optionality, lifetime, and relationships between fields. For numeric data, decide whether fractional, infinite, and NaN values are permitted when that distinction matters.

Distinguish the full representation from the function's domain. A function may accept only a non-empty collection, but a narrower annotation does not remove empty values from real callers. Decide who establishes the precondition or how absence is handled.

Examples:

```text
Temperature = finite real number
interpretation: Celsius degrees

NonEmptyTemperatures = sequence of Temperature with at least one element

average : NonEmptyTemperatures -> Temperature
maybe_average : sequence of Temperature -> Temperature | None
```

Both APIs can be sound. Choose according to the caller's needs and existing conventions.

## Alternatives, sentinels, and optional fields

Prefer tagged alternatives when they prevent ambiguity or invalid combinations:

```text
Job = Pending
    | Running(started_at)
    | Completed(result, finished_at)
    | Failed(reason, finished_at)
```

A record with several booleans may admit contradictory states. A record with optional fields may require a cross-field invariant. Refactoring into variants is useful when it improves the model, but a well-documented existing record can also be appropriate.

A sentinel is not inherently an invalid encoding. HtDP itself uses alternatives such as a value or `#false`. The requirement is that absence or failure be an explicit, distinguishable member of the result definition. `None` can express “no user”; `-1` can express “not found” when valid indices are nonnegative. Preserve established library conventions, and avoid collisions with successful values.

## Recursive data and graphs

For finite inductive data, there must be a route to construct a finite value without continuing recursion forever. A mutual-definition nest may reach its base through another definition; each individual definition need not have a separate local base clause.

Construct examples starting at the bases and applying recursive constructors several times. Examples check that the definition is usable, but do not by themselves prove all constraints are consistent.

A graph with references or cycles is not a finite tree merely because nodes contain child fields. State whether cycles and shared nodes are allowed and whether traversal counts occurrences or distinct identities. This choice affects termination and visited-state handling.

## Choose and evolve a representation

Use the simplest representation that supports the operations and preserves the distinctions the task needs. Hide representation behind an interface when clients should remain independent of it; ordinary public records need not acquire getter wrappers.

Illustrate a new definition with representative values and interpretations. For a proposed change, inspect producers, consumers, serialization, and compatibility before altering the representation.

Existing schemas and types are the starting point. Reconstruct missing constraints from actual callers and requirements; do not assume a generic object accepts every imaginable field combination.
