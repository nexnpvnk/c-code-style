# Agent instructions

This repository defines a personal fork of Tilen MAJERLE's C coding style.
See [Fork-specific changes](CODING_RULES.md#fork-specific-changes) for the
documented differences. If you (an AI coding agent) are
writing, editing, or reviewing `.c`/`.h` files in a project that adopts this
style guide, follow this file.

## Source of truth

Read [`CODING_RULES.md`](CODING_RULES.md) before writing code — it is the authoritative,
complete rule set. Do not assume "typical" C conventions; several rules here
are deliberately non-default (e.g. `(void)` required on zero-parameter
functions, declarations at the top of each block, and a 3-character minimum
name length). `template.c` and `template.h` are
worked, clang-format-verified reference examples — match their formatting
and doxygen alignment exactly.

## What the formatter fixes for you

`clang-format` (this repo ships `.clang-format`) handles indentation,
spacing, brace placement, pointer-asterisk alignment, and function layout.
Keep the return type on the same line as the function name; only empty
functions may fit on one line. Keep `case`/`default` statements on separate
lines from their bodies. Write reasonably and run the formatter afterward
(see below). Add required trailing commas in C struct initializers yourself.

## What you must apply yourself while writing

A formatter cannot infer these; apply them as you write, not after:

- Names (variables, parameters, struct/enum members): lowercase, underscore
  word-separated, **at least 3 characters**. No `__`/`_` prefix (reserved).
  `static` module-private functions get a `prv_` prefix.
- `stdint.h` types except `char`/`float`/`double`; `stdbool.h` and
  `bool`/`true`/`false` are allowed for boolean values.
- Ordinary initialization of `global`/`static` variables at declaration is
  allowed, including zero and `NULL`. Use a module `init()` function when
  runtime initialization or reinitialization is needed; account for custom
  embedded memory sections and startup behavior.
- Declare each variable separately, on its own line, including pointers.
  All locals stay at the top of a block, grouped by type, ordered:
  structs/enums (and pointers) → integers (widest unsigned first) → floating
  point.
- `const`-correct pointers/parameters; generic pointers are `void*`, cast
  internally, never cast away a `void*` return value.
- Every zero-parameter function uses `(void)`, never empty `()`.
- There is no limit on `return` points. Early returns are allowed when they
  clarify the flow. Consider a return-value variable such as `retval` when
  accumulating status or sharing cleanup; release resources on every path.
- Check pointers against `NULL` before dereferencing (unless provably
  non-`NULL` by construction). Set a pointer to `NULL` immediately after
  `free()`.
- Boolean-treated variables: `if (x)` / `if (!x)`, never `== 1`/`== 0`.
  Counter/value variables: `> 0` / `== 0`, never bare `if (x)`.
- Doxygen on every function (including `static`): `\brief`, `\param[in]` /
  `\param[out]` per parameter, `\return` if non-`void`. Written at the
  function's **definition** (the `.c` file body), never duplicated at the
  header declaration. Description text aligned to column 22 (match
  `template.c`). Backtick any identifier/constant/literal in prose.

## Format and verify

After editing any `.c`/`.h` file, run:

```sh
scripts/check-format.sh <changed-file.c> [<changed-file.h> ...]
```

It checks `clang-format` is installed (>= version 20; prints the right
install command for your OS and exits non-zero if not, or if too old), then
runs `clang-format -i` using this repo's `.clang-format`.

To verify without writing changes:

```sh
scripts/check-format.sh --check <file.c> [<file.h> ...]
```

Exits `0` if compliant, non-zero with the specific violations otherwise. This
script and flow were verified directly in this repository (missing-tool path,
detect → fix → re-verify round trip, and both template files passing
`--check` as-is).
