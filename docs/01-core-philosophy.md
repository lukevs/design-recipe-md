# Core philosophy: design, not tinkering

## The failure mode this whole discipline exists to prevent

The default way programs get written — by novices, professionals under pressure, and LLMs alike — is **tinkering**: start typing code that looks plausible, run it, observe failure, patch the symptom, repeat until the output looks right. HtDP's preface opens with the diagnosis:

> "The typical course on programming teaches a 'tinker until it works' approach. When it works, students exclaim 'It works!' and move on. Sadly, this phrase is also the shortest lie in computing, and it has cost many people many hours of their lives."

HtDP calls the approach "garage programming" (ch. 3): it sometimes produces working artifacts — "sometimes it is the launching pad for a start-up company" — but the results can't be maintained, extended, or trusted, because their correctness is accidental. The summary verdict (ch. 7): **"A good programmer designs programs. A bad programmer tinkers until the program seems to work."**

In "The Structure and Interpretation of the Computer Science Curriculum" (JFP 2004), Felleisen, Findler, Flatt, and Krishnamurthi diagnose how the standard curriculum teaches **example-mimicry**: "Students copy the examples and modify them to fit the homework exercises… the teaching of (control) constructs is explicit while the teaching of design principles remains implicit; instructors leave it to the students to discover how to go from a blank screen to a full-fledged program." Even SICP fails this test: "students learn by copying and modifying code, which is barely an improvement over typical programming text books." Their alternative: make design an explicit, checkable process with intermediate products that can each be inspected for quality — and "teach good habits early; otherwise bad habits become ingrained and require costly fixes—just like bugs in programs."

A companion principle from the same paper: **working programs can be justifiably bad**. "It is critically important for students to organize programs according to measurable criteria and for teachers to be able to tell students when working programs are justifiably bad." Passing tests is necessary, never sufficient; recipe conformance, single point of control, honest names, and data-matched structure are measurable criteria that "it works" cannot substitute for.

**LLM translation.** An LLM is the world's most powerful example-mimic. Pattern-matching on millions of programs is exactly the "copy and mutate" strategy at scale. It produces plausible code, and plausible is not the same as correct. The discipline in these documents is the antidote: it forces intermediate artifacts (data definitions, signatures, purpose statements, examples) that expose misunderstanding *before* code exists — and it supplies the measurable criteria by which your own working output can still be judged bad, and improved.

## The central tenets

### 1. Design is a process with steps, and each step has a product

A designed function is not just code. It is: a data analysis, a signature, a purpose statement, worked examples, a template, code, and tests — in that order. Each artifact answers a question:

- Data definition — *what information does the program handle, and how is it represented?*
- Signature — *what does this function consume and produce?*
- Purpose statement — *what does it compute?* (not how)
- Examples — *what does correct behavior concretely look like?*
- Template — *what shape must the code have, given the data?*
- Code — *fill the template's holes.*
- Tests — *do the examples pass, mechanically?*

If you cannot produce one of these artifacts, you have found a gap in your understanding — **at the cheapest possible moment, before code exists**. This is the recipe's deepest mechanism, stated in the preface:

> "The novelty of this approach is the creation of intermediate products… When a novice is stuck, an expert or an instructor can inspect the existing intermediate products… and thus drive the novice to correct himself or herself. And this self-empowering process is the key difference between programming and program design."

For an LLM the "expert inspecting the intermediate products" is you, one step later — and the human reviewing your work. Artifacts that exist can be checked; reasoning that stayed implicit cannot.

### 2. The shape of the data determines the shape of the program

This is HtDP's single most important idea. A function that processes a list has the structure of the list data definition (empty case + first/rest case). A function over a union of three variants has a three-way case analysis. A function over a tree recurs where the tree data definition self-references. You do not invent control flow; you **read it off the data definition**. Invented control flow is where bugs live.

The JFP 2004 paper argues this is what makes software survive change: when programs match the class descriptions of their data, "small changes to the problem statement translate into small changes in the program's code. Considering the rapid changes in the requirements for real-world software, we consider this principle central to our effort." The authors invoke Brooks's old law: "Show me your code and conceal your data structures, and I shall continue to be mystified. Show me your data structures, and I won't usually need your code; it'll be obvious."

### 3. Examples come before code

Writing concrete input→output examples before coding does three things: it forces you to understand the specification concretely; it surfaces edge cases (what *does* happen on the empty list? on zero? on a missing field?) while they're still cheap; and the examples become the test suite for free. Code written before examples is a guess about a specification never stated.

### 4. Information vs. data

Programs compute with **data**; problems live in the world of **information**. Design begins by choosing a representation — how the information of the problem domain becomes data — and stating the interpretation — how to read the data back as information. Most "logic bugs" are actually representation choices nobody wrote down. Always record both directions: *representation* (information → data) and *interpretation* (data → information).

### 5. Iterative refinement

Real problems are too big to design in one pass. HtDP's preface: "iterative refinement recommends stripping away all inessential details at first and finding a solution for the remaining core problem. A refinement step adds in one of these omitted details and re-solves the expanded problem, using the existing solution as much as possible." This is refinement of the *problem*, not sloppy iteration on broken code: every iteration is a complete, working design — all recipe steps, tests passing — for a smaller problem. The book frames it as science: "a programmer is a miniscientist" who builds an approximate model, compares predictions against reality, and revises.

### 6. Programs are written for people

"Programmers write programs for other programmers to read" — where "other" "also includes older versions of the programmer who usually cannot recall all the thinking that the younger version put into the production of the program" (HtDP ch. 3). The Epilogue: "the design structure of programs is really a means of communication among programmers across time," and "unless you can describe the purpose of a piece of code with a concise statement, you cannot produce anything useful for future programmers." Every recipe artifact — signature, purpose, examples, invariants — is communication first.

### 7. The process is more valuable than the artifact

HtDP's preface argues the design recipe teaches a transferable skill: careful problem analysis, explicit assumptions, worked examples before commitment, systematic construction — "program design—but not programming—deserves the same role in a liberal-arts education as mathematics and language skills." That is also precisely the behavior that makes an LLM trustworthy on novel problems, where pattern-matching has nothing to match against.

## What this replaces

| Tinkering habit | Design discipline |
|---|---|
| Start writing code immediately | Start by writing the data definition |
| Infer behavior from the function name | Write a one-line purpose statement first |
| Test after coding, if at all | Write concrete examples before coding |
| Control flow improvised as you type | Control flow derived from the data's shape |
| One giant function that "does the task" | A wish list of single-purpose functions |
| Copy a similar-looking snippet and mutate | Abstract deliberately from two working instances |
| "It runs" = done | Tests pass, every case covered, invariants stated = done |

## Where the recipe came from

Felleisen's essay "The Design Recipe" names two origins worth knowing because they explain the discipline's spirit. From German gymnasium mathematics: teachers graded the *solution process* and "the ability to provide justifications for each step" — full marks for systematic work with an arithmetic slip, poor marks for a bare correct answer. From Quine's *Set Theory and Its Logic*: exhaust what your current axioms (here: your current data definitions and language subset) give you before adding new machinery. The recipe is process-graded mathematics imported into programming — which is why "the answer looks right" never closes a task, and "every step is justified" does.

## The one-sentence version

**Never write a line of code whose shape you cannot justify by pointing at a data definition, and never call code done whose behavior you did not predict with examples first.**
