# Systematic program design: reference index

This collection adapts *How to Design Programs* (HtDP), second edition, by Matthias Felleisen, Robert Bruce Findler, Matthew Flatt, and Shriram Krishnamurthi, for coding agents working in modern repositories.

The [skill entry point](../SKILL.md) contains the working discipline and routes to these references. Load lessons according to the task; a human reader can follow the numbered sequence.

## Lessons

| Reference | Read when |
|---|---|
| [01 — Core philosophy](01-core-philosophy.md) | Understanding inspectable design and its limits |
| [02 — Function design recipe](02-the-design-recipe.md) | Designing a function or identifying a missing design step |
| [03 — Data definitions](03-data-definitions.md) | Choosing or reconstructing representations and constraints |
| [04 — Templates](04-templates.md) | Analyzing cases, recursive structures, or multiple inputs |
| [05 — Composition](05-composing-functions.md) | Defining collaborators or intermediate representations |
| [06 — Abstraction](06-abstraction.md) | Discovering reuse or selecting an existing operation |
| [07 — Generative recursion](07-generative-recursion.md) | Generating subproblems, searching, or reasoning about progress |
| [08 — Accumulators](08-accumulators.md) | Carrying context and deriving invariant-preserving updates |
| [09 — State and boundaries](09-state-and-boundaries.md) | Designing transitions, effects, or validation responsibility |
| [10 — Quality checklist](10-llm-quality-checklist.md) | Checking an implementation or conducting a review |

## Source map

Primary source: [HtDP, second edition, online release of November 6, 2024](https://htdp.org/2024-11-6/Book/index.html).

| Topic | Book location |
|---|---|
| Intermediate products and the six-step recipe | [Preface, Figure 1](https://htdp.org/2024-11-6/Book/part_preface.html) |
| Function design, wish lists, world programs, and initial data forms | [Part I, §§3.1–3.6, Chapters 4–6](https://htdp.org/2024-11-6/Book/part_one.html) |
| Self-referential data, natural recursion, and helper design | [Part II, Chapters 8–11](https://htdp.org/2024-11-6/Book/part_two.html) |
| Abstraction from examples and templates; using abstractions | [Part III, Chapters 15–16](https://htdp.org/2024-11-6/Book/part_three.html) |
| Intertwined data, refinement, and simultaneous processing | [Part IV, Chapters 19–20 and 23](https://htdp.org/2024-11-6/Book/part_four.html) |
| Generative recipe, termination, and search | [Part V, Chapters 26 and 29](https://htdp.org/2024-11-6/Book/part_five.html) |
| Context and accumulator invariants | [Part VI, Chapters 31–34](https://htdp.org/2024-11-6/Book/part_six.html) |
| Review, debugging, and communication | [Epilogue](https://htdp.org/2024-11-6/Book/part_epilogue.html) |

Related primary source for boundary contracts: [Racket Guide, Contracts and Boundaries](https://docs.racket-lang.org/guide/contract-boundaries.html).

## Attribution and adaptation

The lessons summarize and paraphrase the design ideas; they are not quotations from, a reproduction of, or a substitute for the book and its exercises.

HtDP supplies the function recipes, data analysis, structural templates, wish lists, abstraction methods, generative recursion, accumulator reasoning, and world-state modeling.

The repository's guidance on proportional artifacts, existing-code repairs, language-specific behavior, testing methods beyond exact examples, lifecycle management, mutable ownership, transactions, and security boundaries is editorial adaptation. Racket's runtime contract system is related work, not part of HtDP's six-step function recipe.

Templates and contract examples use labeled pseudocode. The complete examples in the generative-recursion and accumulator lessons use Python. Translate their reasoning into the target language's actual APIs, types, evaluation order, and performance model.
