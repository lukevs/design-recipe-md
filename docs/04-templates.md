# Templates: code shape from data shape

A **template** is the skeleton of a function derived mechanically from the data definition of its main input — before any problem-specific thinking. "The purpose of a template is to express the data definition as a function layout… all important pieces of the data definition must find a counterpart in the template" (HtDP ch. 9). Two functions over the same data class start from the *identical* template; only the hole-filling differs. This is the concrete meaning of "the shape of the data determines the shape of the program."

## The template questions (HtDP ch. 9, Figure 52)

Ask these of the input's data definition, in order:

1. **Does the data definition distinguish among different sub-classes of data?** → "Your template needs as many cond clauses as sub-classes that the data definition distinguishes." (Not fewer — missed case; not more — unreachable branch. "If it has seventeen sub-classes, the cond expression contains seventeen clauses.")
2. **How do the sub-classes differ from each other?** → "Use the differences to formulate a condition per clause" — a test that identifies each sub-class from the value itself.
3. **Do any of the clauses deal with structured values?** → "Add appropriate selector expressions to the clause" — every field, listed as available raw material, even ones you suspect you won't need ("the template is an organization schema for everything we know about the data definition, but we may not need all of these pieces for the actual definition").
4. **Does the data definition use self-references?** → "Formulate 'natural recursions' for the template to represent the self-references of the data definition" — a recursive call at exactly each self-reference position, no more and no fewer.
5. **Does the data definition refer to some other data definition?** → "Specialize the template for the other data definition. Refer to this template" — i.e., a call to *that* class's function, not inline drilling.

Global constants also belong in the inventory — "they belong to the inventory of things that may contribute to the function definition" (§3.4).

## Worked shapes

**Enumeration** →

```
fun describe(light):        ; TrafficLight -> String
  match light:
    "red":    ...
    "yellow": ...
    "green":  ...
```

**Structure** →

```
fun process(coord):         ; Coordinate -> ???
  ... coord.x ... coord.y ...
```

**Self-referential list** →

```
fun process(items):         ; ListOfX -> ???
  if items is empty:  ...
  else:               ... items.first ... process(items.rest) ...
```

**Tree / mutually referential nest** → one function per data definition, calling each other where the definitions refer to each other:

```
fun size-entry(e):          ; FileSystemEntry -> Number
  match e:
    File(name, size):         ... name ... size ...
    Directory(name, entries): ... name ... size-entries(entries) ...

fun size-entries(es):       ; List<FileSystemEntry> -> Number
  if es is empty:  ...
  else:            ... size-entry(es.first) ... size-entries(es.rest) ...
```

A data definition with *two* self-references (a binary tree's left and right) yields a template with *two* natural recursions. The arrows in the data definitions and the calls in the templates correspond one-to-one.

## Trust the natural recursion

When filling a hole next to a recursive call, **do not trace the recursion**. Ask only: *the purpose statement says this call returns X for the rest of the input — how do I combine X with this clause's other pieces?* HtDP: "For the natural recursion we assume that the function already works as specified in our purpose statement. This last step is a leap of faith, but… it always works" (it is the induction hypothesis of a proof by induction). Tracing recursion mentally is tinkering; trusting the purpose statement is design.

## Templates in modern languages

- **Pattern matching** (Rust/Python `match`, TS discriminated-union narrowing, sealed classes): the template *is* the match statement; exhaustiveness checking is the compiler enforcing the one-branch-per-clause rule. Prefer these constructs — they machine-check your template. (HtDP's own Intermezzo 3 introduces `match` as exactly this: an abstraction over predicate+selector conditionals.)
- **Iteration**: `for`/`map`/`filter`/`fold` over a collection is a pre-packaged list template (see `06-abstraction.md`). Using them is not skipping the template — it's recognizing that your template matches a standard one.
- **Visitors / recursion schemes**: packaged tree templates.
- **API layering**: a function that reaches through its input (`user.account.plan.limits`) violates question 5 — the `User` template may use `user.account` and must delegate `Account` processing to an `Account` function. (Elsewhere called the Law of Demeter; here it falls out of template discipline.)

## Two complex inputs (HtDP ch. 23)

When a function consumes **two** structurally complex arguments, "how to design such functions depends on the relationship between the arguments." There are exactly three cases — decide which *before* coding (§23.5):

1. **"If one of the parameters plays a dominant role, think of the other as an atomic piece of data as far as the function is concerned."** Template on the dominant one; the other rides along (and often becomes the base-case answer — e.g., appending: recur on the first list, return the second when the first is empty).
2. **Synchronized/lockstep** — the parameters "must have the same size… and are processed in a synchronized manner": template on one, take the rest of *both* in the recursion (zip, pairwise wages). The size assumption is stated explicitly, and what happens on mismatch is decided and tested, not defaulted.
3. **"If there is no obvious connection between the two parameters, you must analyze all possible cases."** Build a **two-dimensional table**: one axis lists the case questions for the first argument, the other axis those for the second. Each cell is a `cond` clause whose condition is the *and* of its row and column conditions; "the examples must cover all possible cases; that is, there must be at least one example per cell in the table." Within a cell, *every* combination of selector expressions is a candidate natural recursion — when several are plausible, run a concrete example through each to decide.

**Simplify after, never instead.** The exhaustive table version often collapses (some conditions are impossible, some checks become redundant) — but derive the full version first and simplify with justified steps. HtDP §23.4: "If we try to find the simple versions of functions directly, we sooner or later fail to take care of a case in our analysis, and we are guaranteed to produce flawed programs." The classic production bug is writing case-1 code for a case-3 problem and meeting the missing cells in production.

## What templates prevent

- **Missed cases**: the empty list, the `null` variant, the enum value added last quarter. The template forces every clause into view before logic distracts you.
- **Improvised control flow**: nested `if`s that reflect the order thoughts occurred rather than the structure of the data.
- **Structure-blind code**: string-munging a URL instead of parsing it; regexing JSON instead of traversing it. If the data definition is recursive, flat code over its serialized form will break on nesting.

## LLM directives

- Before writing a function body, derive the template: how many cases does the input's definition have? Where are the self-references and cross-references? Then check drafted code against it — every missing branch is a latent bug.
- Treat exhaustiveness warnings as template violations, never as noise to suppress with a `default:` branch. When using a language-supplied enumeration with many irrelevant cases, collapsing them into an `else` is acceptable — but as HtDP notes, "this kind of rearrangement is done *after* the function is designed properly."
- When modifying existing code, first identify which template the function follows (or fails to follow). A fix that respects the template is small; a fix bolted onto a template violation should trigger a proposal to restructure.
- Templates are also a performance discipline: HtDP's Intermezzo 5 notes that swapping the template's O(1) predicates/selectors for whole-structure operations (like recomputing a length in every condition) "may shift performance from one class of functions to one that is much worse."
