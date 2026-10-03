# Generative recursion: generated subproblems and progress

Structural recursion descends into components given by a finite inductive data definition. Generative recursion creates subproblems: partitioned lists, narrowed intervals, improved guesses, or candidate search states. HtDP adds design and termination obligations in [Chapter 26](https://htdp.org/2024-11-6/Book/part_five.html).

Both kinds can occur in one program. A generative searcher may use a structural helper to try a list of candidates.

## The adapted recipe

Keep the data definition, signature, behavior examples, and tests. Add an explanation of the algorithm: HtDP includes this in the purpose step because the generative insight cannot be recovered from a structural template.

Answer four questions:

1. Which problems are directly solvable?
2. What are their solutions?
3. How are the other problems transformed into new subproblems?
4. How do the subproblem solutions produce the original answer, and what original information is needed?

A general template is:

```text
solve(problem):
    if directly_solvable(problem):
        return direct_solution(problem)
    subproblems = generate(problem)
    solutions = solve each subproblem
    return combine(problem, solutions)
```

Some algorithms create one subproblem or stop after the first successful alternative. The template expresses the questions, not a mandatory allocation or traversal strategy.

## Termination requires a well-founded argument

Name a measure that strictly decreases on every recursive call and cannot decrease indefinitely before a base case is reached. Natural-number measures are common: remaining interval width, number of unprocessed items, or available search choices.

“Gets smaller” alone is insufficient. A positive real can be halved forever without reaching zero. Numeric convergence needs appropriate assumptions and a stopping criterion; an implementation may also need a limit, stagnation detection, and a specified failure outcome.

Finite structural data provides a ready measure when every recursive call follows a proper substructure and all other work terminates. Cyclic graphs or recursion on unchanged data do not inherit this argument.

For quicksort, exclude the selected pivot occurrence before partitioning the remainder, so both recursive partitions are shorter. Duplicates must still be retained in the final output. A partition condition is not by itself a termination guarantee; check what data actually reaches each call.

## Worked example: lower bound

Definition: `values` is a finite random-access sequence of integers sorted in nondecreasing order. `lower_bound` returns the first index whose value is at least `target`, or the sequence length if none is.

Predicted examples:

```text
lower_bound([], 3)          = 0
lower_bound([1, 3, 3, 7], 0) = 0
lower_bound([1, 3, 3, 7], 3) = 1
lower_bound([1, 3, 3, 7], 4) = 3
lower_bound([1, 3, 3, 7], 9) = 4
```

The generated subproblem is an interval of candidate positions:

```python
def lower_bound(values: list[int], target: int) -> int:
    """Return the first index with value >= target, or len(values).

    Precondition: values is sorted in nondecreasing order.
    """
    lo, hi = 0, len(values)
    # Invariant: 0 <= lo <= hi <= len(values).
    # Every value in values[:lo] is < target; in values[hi:] is >= target.
    # The lower-bound position lies in [lo, hi].
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if values[mid] < target:
            lo = mid + 1
        else:
            hi = mid
    return lo
```

Direct case: the candidate position is fixed when `lo == hi`. Generation: comparison at `mid` preserves the invariant while discarding a region. Combination: the narrowed problem has the same answer. Termination: the nonnegative integer `hi - lo` strictly decreases.

The comments use slice notation mathematically; the implementation does not allocate slices. Sortedness is a caller precondition here. Validate it at the boundary that establishes the sorted representation rather than adding an O(n) check inside each O(log n) search.

## Search and ongoing processes

For finite graph reachability, track visited nodes and explain how each expansion reduces unexplored state. A current-path set and a global visited set express different knowledge; choose according to the search problem. Cycles, self-loops, disconnected targets, and repeated alternatives belong in examples.

Timeouts and retry budgets control resource use; they do not prove that an algorithm finds a correct solution. Define the outcome when the budget is exhausted.

A service or event loop may intentionally continue indefinitely. Specify its lifecycle, cancellation, waiting behavior, and recovery policy. Do not add an arbitrary finite cap to a requested monitor or long-running service merely to satisfy a termination checklist.

Prefer a simple correct design and known algorithms suited to the requirements. Consider complexity before implementation when input size makes it material; there is no requirement to build a predictably unsuitable structural version first.
