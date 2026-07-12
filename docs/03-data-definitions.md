# Data definitions: the taxonomy of data

Design begins with data, not code. A **data definition** names a class of values, says exactly which values belong to it, and states how to interpret those values as information about the problem domain. HtDP §8.1, on being handed the untyped value `("Robbie Round" 3 #true)`: "Without a data definition, you just can't know what data is all about."

A data definition "serves two purposes. First, it names a collection of data—a class—using a meaningful word. Second, it informs readers how to create elements of this class and how to decide whether some arbitrary piece of data belongs to the collection" (§3.1). HtDP develops a small taxonomy; nearly all data you'll ever design is built from these forms. Each form dictates a code shape (see `04-templates.md`) — this is why the taxonomy matters.

## 1. Atomic data

A single indivisible value: a number, a string, a boolean, an image, an ID.

```
; A Distance is a non-negative number
; interpretation: meters from the start line
```

Even atomic data deserves an interpretation — units, encoding, and constraints are where atomic-data bugs live (seconds vs. milliseconds, meters vs. feet, UTC vs. local).

## 2. Intervals

Atomic data restricted to a range.

```
; A GradePercent is a number in [0, 100]
```

Design obligation: every interval boundary becomes a test example, and you must decide open vs. closed at each end. When a function's behavior differs across sub-ranges (tax brackets, shipping tiers), the *intervals are the data definition* and the function gets one branch per interval — with boundary tests at each dividing point. HtDP §4.4 warns specifically about **overlapping endpoints** between adjacent intervals: "Such overlaps usually cause problems for programs, and they ought to be avoided" — make each boundary belong to exactly one interval, explicitly.

## 3. Enumerations

A finite set of named values.

```
; A TrafficLight is one of: "red" | "yellow" | "green"
```

Code obligation: functions over an enumeration handle **every** named value, explicitly. In languages with exhaustive `match`/`switch` checking, use it; without it, make the final branch an explicit error, never a silent default that swallows new variants.

## 4. Itemizations (unions)

The general form of enumerations: a finite set of *alternatives* that may themselves be arbitrary data classes.

```
; A SearchResult is one of:
; - NotFound
; - Found(position: Number)

; A MaybeUser is one of:
; - null
; - User
```

This is the language-agnostic root of `Option`/`Maybe`, tagged unions, sum types, and sealed class hierarchies. The design rule: **when a value can be one of several kinds, say so in the data definition** — do not encode variants with magic values (`-1` for "not found", `""` for "missing") that share a representation with legitimate data. Magic-value encodings destroy the template: the code can no longer branch on the data's shape, only on folklore.

## 5. Structures (compound data)

Several values that belong together, with named parts.

```
; A Coordinate is (x: Number, y: Number)
; interpretation: a point on the canvas; x pixels from left, y pixels from top
```

Design rule: reach for a structure **when several pieces of information always travel together**. Conversely: if two fields never co-vary or one is meaningless for some variants, you likely have an itemization of two structures, not one structure with optional fields. ("A field that is sometimes `null` depending on another field" is an itemization wearing a structure costume — split it.)

## 6. Self-referential (recursive) data

Data of arbitrary size: the definition refers to itself. It needs **at least one self-reference and at least one base case**, or no finite value could ever be constructed.

```
; A ListOfString is one of:
; - empty
; - (first: String, rest: ListOfString)

; A FileSystemEntry is one of:
; - File(name: String, size: Number)
; - Directory(name: String, entries: List<FileSystemEntry>)
```

Lists, trees, JSON documents, ASTs, org charts, comment threads, nested menus — all self-referential. The self-reference in the *data* is exactly where recursion (or iteration) appears in the *code*.

Validity criterion (HtDP ch. 8): a self-referential data definition must have **at least two clauses, and at least one clause that does not refer back to the class being defined** — otherwise no finite value could ever be constructed. Validate any recursive definition by *generating examples from it*: "Start with the clause that does not refer to the data definition; continue with the other one, using the first example where the clause refers to the definition itself… If it is impossible to generate examples from the data definition, it is invalid."

Domain narrowing is a first-class move: when a function can't sensibly handle a clause (what's the average of zero temperatures?), HtDP's answer is a *new, narrower data definition* (`NEList` — non-empty list) rather than a runtime check or a garbage return value. Fix the domain, and the impossible case disappears from the signature instead of lurking in the body.

## 7. Mutually referential data

Two or more definitions that refer to each other. HtDP's canonical nest (§19.3) is the S-expression — the shape of every JSON document, XML tree, and AST you will ever process:

```
; An S-expr is one of:  Atom | SL
; An SL is one of:      '() | (cons S-expr SL)
; An Atom is one of:    Number | String | Symbol
```

HtDP §19.4: "Before you proceed in such situations, draw arrows to connect references to definitions", and validate the nest by constructing examples for *every* definition, starting from the clauses that reference nothing else. The design consequence: "**you must design as many functions in parallel as there are data definitions**" — each specialized to one definition, calling each other exactly where the definitions reference each other. Don't collapse them into one mega-function; the function nest mirrors the data nest arrow-for-arrow (HtDP calls this correspondence a *symmetry* — "evidence that the design recipe provides a natural way for going from problems to solutions").

## Representation and interpretation

A data definition is a bridge with two directions:

- **Representation**: information in the world → data. (The meeting's start time → epoch milliseconds, UTC.)
- **Interpretation**: data → information. (`1720000000000` → July 3 2024, 10:26 UTC.)

State both. Any data whose interpretation isn't written down will eventually be interpreted two different ways by two different functions — that is a whole category of production bugs (unit mismatches, timezone bugs, ID-of-what confusion, "is this list sorted?" disagreements).

**Invariants belong in the data definition.** "The list is sorted descending", "the balance is never negative", "the dates are half-open `[start, end)`" — write these where the data is defined, and make constructors/validators enforce them at the boundary where the data is created, so every function downstream may *rely* on them instead of re-checking or (worse) re-guessing.

## Data examples are part of the definition

Every data definition is illustrated with concrete examples of its values, constructed mechanically (§5.7): favorites for built-ins; several items for enumerations; endpoints plus an interior point for intervals; each clause separately for itemizations; "use the constructor and pick an example from the data collection named for each field" for structures; iterated construction for recursive definitions. If you can't construct an example, the definition is broken — this is the cheapest validity check in the whole discipline.

## Choosing a representation is a design act

"Data representations are rarely unique" (§6.1). For one piece of information there are always several candidate representations; the same game state can be two structure types or one structure with an optional field, "the key for you is to follow the recipe and to find a code organization that matches the data definition." Choose deliberately:

- Make **illegal states unrepresentable** where practical. If a task can't be both `completed` and `in_review`, one status field beats two booleans (`isCompleted`, `isInReview`) that admit the impossible combination.
- Prefer representations whose *shape matches the questions the program asks*. If the program constantly asks "what are this node's children", store children, not parent pointers.
- "Choose simple forms of data" (§3.6) — the simplest representation that carries the needed information wins; complexity in the data taxes every function that touches it.
- When requirements are vague, design the data first and show it — a proposed data definition is the fastest way to expose a requirements misunderstanding.
- Refine in stages. HtDP ch. 20 develops a directory tree through three successive representations (`Dir.v1` → `Dir.v3`), each deciding "which attributes to include and which to ignore," redesigning the functions at each stage. "Revising the data definition during an initial exploration is normal… As long as you stick to a systematic approach, though, changes to the data definition can naturally be propagated through the rest of the design" (§11.4).

## LLM directives

- Before writing any function, name the data definition of every parameter and of the result. If a parameter is "a dict" or "an object", you haven't finished step 0 — say which fields, which types, which invariants.
- When consuming an existing codebase, *find* the data definitions (types, schemas, models) and read them before writing code that touches them; where they're implicit (untyped dicts, JSON blobs), reconstruct and state the definition in a comment or type before proceeding.
- When you meet a magic-value encoding or a boolean explosion, treat it as a smell: propose the itemization it is hiding.
- Every clause of a data definition you write creates obligations downstream: one template branch, at least one example, at least one test. If that feels like too much work, the data definition is too complicated — simplify the representation, not the discipline.
