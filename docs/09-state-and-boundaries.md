# State, contracts, and boundaries

HtDP 2e teaches computation as functions from data to data — and, remarkably, contains **no assignment or mutation anywhere**. That is a design statement, not an omission. The book elevates purity to a principle: "we wish to retain the principle that `(f a)` returns the same result no matter how often or where it is evaluated" (Part VI intro) — even *context* is threaded through accumulator parameters rather than mutated. Stateful *behavior* (interactive programs, editors, games) is modeled throughout with pure state-transition functions; imperative details are deferred to later study because they are harder to test, harder to reason about, and admit whole categories of bugs (aliasing, ordering, races) that pure functions cannot have. Felleisen's research work (higher-order contracts, Racket's module system) extends the same philosophy to program boundaries: **every boundary states its promises, and blame for a violation is assignable.**

## Functional core, thin stateful shell

The design consequence of HtDP's ordering:

- **Default to functions.** A function that takes data and returns data can be designed by the recipe, tested with plain examples, and composed freely.
- **Introduce state only for what is genuinely stateful**: the world's actual state (the database, the file, the socket), caches, and interaction. Represent *program* state as data passed and returned, not as mutation, until the boundary forces otherwise.
- **Keep the stateful layer thin.** The classic shape: pure functions decide *what* to do (returning a description of the change); a small imperative shell *does* it. Business logic that lives inside the shell inherits the shell's untestability.

HtDP's world programs (`big-bang`) model this exactly: "Those properties that change over time—in reaction to clock ticks, keystrokes, or mouse actions—give rise to the current state of the world" (§3.6) — the *world state* gets a data definition; each event handler is a **pure function** `WorldState, Event -> WorldState`; the framework owns the loop. The world-program design recipe: (1) name the constants; (2) design the data representation of the changing state — "Choose simple forms of data to represent the state of the world"; (3) design each handler by the ordinary function recipe (the handlers are the program's initial wish list); (4) a trivial `main` assembling the loop — the only function exempt from design and testing, because it contains no logic. This pattern — explicit state data definition, pure transition functions, a dumb loop — reappears as Elm/Redux reducers, state machines, game loops, and Temporal-style workflow engines. When asked to build anything interactive or event-driven, reach for it.

HtDP also endorses the boundary split directly (§3.1): separating data processing from parsing input and producing output "is now accepted wisdom that well-engineered software systems enforce… model-view-controller architecture."

## Designing state that must exist

When mutation is genuinely required, the recipe still applies, with additions:

1. **Give the state a data definition** — including its invariants ("the cache maps user IDs to *fresh-as-of-timestamp* profiles"; "`balance` is never negative").
2. **Enumerate the operations** that may change it — and make them the *only* way to change it (single point of control for each state change). Ad-hoc mutation from arbitrary call sites is how invariants die.
3. **Each operation gets a purpose statement covering its *effect***, not just its return value: "adds the item and increments the count" — effects are part of the specification and part of what examples/tests must check.
4. **State the invariant preservation**: every operation assumes the invariant on entry and must re-establish it on exit. (This is the accumulator invariant discipline, `08-accumulators.md`, applied to long-lived state.)
5. **Beware aliasing.** Once data is mutable, two references to the same value make every mutation a potential action-at-a-distance. Prefer immutable data for anything that is shared; copy at boundaries when ownership is unclear.

## Contracts: promises at boundaries

Felleisen's contract work (Findler & Felleisen, "Contracts for Higher-Order Functions", ICFP 2002; Racket's contract system) turns signatures and invariants into *enforced, blame-assigning* boundary checks. The Racket Guide's framing: a contract is "an agreement between two parties" that "specifies obligations and guarantees for each value that is handed from one party to the other"; contracts attach to *module boundaries*, and on violation the system "blames the module for breaking its promises." The research contribution was precisely **blame assignment** — for first-order calls it's trivial (bad argument: caller's fault; bad result: callee's fault); the hard, solved problem was tracking responsibility for function values that cross boundaries repeatedly. The language-agnostic discipline:

- **A signature is a contract**: the caller promises the inputs, the function promises the result. Write signatures precise enough to be promises (`NonEmptyList`, `SortedList`, `PositiveInt`, `SanitizedHtml` — not `list`, `int`, `str`).
- **Validate at boundaries, trust inside.** Where data enters the system (API handler, file parser, queue consumer, user input), validate it into a data definition *once*, producing a value whose type/name certifies the invariant. Interior functions then *rely* on the definition instead of defensively re-checking. Scattered defensive checks are worse than useless: they hide where validation actually happened and mask the true boundary.
- **Assign blame.** When a contract is violated, the error should say *which side broke the promise* — bad input (caller's fault) or bad result (callee's fault). Error messages like "invalid state" assign no blame; "expected sorted list at merge(): caller passed unsorted data" does. Design your assertions and error types so failures point at the responsible boundary.
- **Assert what you can't type.** Where the type system can't express an invariant, an assertion at the constructor/boundary is the executable form of the data definition's fine print.

## Interfaces and information hiding

The module-level version of the same idea, as taught in Felleisen's software-development courses:

- A component exports a **small interface of operations with signatures and purpose statements**; its data representation stays private. Clients that reach into the representation make it permanently unchangeable.
- Design the interface from the *client's* wish list ("what do callers need to ask?"), not from the representation ("what fields do we happen to have?").
- Every interface operation is specified well enough that a client can be written — and a mock can be built — from the spec alone. If you must read the implementation to use it, the interface is unfinished.
- CS4500's litmus test for whether you've built a component at all: can you answer "how do I construct a [domain object] from an instance of my chosen data representation?" — "If you can answer [that] you have a software component. If you don't… you have 'stuff.'"

**Testability forces this architecture.** The JFP 2004 paper's observation: to write automatic tests at all, "students must split a program into a part that deals with computation proper (the 'model') and another part that interacts with the user (the 'view')… they don't want to print results but hand them over directly to a comparison function." If code is hard to test, that difficulty is the design smell — the model and the shell are entangled. Fix the boundary, and the tests become easy; never respond by testing less.

## Errors are data

Failure cases belong in signatures as itemizations (`Result = Ok(value) | Err(reason)` or checked exceptions or documented raised errors — the idiom varies, the principle doesn't):

- Decide **for each function** whether it can fail, how the failure is represented, and what the caller can do about it — at design time, in the signature, not ad hoc at the first traceback.
- Never encode failure as in-band magic values (`-1`, `null`-that-means-error, empty string) — that's the itemization-hiding smell from `03-data-definitions.md`, and it disables the template's case analysis at every call site.
- Handle-or-propagate, explicitly. Swallowed exceptions and bare `except: pass` destroy the blame chain.

## LLM directives

- Before writing a class with mutable fields, ask whether a pure function over an explicit state value does the job. Prefer the pure version unless the state is genuinely long-lived and shared.
- When touching stateful code, identify its invariants first (from the data definition if someone wrote it; by reconstruction if not) and verify your edit preserves every one. State-editing without invariant reconstruction is the highest-risk edit an LLM performs.
- Put validation at the boundary the data actually crosses, not at the function you happen to be writing. If validation already exists upstream, don't duplicate it downstream "to be safe" — find the boundary and trust it, or fix the boundary.
- Make every error you introduce carry blame: what promise, which side, what data. Your future debugging self is the client.
