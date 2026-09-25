# Verified codebase refactor audit

## Use when
You need an evidence-based audit of code smells and architectural problems, with changes that are worth the cost.

## Prompt
You are auditing the codebase in the current working directory for code smells and architectural problems. Produce an audit the maintainers can act on. Every finding must be real, verified against the current code, and not already tracked.

Use these defaults unless the user overrides them:

- Tracker file: AUDIT.md at the repository root. Create it if it is missing. Add to it if it exists.
- Scope: all first-party source. Exclude vendored, generated, and build output, including vendor/, node_modules/, dist/, build/, lockfiles, migrations' generated SQL, and compiled assets.
- Changes allowed: the tracker file only. Do not edit source or tests, and do not commit.

Learn the project before judging it.

1. Read guidance files that exist: CLAUDE.md, AGENTS.md, CONTRIBUTING.md, the README's architecture or contributing sections, and ADRs or docs/architecture*.
2. Read linter, formatter, and type-checker configuration. Their rules, especially their ignored paths, show deliberate choices.
3. Read the dependency manifest so you know the framework and do not flag its idioms as smells.
4. Run git log --oneline -20 and git status. Trust only what you read in this session.
5. If the tracker file exists, read it in full. Do not report anything already recorded, including rejected or won't-do items.

Write down the project's deliberate conventions. Treat them as design decisions, not smells. Report code that breaks those conventions. When consistent practice implies a convention, follow it too.

Map the structure: top-level modules or packages, entry points such as HTTP routes, CLI commands, jobs, and public API, plus the rough size of each area.

Check the code against the Refactoring Guru smell catalog. Translate each smell to the paradigm in use. In functional or module-based code, class means module or closure, inheritance includes composition hierarchies, and switch statements includes branching on a type tag or discriminant.

Check for:

- Bloaters: Long Method, Large Class, Primitive Obsession, Long Parameter List, and Data Clumps.
- Object-orientation abusers: Switch Statements, Temporary Field, Refused Bequest, and Alternative Classes with Different Interfaces.
- Change preventers: Divergent Change, Shotgun Surgery, and Parallel Inheritance Hierarchies.
- Dispensables: Comments, Duplicate Code, Lazy Class, Data Class, Dead Code, and Speculative Generality.
- Couplers: Feature Envy, Inappropriate Intimacy, Message Chains, Middle Man, and Incomplete Library Class.

Also check:

- SOLID. Units with more than one reason to change. Adding a provider, format, handler, or plugin that requires editing core code. Implementations that cannot honour their contract. Interfaces that force members on implementers that do not need them. Core code that depends on concrete implementations or test doubles.
- Boundaries and layering. Dependencies that point the wrong way. Transport, framework, or vendor details leaking into domain logic. Global state and service locators where injection is available. Circular imports.
- Extension points. If the project exposes plugins, drivers, hooks, or a public API, check whether a third-party extension gets correct behaviour from defaults, rather than silent no-ops or hard-coded lists of built-in variants.
- Consistency. Sibling implementations that give the same method or concept different meanings, including null versus empty, units, ID formats, and error handling.
- Behaviour. Read the code paths end to end. Where two layers disagree, work out what happens at runtime.

Split in-scope source into slices along module boundaries, sized so each can be read in full. Use about three slices for roughly 15,000 lines and add one per further 5,000 to 10,000 lines, up to six. Give each read-only subagent the conventions, tracker coverage, this checklist, the evidence rules, and its slice's file list. Tell each to read every file in its slice in full and to read outside it only for cross-file context. A slice's findings must be rooted in that slice.

While subagents run, do not repeat their work. Verify their reports yourself:

- Open the cited lines for every P0 and P1 finding, and every claim of dead code or a runtime bug.
- Check runtime claims with a one-line program in the project's language or by reading the test that covers the path.
- Drop findings that do not hold up. Merge ones found separately, especially change preventers that span slices.

## Refactor justification

Do not recommend a refactor because code merely looks untidy or differs from an ideal pattern. A finding must show that the current design creates a real cost.

For every finding, state why the change is worth making. Use evidence from the current tree, tests, Git history, or linked tracker items.

Use one or more of these forms of proof:

- A verified bug, regression, or support issue. Name the issue, pull request, commit, or failing test. State a count only when the records support it.
- Duplicate implementations of the same rule or shape. Name every copy and explain how they can drift or already behave differently.
- A change that must touch several named files, modules, or tests. State the likely change and every place it must reach.
- A mismatch between sibling implementations that produces different results, errors, units, IDs, or empty states.
- Dead code with proof that it no longer runs and can be deleted.
- A measured cost such as repeated failures, slow work, or a large amount of code that one change would delete.

Do not infer bug counts, maintenance cost, or user harm. If the evidence does not show a concrete cost, leave the item out.

Include a finding only if all of these hold:

- It cites path:line from the current tree and names the code involved.
- It states a concrete consequence: a wrong result, a change that must touch several named places, an API callers will misuse, or code that can be deleted.
- Dead-code claims have grep proof across source, tests, examples, and docs. Allow for dynamic dispatch, reflection, framework conventions, exported public API, and string-based lookups before calling something unused.
- A duplication claim names every copy.
- A suggested fix names a Refactoring Guru technique, fits the project's conventions, and adds no abstraction the problem does not need.

Fewer certain findings are worth more than many plausible ones. If something has a sound reason, such as framework constraints, serialization, performance, public-API stability, or a rejected tracker item, leave it out. Also leave out any item whose fix only moves code around without making it simpler, unless the unit is changing for several reasons at once.

Write verified findings to the tracker file. If it already has a format, match it. Otherwise use this format:

# Code Audit

Tick an item when it lands, and note the commit next to it.

- P0: broken today, the behaviour is wrong.
- P1: misleading today, users or contributors will get it wrong.
- P2: inconsistent with siblings or other layers, or costly to change.
- P3: polish, duplication, or dead code.

## P0: Broken

- [ ] 1. <Problem>. Evidence: <paths, lines, and records>. Cost today: <specific cost>. Why refactor: <what the change removes>. Fix: <refactoring technique>.

Number items in one sequence across all sections, continuing from the highest existing number. Add Needs a decision where a maintainer must choose. Group P3 items under short subheadings such as Duplicates to remove, Dead code, Primitive values, and Splits.

Then reply in chat with:

- Finding count at each priority.
- Each P0 item in one line.
- The three changes that would delete the most code, with rough line counts.
- Anything dropped during verification, with one line on why.

The audit is finished when every slice has been read in full, every P0 and P1 finding has been checked against source, and the tracker is written. If a subagent has gaps, send it back or cover the gap before writing the file. Stop only when something blocks the work and cannot be settled from the code or project guidance.

## Notes
This prompt requires proof that a refactor removes a real problem. It does not approve refactors for style alone.
