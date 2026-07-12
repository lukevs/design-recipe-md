# Systematic software design (HtDP for LLMs)

A structured, language-agnostic distillation of Matthias Felleisen's teachings on software design — primarily *How to Design Programs* (HtDP, 2nd ed., Felleisen, Findler, Flatt, Krishnamurthi) plus the surrounding PLT pedagogy. It is written for LLMs (and humans) who write software: follow these documents as a working discipline, not background reading.

## The core claim

Most bad software — human- or LLM-written — comes from **tinkering**: guessing at code, running it, and patching symptoms until output looks right. Felleisen's answer is that program design can be a *systematic, teachable process*: every function is derived from an analysis of the data it consumes, examples are written before code, and the shape of the code is dictated by the shape of the data. When the process is followed, correctness stops being luck.

> "The typical course on programming teaches a 'tinker until it works' approach. When it works, students exclaim 'It works!' and move on. Sadly, this phrase is also the shortest lie in computing, and it has cost many people many hours of their lives." — HtDP, Preface

> "By 'good programming,' we mean an approach to the creation of software that relies on systematic thought, planning, and understanding from the very beginning, at every stage, and for every step." — HtDP, Preface

## How an LLM should use this folder

1. **Before writing any function**: apply the design recipe (`02-the-design-recipe.md`). No exceptions for "simple" functions — the recipe is cheap when the function is simple.
2. **Before writing any function, decide what the data is** (`03-data-definitions.md`). If you can't state the data definition, you don't understand the problem yet.
3. **Derive the code skeleton from the data** (`04-templates.md`), then fill it in.
4. When a problem needs many functions, **maintain a wish list** and give every function one task (`05-composing-functions.md`).
5. When two definitions look alike, **abstract deliberately** (`06-abstraction.md`).
6. When structural processing of the input can't express the algorithm, switch consciously to **generative recursion** and add a termination argument (`07-generative-recursion.md`).
7. When a recursive traversal loses knowledge of its context, add an **accumulator with a stated invariant** (`08-accumulators.md`).
8. Contain **state and mutation** behind clear boundaries with stated invariants (`09-state-and-boundaries.md`).
9. Before declaring anything done, run the **quality checklist** (`10-llm-quality-checklist.md`).

## Contents

| File | What it teaches |
|---|---|
| `01-core-philosophy.md` | Design vs. tinkering; why the process matters more than the output |
| `02-the-design-recipe.md` | The six-step recipe every function follows |
| `03-data-definitions.md` | The taxonomy of data and how data analysis drives design |
| `04-templates.md` | Mechanically deriving code structure from data structure |
| `05-composing-functions.md` | Wish lists, helper rules, one function per task |
| `06-abstraction.md` | The abstraction recipe; higher-order thinking |
| `07-generative-recursion.md` | Algorithms, the four questions, termination |
| `08-accumulators.md` | Context loss and accumulator invariants |
| `09-state-and-boundaries.md` | State, contracts, and interface discipline |
| `10-llm-quality-checklist.md` | The operational checklist to run on every piece of code |

## Sources

Primary (read directly during the preparation of these documents):

- *How to Design Programs*, 2nd edition (Felleisen, Findler, Flatt, Krishnamurthi; MIT Press) — free online edition: https://htdp.org/2024-11-6/Book/index.html — Preface, Prologue, Parts I–VI, Intermezzos, Epilogue
- "The Structure and Interpretation of the Computer Science Curriculum", *J. Functional Programming* 14(4), 2004 — https://www2.ccs.neu.edu/racket/pubs/jfp2004-fffk.pdf
- Felleisen's essays: "Developing Developers" and "The Design Recipe" — https://felleisen.org/matthias/Thoughts/
- Northeastern CS4500 Software Development course materials (code walks, interfaces, maintenance) — https://felleisen.org/matthias/
- Findler & Felleisen, "Contracts for Higher-Order Functions", ICFP 2002; Racket Guide, "Contracts and Boundaries" — https://docs.racket-lang.org/guide/contract-boundaries.html
- Friedman & Felleisen, *The Little Schemer* — the Ten Commandments

Derived:

- Gregor Kiczales, *How to Code* / Systematic Program Design (UBC CPSC 110 / edX), which operationalizes HtDP as the HtDF/HtDD/HtDW recipes and the four helper rules

All principles are language-agnostic. Examples use neutral pseudocode; everything applies equally to Python, TypeScript, Go, Rust, or any other language.
