---
name: clean-code
description: Reference for code-level quality rules inside any layer, in any language. Covers magic numbers and booleans, cognitive complexity, guard clauses, naming, single responsibility, locality, tempered DRY, parameter count, error handling, and comments. Use when implementing a task or reviewing a diff for code quality.
---

# Clean Code Reference

This skill is the shared source of truth for the code-level quality rules that `/forge:do-work` and `/forge:review` apply. It is language-agnostic and framework-agnostic. It complements the `clean-architecture` skill. That one covers structural concerns across layers. This one covers how individual functions, variables, and expressions are written inside any layer.

## No magic numbers

Never pass a raw numeric literal into a function call, a comparison, or an assignment. Do this only when the meaning is immediately obvious from context. Numbers like `0`, `1`, `true`, and `false` are generally fine. Anything else gets a named constant.

Bad: `field.set_height(48)`
Good: `const PLAYABLE_ROWS: u32 = 48;` then `field.set_height(PLAYABLE_ROWS)`

The constant name should explain the meaning, not restate the value. Use `MAX_RETRIES`, not `THREE`. Use `HUD_ROW_TOP`, not `ROW_ZERO`. A reader should understand the code without looking up what the number means.

This also applies to string literals used as keys, identifiers, or config values. If a string appears more than once, or carries meaning beyond its face value, give it a name.

## No magic booleans

Do not pass a bare `true` or `false` as a function argument when the meaning is not obvious at the call site. Use a named constant, an enum, or a builder pattern that makes the intent clear.

Bad: `render(grid, true, false)`
Good: `render(grid, ShowBorder::Yes, ShowHud::No)`, or use named arguments if the language supports them.

## Cognitive complexity

Optimize for the total cost of understanding the code, not for any single metric. That total is the local complexity of the function you are reading. It also includes the cost of jumping to everything it calls to follow the logic. Line count alone measures neither.

The real signals of high complexity are deep nesting and deep boolean expressions. Both force a reader to hold several conditions in their head at once.

- If a function has more than three levels of nesting, that is a genuine signal. Reach for guard clauses first (see below), and extract a named helper when one earns its place.
- If a conditional has more than two or three terms joined by `&&` or `||`, extract it. Use a named boolean or a small function that describes the intent.

Bad: `if score > 25000 && !game_over && wave > 3 && lives > 0`
Good: `if is_eligible_for_bonus(state)`

Length is a weak signal, so treat it as a prompt to look, not a limit to enforce. Extract a helper when its name lets the caller be understood **without reading the helper's body**. The name fully stands in for what it does. Do not extract just to make a function shorter. A linear 40-line process you read top to bottom can be clearer. It often beats eight tiny helpers you have to reassemble in your head.

Fragmentation has its own cost. Shattering one coherent function into many small ones does not remove complexity. Understanding a single operation might now mean hopping through ten functions scattered across a file. That is the same cognitive load wearing a different outfit. Split a function when the pieces are independently meaningful, not to hit a number.

## Guard clauses and early returns

Handle edge cases, invalid input, and error conditions at the top of a function and return early. Do not wrap the main logic in a nested `if`. This keeps the happy path at the lowest indentation level, where it is easiest to read. It is the highest-leverage way to remove nesting.

Bad:

```
if user != nil {
    if user.is_active {
        // real work, two levels deep
    }
}
```

Good:

```
if user == nil { return error("no user") }
if !user.is_active { return error("inactive user") }
// real work, no nesting
```

## Naming

Names should describe what something is or does, not how it works. Use `calculate_score`, not `loop_and_add`. Use `active_players`, not `filtered_list`.

Avoid abbreviations unless they are universal in the domain. `ctx` for context is fine in Go, where it is idiomatic. `mgr` for manager is not helpful anywhere.

Boolean variables and functions should read as yes/no questions. Use `is_valid`, `has_permission`, `should_retry`. Not `valid`, `check`, `retry_flag`.

## Single responsibility at the function level

Each function should do one thing. If you can describe what a function does and the description includes the word "and," consider splitting it. This is the function-level version of the single responsibility principle.

"Calculate the score and update the leaderboard" is two things. Make them two functions.

This rule lives in tension with the locality rule below, and that is deliberate. Splitting by responsibility is worth it when the pieces are independently meaningful. It is not worth it when it scatters one coherent operation across the file. Judge by total reading cost, not by the count of functions.

## Keep related code close

Code that is read together, and changes together, should live together. Prefer ordering a file so a reader can move top to bottom and follow the flow, rather than chasing definitions around. A helper used in exactly one place, and only meaningful there, often belongs right next to its caller. Avoid moving it to a distant utilities module.

This is the counterweight to aggressive extraction. Every function you pull out adds a jump for the reader. That jump pays for itself when the name lets them skip the body. It does not pay for itself when they have to go read the body anyway to understand what the caller does. Extract for clarity, not to relocate complexity somewhere less visible.

## Don't repeat yourself, tempered

Real duplication should be pulled into one place. This applies to the same logic or rule expressed in two places that must stay in sync. When that rule changes, you want to change it once.

But a wrong abstraction costs more than a little repetition. Two pieces of code that look alike today are not always the same thing. Forcing them under one abstraction couples them. A change to one then drags the other along. Wait until the duplication is proven, and the shared concept is real, before unifying. You might find yourself adding flags or branches to a shared function so it can serve two callers that are drifting apart. That is a sign the abstraction was wrong. Prefer the duplication.

## Too many parameters

A long parameter list is a smell. It usually means a missing type that should group the related arguments. It could also mean a function is doing too much and should be split. It also makes call sites hard to read, especially when several parameters share a type and can be swapped by mistake.

When a function takes more than three or four arguments, look for a natural grouping. Bundle the ones that travel together into a struct, record, or options object whose name describes them. This reads better at the call site and survives the addition of a new field without changing every signature.

## Error handling

Handle errors at the point where you have enough context to do something meaningful. Do not swallow errors silently. Do not catch a broad exception type and ignore it. Do not return a default value when an error occurs, unless the default is explicitly documented as intentional.

If an error cannot be handled meaningfully at the current level, propagate it upward. Add context when you propagate. "Failed to save user" is better than the raw database error alone.

## Comments

Write comments that explain why, not what. The code already says what it does. If the code is too unclear to read without a "what" comment, rewrite the code first.

Good comment: `// Offset by 1 to reserve row 0 for the HUD score display`
Bad comment: `// Add 1 to y`

Do not leave commented-out code in the codebase. Delete it. Version control remembers.
