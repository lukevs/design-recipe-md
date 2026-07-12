# Accumulators: recovering lost context

A recursive function is **context-independent**: it computes the same answer wherever it is called — "a function does not 'know' whether it is called on a complete list or on a piece of that list" (HtDP Part VI intro). That independence is what makes structural design easy, and it has a cost:

> "For structurally recursive programs, this loss of knowledge means that they may have to traverse data more than once, inducing a performance cost. For functions that employ generative recursion, the loss means that the function may not be able to compute the result at all."

> "Since we wish to retain the principle that (f a) returns the same result no matter how often or where it is evaluated, the only solution is to add an argument that represents the context of the function call. **We call this additional argument an accumulator.**"

HtDP's two motivating failures: converting relative distances to absolute ones structurally re-traverses every natural-recursion result — O(n²) where a human "would tally up the total distance" in O(n); and `path-exists?` on a cyclic graph re-encounters the identical call and diverges, because it can't remember where it has been.

## Recognizing the need (§32.1)

The recipe's precondition: **"it is critical that we first built a complete function based on a conventional design recipe. Then we study the function."** Accumulators are a *transformation of a working design*, not a starting point. The signals:

- Structural: "**If a structurally recursive function traverses the result of its natural recursion with an auxiliary, recursive function, consider the use of an accumulator parameter.**" (The reverse-a-list shape: `invert` calling recursive `add-as-last` on every recursion's result.)
- Generative: ask "whether the algorithm can fail to produce a result for inputs for which we expect a result. If so, adding a parameter that accumulates knowledge may help." (The visited-set shape.)

## The accumulator recipe (§32.2)

1. **Template**: wrap the worker in a local/inner function that takes the extra parameter `a`; the outer function keeps its clean signature and fixes the initial value, so callers never see the plumbing:

```
fun total(items):                       ; ListOfNumber -> Number
  fun total-acc(remaining, so-far):
    ; ACCUMULATOR so-far: the sum of the numbers that
    ; `remaining` lacks from the original `items`
    if remaining is empty: return so-far
    else: return total-acc(remaining.rest, so-far + remaining.first)
  return total-acc(items, 0)
```

2. **Sketch a hand-evaluation** to understand what the accumulator must track.
3. **Determine the kind of data** the accumulator holds (it gets a data definition too).
4. **Write the accumulator statement — the invariant.** "Write down a statement that explains the accumulator as a relationship between the argument `d` of the auxiliary function and the original argument `d0`." HtDP: "The relationship remains constant, also called invariant… an accumulator statement is often called an invariant." The book's own examples of the genre: *"`a` is the sum of the numbers that `alon` lacks from `alon0`"*; *"`a` is the list of all those items on `alox0` that precede `alox`, in reverse order"*; *"`seen` is the list of nodes already inspected in the current chain."*
5. **Derive everything else from the invariant**: the initial value (what makes it true when nothing is consumed?), the update rule (what keeps it true on each step?), and the harvest (what does it yield where the knowledge is needed — usually the base case; for generative recursion, often a **new trivial case**: `path-exists` gains the clause "if origin is in `seen`, the answer is false", which is precisely what restores termination).

> "The key is the precise description of the role of the accumulator. … Articulating the accumulator statement is difficult, but, **without formulating a good invariant, it is impossible to understand an accumulator-style function.**" — §32.2–32.3

HtDP ch. 34 sums it up as "two and a half design lessons": recognize the need; **formulate the accumulator statement** ("what knowledge the accumulator gathers as what kind of data… in most cases, the difference between the original argument and the current one"); then mechanically deduce initial value, maintenance, and exploitation.

## Every imperative loop is an accumulator function

HtDP ch. 34, on imperative languages: programmers "encounter accumulators… mostly via assignment statements in primitive looping constructs… Designing such imperative accumulator programs proceeds just like the design of accumulator functions here." A `for` loop with mutable locals *is* the accumulator template:

```
result = 0                       ; establishes: result = sum of items[0..i)
for item in items:
    result += item               ; preserves the invariant for i+1
return result                    ; harvests: i = n, so result = total
```

For any nontrivial loop, be able to state what each mutated variable means *at the top of each iteration*, relative to how much input has been consumed. If you can't, the loop is not designed — it's improvised. (This is classic loop-invariant reasoning, taught as a design step instead of a verification afterthought.)

## Common accumulator patterns

- **Running aggregate**: sum, max-so-far, count — turns O(n²) re-traversal into one pass, and enables tail-recursive shapes.
- **Reversed prefix / building output**: collecting results as you go (remember the final reverse if order matters — a classic invariant-harvesting bug). Note HtDP's subtlety: accumulator-style `sum` adds left-to-right where the structural version adds right-to-left — indistinguishable for exact numbers, *observable* for floating point. An accumulator transformation can change behavior at representation edges; keep the original's tests.
- **Visited set**: what turns diverging graph search into terminating search.
- **Environment/context**: interpreters and tree walks carrying enclosing scope (bindings, current path, indentation) — "everything between the root and here that matters."
- **Position/index**: line numbers, offsets during parsing.

## LLM directives

- When a recursion or loop needs "what came before", name that knowledge, make it a parameter (or fold state), and **write the invariant as a comment**. The accumulator statement is one of the few comments that is always worth its cost.
- When editing an existing loop, reconstruct each mutated variable's invariant *before* changing anything; most loop-edit bugs preserve the syntax while breaking the invariant.
- Keep accumulator plumbing out of public signatures — wrap it. An API demanding `f(x, [], 0, None)` is leaking its implementation.
- Three or more threaded accumulators → bundle them into a named state structure with its own data definition and invariants; it has become compound data.
