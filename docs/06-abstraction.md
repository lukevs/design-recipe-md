# Abstraction: from similarity to reuse

HtDP Part III teaches abstraction as a *disciplined, mechanical act* — not a speculative one. You abstract from **two concrete, working, tested definitions that are similar**, never from one definition plus a guess about the future.

Why it matters (Part III intro):

> "The similarities come about because programmers—physically or mentally—copy code. … Copying code, however, means that programmers copy mistakes, and the same fix may have to be applied to many copies. … This process is both expensive and error-prone."

> "Good programmers try to eliminate similarities as much as the programming language allows. A program is like an essay. The first version is a draft, and drafts demand editing."

The boxed guideline (§15.3): **"Form an abstraction instead of copying and modifying any code."**

## The abstraction recipe (§15.1)

1. **Compare.** "Compare two items for similarities… compare them and mark the differences. If the two definitions differ in more than one place, connect the corresponding differences with a line." Differences come in *pairs*; each connected pair becomes **one** parameter. Rename away inessential differences (function names, parameter names) first. If the definitions differ in more than a few positions, they are not similar enough — stop.

```
fun names-of(users):                 fun emails-of(users):
  if users is empty: return []         if users is empty: return []
  else: return [users.first.NAME]      else: return [users.first.EMAIL]
        + names-of(users.rest)               + emails-of(users.rest)
```

2. **Abstract.** "Replace the contents of corresponding code highlights with new names and add these names to the parameter list" — one parameter per linked pair, threaded through the recursive calls too. The two definitions become identical; keep one:

```
fun map(f, items):
  if items is empty: return []
  else: return [f(items.first)] + map(f, items.rest)
```

3. **Validate by re-definition.** "To validate means to test, which here means to define the two original functions in terms of the abstraction… So reformulate and rerun those tests… and make sure they succeed." The abstraction is *proven* against two known-good instances before anyone else uses it:

```
fun names-of(users):  return map(u -> u.NAME, users)
fun emails-of(users): return map(u -> u.EMAIL, users)
```

4. **Formulate the parametric signature.** Signatures are data definitions, so the same compare-and-generalize move applies to them: replace the paired differences with type variables — `map : [X Y] List<X>, (X -> Y) -> List<Y>`. Then *test the signature*: substituting concrete types must recover both originals' signatures, and the generalized signature must be in sync with the code (a parameter typed `(A, B) -> C` must always be applied to an A and a B, its result used as a C). The signature documents exactly *what varies* and *what is fixed*.

5. **Look for further uses.** "Once you have abstracted two functions, you should check whether there are other uses for the abstract function. If so, the abstraction is truly useful… add it to a library of useful functions."

The same recipe abstracts **data definitions**: `List<ITEM>`, `Maybe<X>` — parametric/generic types are the abstraction recipe applied to data.

## When to abstract — and when not to

- **Two concrete instances first.** The recipe's precondition is two existing, tested definitions. Speculative abstraction ("we might need to swap databases someday") produces parameters nothing varies and indirection nothing needs. Duplication is cheaper to fix than the wrong abstraction.
- **Single point of control is the payoff** (§15.3): "When you discover a mistake, you have to go to just one place to fix it… If you had made copies… you would have to find all copies and fix them; otherwise the mistake might live on."
- **Abstract shared meaning, not shared syntax.** Two functions that are textually similar but conceptually unrelated (a retry loop and a pagination loop) should stay separate — their futures diverge.
- **Editing is a scheduled pass, not a maybe.** The Epilogue: "a function or program isn't done the first time it passes the test suite. You must find time to inspect it for design flaws and repetitions of designs. If you find any design patterns, form new abstractions or use existing abstractions to eliminate these patterns."
- **Correctness first, elegance second.** The Little Schemer (Friedman & Felleisen), Sixth Commandment: "Simplify only after the function is correct." Its Ninth is this whole chapter in 1974 form: "Abstract common patterns with a new function." Never refactor and fix in the same motion — get it right against the tests, then simplify with the tests as a net.

## Using existing abstractions (§16.6)

The complementary skill: a designed ecosystem already packages the common templates — `map`, `filter`, `fold`, `sort`, `any`/`all`, `build-list`. "The use of an abstraction helps readers of your code understand your intentions. If the abstraction is well-known and built into the language… it signals more clearly what your function does than custom-designed code" (ch. 16).

HtDP's recipe for reuse is **signature-driven matching**:

1. Run the ordinary recipe through signature, purpose, examples, stub.
2. "Exploit the signature and purpose statement to find a matching abstraction… It is often best to start with the desired output and to find an abstraction that has the same or a more general output." (Need an `Image` from a `List<Posn>`? `fold`'s result variable matches; unification then *tells you* the helper's signature: `Posn, Image -> Image`.)
3. Template: note which abstraction, and stub the helper function — most of its signature came free from step 2.
4. Define the helper (it may close over the enclosing function's parameters).
5. Test as usual.

The same rule scales to libraries and codebases: prefer the ecosystem's existing, tested abstraction — the date library, the schema validator, the retry helper *already in this repo* — over a bespoke reimplementation, which is a second point of control for a solved problem.

## Local definitions and scope (§16.2–16.4, Intermezzo 3)

Define things in the smallest scope that works. A helper used by one function can live inside it — and local definition *adds expressive power*: the inner function can use the enclosing parameters directly, eliminating argument-threading. Name intermediate results instead of repeating computations. But a local definition can't be tested independently — if it's complex enough to deserve its own examples, promote it to a top-level (private) function with the full recipe. Intermezzo 3's motto: **"Repetition calls for abstraction."** Its note on loop constructs applies to every modern language: syntactic loops and comprehensions "signal intentions more directly" than hand-rolled recursion — use the form that names its pattern.

## LLM directives

- Before writing any loop or traversal, check whether it is a standard abstraction (`map`/`filter`/`fold`/comprehension). Hand-rolled loops are the assembly language of data processing.
- Before writing any new utility, **search the codebase** for the existing one. Writing the second `formatDate` helper in a repo is an abstraction failure regardless of how good it is.
- When you notice you are about to copy-paste-and-tweak — your own output from earlier in the session or the codebase's code — stop: call the existing function, or run the abstraction recipe on the two instances. Never leave the near-duplicate.
- Apply the two-instances rule in reverse: asked for "a flexible/generic/pluggable X" with one concrete use case in sight, build the concrete X well and note that abstraction should wait for the second instance.
- After the tests pass, run the draft-editing pass: scan what you wrote for repeated shapes and eliminate them. Passing tests is the midpoint of done, not the end.
