# Templates: derive an inventory from the data

A structural template records what the input definition makes available: alternatives, selectors, and recursive substructures. HtDP's derivation appears in [Chapter 9](https://htdp.org/2024-11-6/Book/part_two.html); simultaneous inputs are treated in [Chapter 23](https://htdp.org/2024-11-6/Book/part_four.html).

## Derive the inventory

1. Identify the alternatives the function's valid input can take.
2. Find predicates or patterns that distinguish them.
3. List the fields available in each structured alternative.
4. Identify recursive substructures and the natural recursive calls they suggest.
5. Identify references to other data definitions and the processing contracts those may require.

Global constants and additional arguments may also be available. The inventory is deliberately broader than the final implementation: a function need not use every field, traverse every recursive child, or explicitly branch where an operation works uniformly.

## Worked shapes

Enumeration:

```text
describe : TrafficLight -> String
describe(light):
    match light:
        Red:    ...
        Yellow: ...
        Green:  ...
```

Recursive list:

```text
process : List<X> -> Y
process(items):
    match items:
        Empty:             ...
        Cons(first, rest): ... first ... process(rest) ...
```

Mutually referential tree and list:

```text
Entry = File(name, bytes) | Directory(name, List<Entry>)

process_entry(entry):
    match entry:
        File(name, bytes):        ... name ... bytes ...
        Directory(name, entries): ... name ... process_entries(entries) ...

process_entries(entries):
    match entries:
        Empty:            ...
        Cons(first, rest): ... process_entry(first) ... process_entries(rest) ...
```

These are incomplete pseudocode templates, not runnable implementations. In final code, a fold or collection traversal can package the list helper. A short-circuiting search may visit only one child.

## Interpret the recursive result

Use the recursive function's purpose to determine what its result means for the substructure. If `count_entries` counts file occurrences below an entry, its recursive result already supplies that count for a child. The current case must combine it correctly.

This assumption rests on valid recursive inputs and well-founded descent. Tracing a concrete example can check the combination; it does not replace the argument for all inputs.

## Two complex inputs

Decide how the inputs relate:

- **One drives the traversal.** Appending lists can recurse over the first and return the second at the base. The second argument is treated as an available value rather than independently traversed.
- **They move in lockstep.** Pairwise processing consumes corresponding elements. State whether unequal lengths are invalid, truncated, padded, or represented as failure; respect the established API.
- **Their cases interact independently.** Build a case table, such as empty/non-empty against empty/non-empty for merging lists. Decide the behavior of each reachable combination before simplifying the code.

Use the table when it reveals otherwise easy-to-miss interactions. More than two inputs require analysis of their relationships, not automatically a full Cartesian enumeration of all values.

## Translate into the host language

Matches, loops, comprehensions, visitors, and standard collection operations can embody the same structure. Check the actual language's guarantees: Python `match` does not generally provide compile-time exhaustiveness; TypeScript narrowing alone does not reject every omitted variant.

A grouped branch is valid when its alternatives share specified behavior. A catch-all is suspect when it hides a newly added variant that needs distinct behavior. Invalid inputs need explicit handling only where the contract includes them.

A nested field access does not automatically violate the recipe. Delegate when the nested value requires its own traversal, domain reasoning, reusable operation, or private interface. Adding a helper solely for each field access creates indirection without strengthening the design.

## Check the implementation

Ask whether each relevant case is accounted for explicitly, by a proven precondition, or by a reused abstraction. Check recursive calls for descent and contract preservation.

Review costs as well as shape. Repeated slicing or length calculation may turn a linear traversal into quadratic work. Language recursion limits and absent tail-call optimization can make an iterative implementation preferable even when the reasoning is structural.
