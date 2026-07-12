# Generative recursion: designing algorithms

Everything so far is **structural**: the function's recursion follows the data definition, so the case analysis is given and termination is automatic — "input shrinks at every stage. Eventually the function is applied to an atomic piece of data, and the recursion stops" (HtDP §26.2). Part V introduces the other kind. **Generative recursion** — what HtDP calls *algorithms* — rearranges a problem into new sub-problems by insight:

> "An algorithm tends to rearrange a problem into a set of several problems, solve those, and combine their solutions into one overall solution. … its recursion uses newly generated data, not immediate parts of the input data." — Part V intro

Quicksort doesn't recur on `rest(list)`; it invents two new lists. Binary search computes a midpoint. Newton's method generates a better guess. "The key to designing algorithms is the 'generation' step… figuring out a novel way of dividing a problem requires insight" — the *eureka*. Because that insight has no data definition backing it, the design recipe grows extra obligations.

## Keep the two kinds distinct

HtDP explicitly rejects "everything is generative" (§26.3): conflating the two "confuses two kinds of design that require different forms of knowledge… One relies on a systematic data analysis and not much more; the other requires a deep, often mathematical, insight… **One leads programmers to naturally terminating functions; the other requires a termination argument.**"

And the default is structural (§26.4): "Most functions in a program employ structural design; only a few exploit generative recursion." When either could work, "the best approach is to start with a structural version. If the result turns out to be too slow for the task at hand—and only then—it is time to explore the use of generative recursion." (The payoff can be enormous — HtDP's structural gcd needs ~45 million steps where Euclid's generative insight needs 9 — but quicksort also *loses* to structural insertion sort on small lists; measure before committing.)

## The adapted recipe (§26.1)

- **Purpose statement**: "Since the generative step has no connection to the structure of the data definition, **the purpose statement must go beyond what the function is to compute and also explain how the function computes its result.**" Structural functions say *what*; algorithms say *what and how*. Future readers cannot re-derive your eureka from the code.
- **Examples**: illustrate the *process*, not just input→output — show the partition, the halving, the descent.
- **Template** (fixed, not derived from data):

```
fun solve(problem):
  if trivially-solvable(problem):
    return trivial-solution(problem)
  else:
    subs = generate-subproblems(problem)      ; the eureka lives here
    solutions = [solve(s) for s in subs]
    return combine(problem, solutions)        ; may need the original problem
```

- **Definition — the four questions** (§26.1, near-verbatim):
  1. **What is a trivially solvable problem?** ("Trivial" is problem analysis, not data analysis: an empty *or one-element* list is trivially sorted.)
  2. **How are trivial problems solved?**
  3. **How does the algorithm generate new problems that are more easily solvable than the original? Is there one new problem, or several?**
  4. **Is the solution of the given problem the same as the solution of (one of) the new problems? Or must solutions be combined? And if so, is anything needed from the original problem data?**

  When stuck on the combining step, the table method from the structural recipe still applies: tabulate example inputs, the sub-problem solutions, and the expected output until the combinator pattern emerges.

## The termination argument — the mandatory seventh step (§26.2)

> "Generative recursion adds an entirely new aspect to computations: non-termination."

Nothing forces generated sub-problems toward a trivial case, so HtDP adds a seventh recipe step. A termination argument takes one of two forms:

1. **A shrinking argument**: why each recursive call works on a *smaller* problem — name the measure that strictly decreases (the interval halves; each partition excludes the pivot and is shorter than the input).
2. **An honest non-termination note**: "illustrate with an example that the function may not terminate… Ideally it should also describe the class of data for which the function may loop" — and guard accordingly.

The argument goes **into the header comments**, next to signature and purpose:

```
; List<1String>, N -> List<String>
; bundles sub-sequences of s into strings of length n
; termination: (bundle s 0) loops unless s is '()
```

Termination arguments are *brittle* under edit: HtDP's own exercise shows that changing quicksort's partition from `<` to `<=` makes it loop forever, because the argument relied on both partitions excluding the pivot. **When you edit a generative function, re-verify the termination argument** — it is part of the code's correctness, not decoration.

The obligation is older than HtDP — The Little Schemer's Fourth Commandment (Friedman & Felleisen, 1974): "Always change at least one argument while recurring. It must be changed to be closer to termination. The changing argument must be tested in the termination condition."

**This generalizes to every loop whose bound is not the structure of its input**: `while` loops on conditions, retry loops, polling loops, fixpoint iterations, agent loops that run "until done." Each needs a one-line termination argument; if you can't write one, add an explicit bound with defined behavior at the bound. Unbounded improvised loops are among the most damaging code an LLM can emit — a "trim until it fits" loop that can fail to make progress is exactly a missing termination argument.

## Backtracking and search (ch. 29)

For problems shaped like "find a path/assignment/solution among alternatives":

- Represent the search state as data (`; A Path is a List<Node>`), and make failure a value the caller can react to: the result is an itemization — `Maybe<Path>`, i.e. a path or `#false`. "If find-path indeed produces a path, that path is its answer. **Otherwise, find-path/list backtracks**" and tries the next neighbor.
- The generation step over "arbitrarily many alternatives" is a *structural* helper over the list of alternatives, mutually recursive with the generative searcher — each function designed by its own kind's recipe.
- **Termination on cyclic structures fails**: HtDP shows `find-path` re-encountering the identical call on a cyclic graph — "Since the same input triggers the same evaluation for any function, find-path does not terminate for these inputs." The fix is accumulated knowledge of visited states (`08-accumulators.md`).

HtDP's note on data abstraction here: `find-path` never touches the graph representation directly, only through `neighbors` — so the representation can change freely. "When programs grow large, data abstraction becomes a critical tool."

## Correctness beyond termination

- With no data definition to guarantee coverage, examples must cover **the generation logic itself**: duplicates of the pivot; targets absent below/above/between; non-convergent inputs for numeric methods.
- State the **algorithm's precondition** in the header ("assume f is continuous"; "assume the list is sorted") and decide who guarantees it. If checking the precondition would destroy the complexity class (verifying sortedness is O(n) inside an O(log n) search), the precondition becomes an invariant of the input's data definition.
- Prefer known algorithms with known names. If it's Dijkstra, call it Dijkstra, follow the canonical form, and inherit fifty years of debugging.

## LLM directives

- Classify every function you write: structural or generative? Structural → verify the template matches the data definition. Generative → write the four answers, the "how" purpose statement, and the termination argument in the code's comments, not just in your head.
- Audit every `while` loop and recursion for a decreasing measure. No measure → explicit bound + documented behavior at the bound.
- Default structural; go generative only when structural is demonstrably too slow or cannot express the algorithm — and say which of those reasons applies.
