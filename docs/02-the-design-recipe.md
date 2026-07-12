# The design recipe

The design recipe is HtDP's core contribution: a six-step process from problem statement to tested function. Every function gets the recipe. The steps are ordered so each consumes the product of the previous one; skipping a step means guessing at what it would have told you.

> "Until the steps become second nature, never skip one because doing so leads to easily avoidable errors. There is plenty of room left in programming for complicated errors; we have no need to waste our time on silly ones." — HtDP §3.2

The six steps, as HtDP's Preface (Figure 1) states them:

## Step 1 — From problem analysis to data definitions

> "Identify the information that must be represented and how it is represented in the chosen programming language. Formulate data definitions and illustrate them with examples."

Programs compute with *data*; problems are stated as *information*. Decide the representation (information → data) and the interpretation (data → information), and write both down (see `03-data-definitions.md`):

```
; A Temperature is a Number.
; interpretation represents Celsius degrees
```

Then construct **data examples** — actual values of the class — to prove the encoding works both directions. In typed languages the type declaration carries part of this, rarely all of it (the range constraint, the units, the meaning). Write the rest anyway.

## Step 2 — Signature, purpose statement, header

> "State what kind of data the desired function consumes and produces. Formulate a concise answer to the question *what the function computes*. Define a stub that lives up to the signature."

**Signature** — `; Temperature -> String`. Use the most specific data definitions available; the signature also says, implicitly, "which part of the universe of data it won't deal with" (HtDP §5.7). If the function only works on non-empty lists, the signature says `NEList`, not `List` — narrowing the domain in the data definition is HtDP's own move for making functions total.

**Purpose statement** — one line. HtDP §3.1: "If you are ever in doubt about a purpose statement, write down the shortest possible answer to the question **what does the function compute?** Every reader of your program should understand what your functions compute without having to read the function itself." Use the parameter names in it ("adds `s` to `img`, `y` pixels from the top"). Focus on *what*, never *how* — with one exception: generative recursion, where the "how" becomes mandatory (`07-generative-recursion.md`). Multi-function programs get two purpose statements: one for the reader who must modify the code, one for the person who wants to use it without reading it.

**Header (stub)** — a runnable definition whose body is any dummy value from the output class. The program compiles and runs from the very first step, so examples can be checked for well-formedness immediately.

## Step 3 — Functional examples

> "Work through examples that illustrate the function's purpose."

Concrete input→output pairs, worked out by hand **before any code**, placed between purpose and header (or written directly as tests):

```
; given: 2, expect: 4
; given: 7, expect: 49
```

Coverage rules (HtDP §4.6, ch. 9):
- "It is imperative that you pick at least one example from each sub-class in the data definition."
- "If a sub-class is a finite range, be sure to pick examples from the boundaries of the range and from its interior."
- For self-referential data, "work through inputs that use the self-referential clause of the data definition several times."

Examples are where hidden ambiguity surfaces. HtDP's sales-tax walkthrough: working the boundary example reveals the problem statement's "less than $1,000" is ambiguous — and the designer must decide, or ask: "A programmer at a tax company would have to ask a tax-law specialist." For an LLM this is the highest-leverage step: it converts a fuzzy natural-language request into concrete committed behavior the requester can inspect and veto. If you can't compute an expected output by hand, you don't understand the problem — re-read or ask; don't guess.

A refinement that pays off later: express the expected value as a *computation* where possible — `check-expect (sales-tax 1000) (* 0.05 1000)` — "This makes it easier later to formulate the function definition."

## Step 4 — Function template

> "Translate the data definitions into an outline of the function."

The template is derived *mechanically* from the input's data definition — "the template mirrors the organization of sub-classes with a `cond`" (§4.6) — one branch per clause, selectors for structure fields, a natural recursion per self-reference. HtDP calls it taking **inventory**: "an organization schema for everything we know about the data definition." It contains no problem-specific logic; two functions over the same data start from the identical template. Full derivation rules and the template questions: `04-templates.md`.

Escape hatch (§6.1): "If the problem statement suggests that there are several tasks to be performed, it is likely that a composition of several, separately designed functions is needed instead of a template. In that case, skip the template step" — and go to the wish list (`05-composing-functions.md`).

## Step 5 — Function definition

> "Fill in the gaps in the function template. Exploit the purpose statement and the examples."

The book calls this "the one creative step," made tractable by the scaffolding. Start with the **base cases** (clauses without recursion) — the examples usually give their answers directly. Then the recursive clauses, using HtDP's four questions (ch. 9, Figure 53):

1. *What are the answers for the non-recursive cond clauses?* — "The examples should tell you which values you need here. If not, formulate appropriate examples and tests."
2. *What do the selector expressions in the recursive clauses compute?* — the data definition's interpretation tells you.
3. *What do the natural recursions compute?* — "Use the purpose statement of the function to determine what the value of the recursion means, not how it computes this answer. **If the purpose statement doesn't tell you the answer, improve the purpose statement.**"
4. *How can the function combine these values to get the desired answer?* — find a combining operation; "if that doesn't work, make a wish" for a helper function. HtDP calls the combining operation the **combinator**.

Trusting the natural recursion is what the book calls a **leap of faith**: "we assume that the function already works as specified in our purpose statement… this leap is always justified, which is why it is an inherent part of the design recipe" (it is the induction hypothesis of a proof by induction). Tracing recursion mentally is tinkering; trusting the purpose statement is design.

**Stuck on the combinator? Use the table method** (Figure 54): arrange the examples in a table — input in the first column, desired output in the last, values of each selector expression and each natural recursion in between. Add examples until a pattern connecting the middle columns to the last one emerges. This turns "stare at the code and hope" into a mechanical search.

## Step 6 — Testing

> "Articulate the examples as tests and ensure that the function passes all. Doing so discovers mistakes. Tests also supplement examples in that they help others read and understand the definition when the need arises—and it will arise for any serious program."

- **Mechanize.** Manual testing "quickly becomes a labor-intensive chore… programmers start to neglect it. At the same time, testing is the first tool for discovering and preventing basic flaws. Sloppy testing quickly leads to buggy functions." Tests re-run automatically after every change.
- **Failure triage.** When a test fails there are exactly three possibilities: the expected value is wrong, the function is wrong, or both. "First reassure yourself that the expected results are correct. If so, assume that the mistake is in the function definition." Never edit the expectation to match the output without re-deriving it by hand from the problem statement.
- **Coverage.** "Run the tests and ensure that they cover all cond clauses" — every branch of the template must be exercised. "Test as soon as the function header is written. Test until all expressions have been covered. Test again when you make changes." (§5.8)
- **Failing tests mid-build are healthy.** With stubs and wish lists in play, "many tests will fail. That's progress." Done means: wish list empty, all tests pass, all branches covered.
- **Isolate nondeterminism to keep code testable**: split a function that uses randomness (or the clock, or I/O) into a thin wrapper applying the nondeterminism and a testable pure core — "most of the code remains testable" (§6.1, ex. 99).

## The recipe as an error-localization tool

Each step has a concrete product, so failures trace to a step:

| Symptom | Broken step |
|---|---|
| Can't state what values are valid inputs | 1: no data definition |
| Callers keep converting arguments / awkward to call | 2: wrong signature |
| Can't say what it returns without reading the body | 2: no real purpose statement |
| Surprised by an edge case in production | 3: examples didn't cover a clause or boundary |
| Deeply nested improvised if/else | 4: template ignored the data definition |
| A branch you can't fill in | 5: missing helper or accumulator |
| "It works on my example" | 6: examples never became tests |

This is the recipe's deepest property: it creates **intermediate products** that can be inspected. "When a novice is stuck, an expert or an instructor can inspect the existing intermediate products… and thus drive the novice to correct himself or herself" (Preface). When you are stuck, identify which step you're stuck on and repair that step's artifact — don't flail at the code.

## Debugging, recipe-style (Epilogue)

When a whole program misbehaves: formulate a failing test for the main function; then derive tests for each function it mentions; recurse into whichever one fails. Debugging is test-driven descent through the call graph, not print-statement archaeology.

## LLM directives

- Emit the artifacts, don't just think them: the signature is the type annotation, the purpose statement is the docstring, the examples are the test file. The recipe's products map one-to-one onto professional practice and are the deliverable, not scaffolding.
- The recipe scales both ways: "it works for 10-line programs as well as for 10,000-line programs" (Preface). For a module or service, step 1 is the domain model/schema, step 3 is acceptance examples, and the wish list is the function inventory.
- Never present code as an answer while step 3 or step 6 is missing. If constraints genuinely prevent tests, say so explicitly rather than silently skipping.
