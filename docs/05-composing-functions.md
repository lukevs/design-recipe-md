# Composing functions: wish lists and helper discipline

A program is not one function; it is a web of small functions, each designed by the recipe. HtDP's tools for managing the web: the wish list, two standing guidelines, and explicit rules for when a helper function must exist.

## The wish list (HtDP §3.4, §11.2)

While designing any function, you will discover computations you need but don't have. **Do not inline them. Wish for them.**

> "Each entry on a wish list should consist of three things: a meaningful name for the function, a signature, and a purpose statement. … As long as the list isn't empty, pick a wish and design the function. If you discover during the design that you need another function, put it on the list. When the list is empty, you are done." — §3.4

Wishes are written as full stubs — signature, purpose, header returning a dummy value — so the whole program runs and tests execute (and fail informatively) at every moment: "Writing down complete function headers ensures that you can test those portions of the programs that you have finished, which is useful even though many tests will fail. Of course, when the wish list is empty, all tests should pass" (§11.2). Mid-build failing tests are progress markers, not embarrassments: "Stop! Test the program as is. Some test cases pass, and some fail. That's progress." (§11.3)

**Before wishing, search**: "Before you put a function on the wish list, you should check whether something like the function already exists in your language's teachpack or whether something similar is already on the wish list" (§11.2). In a codebase: search the repo and the standard library first.

Why the wish list works:

- **One function at a time.** You never hold two half-designed functions in your head. Each function is designed against its collaborators' *signatures and purpose statements*, not their bodies.
- **Top-down planning, bottom-up verification.** "The design of a program proceeds in a top-down planning phase followed by a bottom-up construction phase" (Preface). Present code the same way: "function definitions are presented 'top down,' starting with the main function" (ch. 7).
- **It is the natural LLM planning artifact.** Enumerate the functions a feature needs — name, signature, purpose for each — before writing any body. The plan is inspectable and correctable at the cheapest possible moment.

## Two standing guidelines (§11.2)

> "**Design one function per task.** Formulate auxiliary function definitions for every dependency between quantities in the problem."

> "**Design one template per data definition.** Formulate auxiliary function definitions when one data definition points to a second data definition."

Symptoms of violating the first: the purpose statement contains "and" ("parses the config **and** connects to the database"); the name is vague (`process`, `handle`, `doStuff`) because no honest specific name exists; parameters used only in some branches. The fix is always the same: split along the "and", wish for the parts. (HtDP's own worked case: `average` "is a function of three tasks: summing, counting, and dividing… write one function per task.")

## When a helper is mandatory: HtDP's four situations (§11.2)

During the definition step, an auxiliary function is called for when:

1. **The combination requires domain knowledge** — "If the composition of values requires knowledge of a particular domain of application… design an auxiliary function." Geometry inside rendering, tax law inside checkout, retry policy inside business logic: each domain gets its own function, designed and tested in that domain's terms.
2. **The combination requires a case analysis** — "If the cond looks complex, design an auxiliary function whose arguments are the template's expressions and whose body is the cond expression."
3. **The combination must traverse self-referential data** — "If the composition of values must process an element from a self-referential data definition—a list, a natural number, or something like those—design an auxiliary function." One function, one template: a second traversal inside a branch is always its own function (HtDP's `insert` inside `sort`).
4. **Everything fails → generalize** — "If everything fails, you may need to design a *more general function* and define the main function as a specific use of the general function. This suggestion sounds counterintuitive, but it is called for in a remarkably large number of cases." When the direct function is unwritable, a slightly more general problem often has a clean structural solution (and often the generalization is an accumulator — `08-accumulators.md`).

Kiczales' *How to Code* (the UBC course built on HtDP) operationalizes these as four crisp helper rules: **reference rule** (a reference to another non-primitive data definition in a type → a call to that type's function in the template), **function-composition rule** ("when a single function includes two or more distinct operations… break them up into separate functions"), **knowledge-domain-shift rule** ("if a subtask involves special domain knowledge a helper function should be used" — sorting images vs. comparing image sizes is already a shift), and **arbitrary-sized-data rule** ("when an expression must operate on a list — and go arbitrarily far into that list — it must call a helper function to do that"). Students complain the approach "results in lots and lots of extra code" — that is the cost of every function having one template and one justification, and it is worth paying.

## Design by composition

HtDP 2e names a second decomposition mode: sometimes the insight is that *inventing an intermediate data definition* splits the problem — one function produces the intermediate data, another consumes it ("sort, then take top 3"; "parse to AST, then evaluate"). "This approach also needs a wish list, but formulating these wishes calls for an insightful development of an intermediate data definition" (Preface). When a single function resists its template, ask: what intermediate representation would make this two easy functions?

## Function-level hygiene and the code walk

Felleisen's Northeastern curriculum (Fundamentals I through the CS4500 Software Development capstone) makes design *social*: pair programming where the co-pilot "checks the evolving design against the question-and-answer game from the design recipe and questions any deviations," and formal **code walks** where authors "present the components and their functionality in a top-down fashion," must ground every claim "in concrete pieces of design documents, comments, or lines of code," and must "defend their design decisions" — knowing both "when to acknowledge a potential problem" and "when it is time to reject criticism." The walks exist because code that works can still fail review: they "discover questionable design decisions, mistakes, and omissions." The capstone's framing quote: **"If your software survives the prototype stage, it will survive you."** Its method: "plan top-down, build bottom-up."

An LLM has no panel in the room, so it plays both roles: after writing, re-read your own code as the assistant reader whose job is to find mistakes and omissions. The rules to self-enforce:

- **Names say what things are.** A name is a one-word purpose statement; parameter names should "reflect what kind of data the parameter represents" or its purpose (§3.1). If naming is hard, the design is wrong — a thing with no honest name is usually two things.
- **Single point of control.** "Good programmers establish a single point of control for all aspects of their programs" (§3.6): every fact — a constant, a format, a rule — lives in exactly one place. Named, computed constants (change `WHEEL-RADIUS`, the whole car resizes) rather than magic literals scattered as unlabeled assumptions.
- **No reaching through representations.** Callers use a data definition's interface, not its innards.
- **Write for readers.** "Programmers write programs for other programmers to read" — where "other" "also includes older versions of the programmer who usually cannot recall all the thinking that the younger version put into the production of the program" (ch. 3). The Epilogue: "the design structure of programs is really a means of communication among programmers across time."
- **Defend every line.** If you couldn't justify a line in a code walk ("why is this branch here?" — "the data definition has three clauses"), the line isn't done.

## Composition over configuration

When a function grows parameters that switch its behavior (`render(item, compact=True, skipHeader=False, forEmail=True)`), it has become several functions sharing a body. Prefer several honestly-named functions composed from shared helpers. Boolean parameters that select behavior usually mark suppressed wish-list entries.

## LLM directives

- For any task needing more than one function, **emit the wish list first**: names, signatures, purpose statements. This is the artifact to show a human before writing bodies.
- Design against signatures: when calling a collaborator, trust its purpose statement; don't defensively re-validate what its signature promises.
- When editing existing code, respect the existing decomposition; if the task doesn't fit it, extend the wish list rather than swelling the nearest function.
- Never leave a wish silently unfulfilled. Every stub either gets a body and tests, or an explicit TODO stating it is unimplemented — dummy-returning stubs presented as done are worse than crashes.
