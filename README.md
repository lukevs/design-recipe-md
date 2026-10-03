# HtDP design recipes for coding agents

A language-agnostic skill and reference collection based on *How to Design Programs*, second edition, by Matthias Felleisen, Robert Bruce Findler, Matthew Flatt, and Shriram Krishnamurthi.

The focus is systematic design: define and interpret the data, state the behavior, work examples, choose a structure, implement, and verify. The guidance covers composition, abstraction, generative recursion, accumulators, and state boundaries, with adaptations for existing codebases.

## Use

[SKILL.md](SKILL.md) is the skill entry point. Keep it and `docs/` together when installing the folder as `design-recipe` in your coding agent's skill directory. Invoke it as `$design-recipe`, or ask your agent to apply the skill in this repository. This repository does not install itself or modify agent configuration.

Example requests:

- “Use the design recipe to implement this tree transformation.”
- “Review this parser for missing cases and unclear data assumptions.”
- “Refactor this traversal and explain its accumulator invariant.”

For human reading, start with the [reference index and source map](docs/README.md). The ten lessons are supporting references for one skill, rather than ten overlapping skill triggers.

The skill scales the written design to the task. Existing types and tests can already supply recipe artifacts; a small fix does not require a new design document. Passing tests provide evidence, and algorithm invariants and termination arguments make that evidence easier to assess.
