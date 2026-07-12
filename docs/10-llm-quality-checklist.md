# The LLM quality checklist

The operational distillation of everything in this folder. Run it on every piece of code you produce. It is ordered: the early items are cheap and prevent the expensive failures.

## Before writing code

- [ ] **Data first.** Can you state the data definition of every input and output — fields, types, units, invariants, and which values are impossible? If any parameter is "a dict/object/some JSON", stop and pin it down (read the schema, the type, the upstream producer).
- [ ] **Signature.** Written, with the most specific data definitions available. Failure cases represented explicitly (result types, documented exceptions) — not magic values.
- [ ] **Purpose statement.** One sentence, *what* not *how*. Contains no "and". If it needs "and", split the function first.
- [ ] **Examples before body.** Concrete input→output pairs, computed by hand from the problem statement: at least one per clause of every input's data definition, plus every interval boundary. If you cannot compute an expected output by hand, you don't understand the problem — re-read the source material or ask; do not guess.
- [ ] **Wish list for multi-function work.** Names + signatures + purpose statements for the whole decomposition before any body. This is the plan a human can veto cheaply.

## While writing code

- [ ] **Template check.** The body's case analysis matches the input's data definition: one branch per variant, empty/null/missing cases present, recursion (or delegation) exactly where the data definition references another definition. No improvised control flow.
- [ ] **Two complex inputs?** Consciously picked: one drives / lockstep / full case matrix. Mismatch cases (different lengths, absent keys) decided and tested, not defaulted.
- [ ] **One task per function.** Vague names (`process`, `handle`, `doStuff`) are a design alarm, not a naming problem.
- [ ] **Helpers where the rules demand them**: field of another data definition → delegate; domain shift → new function; stage-then-stage → compose; different traversal → separate function.
- [ ] **Reuse before reinvention.** Searched the codebase (and the standard library) for the existing helper/abstraction before writing a new one. Standard shapes use standard tools (`map`/`filter`/`fold`/comprehension) rather than hand-rolled loops.
- [ ] **Abstract only from two working instances.** No speculative generality; no copy-paste-tweak left standing either.
- [ ] **Every loop/recursion classified.** Structural (shape follows the data — termination free) or generative (write the four questions and a termination argument: what measure strictly decreases?). Any loop with no decreasing measure gets an explicit bound and defined behavior at the bound.
- [ ] **Every mutated variable has an invariant** you can state: what it means at the top of each iteration relative to input consumed. Accumulator invariants written as comments.
- [ ] **State minimized and fenced.** Pure functions compute decisions; a thin shell applies effects. Mutable state has a data definition, named invariants, and a closed set of mutating operations.
- [ ] **Boundaries validate, interiors trust.** Validation happens once, where data enters; no scattered defensive re-checks; errors assign blame (which promise, which side, what data).

## After writing code

- [ ] **Every example became an executable test**, plus cases discovered during coding. They run and pass — actually run, not "should pass."
- [ ] **Failing test triage**: decided whether the test or the code is wrong by recomputing the example by hand — never by editing the expectation to match the output.
- [ ] **Debugging is descent, not archaeology.** For a misbehaving program: write a failing test for the top-level function, then derive tests for each function it mentions, and recurse into the one that fails (HtDP Epilogue). Fix at the step whose artifact is wrong.
- [ ] **Case coverage audit.** Diff the tests against the data definitions: every variant, every boundary, both sides of every mismatch case. Exhaustiveness warnings are template violations, not noise.
- [ ] **Code-walk test.** Could you defend every line aloud? Every branch justified by a data definition clause; every constant named; every fact of the system in exactly one place; no reaching through representations.
- [ ] **The artifacts shipped with the code**: types/signatures in the language's syntax, purpose statements as doc comments, invariants and termination arguments as comments, examples as tests. The recipe's products are the deliverable, not scaffolding to discard.
- [ ] **Honest status report.** Anything skipped (untested path, unhandled case, assumed invariant) is stated explicitly to the requester, not left for production to discover.

## The three questions that catch most LLM-generated bugs

If the full checklist is too much for a small change, never skip these:

1. **What are all the cases of the input, and does the code visibly handle each one?** (empty, null, zero, missing key, unexpected variant)
2. **Why does every loop terminate?** (name the decreasing measure or the bound)
3. **Did I predict the outputs before running, and did I actually run them?** (examples → tests → executed)

## The disposition behind the checklist

Felleisen's deepest lesson is not any single step; it is that **plausibility is not correctness** — and that **working programs can be justifiably bad** (JFP 2004). "It works!" is "the shortest lie in computing." An LLM's native failure mode is emitting code that pattern-matches on having-seen-similar-code — the industrialized version of the tinkering HtDP was written to abolish. The recipe's artifacts (data definitions, signatures, purposes, examples, invariants, termination arguments) are precisely the things pattern-matching does not supply and cannot fake: each one is a falsifiable commitment about *this* problem. Produce them, and the quality bar rises from "looks right" to "is right, and here is the evidence."
