# Accumulators: make context and invariants explicit

An accumulator carries knowledge that is not present in the current subproblem: a consumed prefix, visited nodes, an enclosing scope, or a partial result. HtDP develops the accumulator invariant and derives initialization, maintenance, and use from it in [Chapter 32](https://htdp.org/2024-11-6/Book/part_six.html).

## Recognize the need

Consider additional state when a traversal repeatedly processes earlier results, needs information about where it is, or must remember previous choices. A visited set can prevent cyclic search; a running prefix can avoid repeated summation.

Studying a simpler implementation often reveals the missing knowledge. You can also design an accumulator directly when the need is already clear, rather than constructing a slow or diverging implementation first.

An accumulator does not automatically improve asymptotic complexity. A structural sum and an accumulator sum both traverse each element once. Costs depend on the operations, representation, and language.

## State the invariant

Describe the relationship between the original input, current subproblem, and accumulated value. “Partial result” is usually too vague.

Useful statements include:

- `total` equals the sum of the first `i` original values.
- `reversed_prefix` contains the consumed prefix in reverse order.
- `ancestors` contains the nodes on the current root-to-parent path.
- `seen` contains graph nodes already discovered, so each node is enqueued at most once.

Derive three decisions:

1. **Initialization:** what makes the invariant true before processing starts?
2. **Maintenance:** why does each transition preserve it?
3. **Use:** why does the invariant imply the required result when processing ends or a particular case is reached?

Keep implementation state private unless exposing it is part of the intended API. Several related fields can form a named worker state; no fixed number of accumulators demands a new structure.

## Worked example: prefix totals

Definition: a finite sequence of integers represents successive changes. The result contains one total per input element, with no extra initial zero.

Examples derived before implementation:

```text
prefix_totals([])        = []
prefix_totals([4])       = [4]
prefix_totals([4, -1, 2]) = [4, 3, 5]
prefix_totals([0, 0])    = [0, 0]
```

```python
def prefix_totals(changes: list[int]) -> list[int]:
    """Return the total through each successive change."""
    totals: list[int] = []
    total = 0
    # Before iteration i:
    # len(totals) == i
    # total == sum(changes[:i])
    # totals[j] == sum(changes[:j + 1]) for every j < i
    for change in changes:
        total += change
        totals.append(total)
    return totals
```

Initialization: the empty prefix has sum zero and no output entries. Maintenance: adding the next change yields the next prefix total, and appending preserves all prior totals. At the end, the invariant covers every input position. The finite collection controls termination.

This is the imperative counterpart of an accumulator worker. The local mutations are compatible with a data-to-data public contract when the function does not mutate shared inputs.

## Preserve observable behavior

Recheck invariants when changing updates, traversal order, or the base case. An invariant can involve several state variables together; separate one-line claims about each may miss their relationship.

An accumulator refactor can alter evaluation order. Floating-point addition is not associative, and callbacks or effects can make order observable. Python and many other languages do not eliminate tail calls, so accumulator recursion does not guarantee constant stack usage.

For output construction, check whether prepending, appending, copying, or final reversal changes order and complexity. For graph algorithms, verify that the remembered state is sufficient without suppressing valid alternatives.
