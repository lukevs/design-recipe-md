# Abstraction: shared behavior with a clear contract

HtDP teaches both abstraction from concrete examples and abstraction from templates. It also teaches using existing abstractions by matching their signatures and purposes. See [§§15.1–15.4 and 16.6](https://htdp.org/2024-11-6/Book/part_three.html).

## Discover an abstraction from examples

Two working, tested definitions provide evidence about what varies:

```text
names_of(users)  = transform each user into user.name
emails_of(users) = transform each user into user.email
```

The shared behavior is transformation of a collection; the varying part is the element transformation.

1. Compare the definitions, ignoring incidental names.
2. Identify corresponding differences that represent meaningful variation.
3. Replace those differences with parameters, threading them through recursive calls when necessary.
4. Define the original operations using the abstraction and rerun their checks.
5. State the generalized signature and confirm that concrete substitutions recover the original contracts.

For this example, a conventional argument order might be:

```text
map : (X -> Y), List<X> -> List<Y>

names_of(users)  = map(user -> user.name, users)
emails_of(users) = map(user -> user.email, users)
```

Use the library's real argument order and evaluation semantics. These pseudocode lines demonstrate the contract rather than propose a replacement for a built-in map.

Passing the original examples validates those uses; it does not prove every possible instantiation correct.

## Abstraction from a template

A list template contains an empty-case result and a combination of the first element with the result for the rest. Parameterizing those pieces yields a fold:

```text
fold_right : ((X, Y) -> Y), Y, List<X> -> Y
fold_right(combine, base, Empty) = base
fold_right(combine, base, Cons(first, rest)) =
    combine(first, fold_right(combine, base, rest))
```

This derives a useful abstraction without requiring two production implementations first. Likewise, an explicitly requested generic API may have sufficient requirements to design directly.

Two concrete instances are a practical guard against speculative generality, not a universal precondition. Honor the user's requested interface and validate its actual use cases.

## Use an existing abstraction

Match the desired input, output, and semantics to the available operation:

- `map` transforms each element.
- `filter` retains elements satisfying a predicate.
- `any` or `all` answers a predicate question, often with short-circuiting.
- A fold combines a traversal into a result with an explicit initial value.
- A sort orders values according to a specified comparison.

Check ordering, eager versus lazy evaluation, failure behavior, empty inputs, and costs. A loop can be clearer than a complicated fold, especially with early exits or multiple pieces of state.

Search the repository and standard library before adding a utility. Similar names do not guarantee equivalent contracts; verify the existing operation fits.

## Preserve shared meaning

Abstract code that implements the same responsibility or policy. Textual similarity alone does not establish that two operations should evolve together. Duplication can be acceptable when responsibilities differ or a premature shared interface would introduce awkward options.

Use names and types to expose the abstraction's fixed behavior and variation. If generic parameters and callbacks make the contract harder to explain than the original code, reconsider the boundary.

Keep helpers in a useful scope. A local helper can capture enclosing context and be tested through observable behavior; it need not become top-level solely because it has branching. Extract it when independent reuse, complexity, or verification makes a separate contract valuable.

## Refactor with evidence

Preserve behavior examples while extracting shared logic. Check side effects, traversal order, and numerical behavior: reassociation of floating-point sums can change results.

When possible, establish the faulty behavior before refactoring and keep the changes reviewable. If restructuring is needed to fix the defect, explain that relationship rather than enforcing an absolute ban on fixing and refactoring together.
