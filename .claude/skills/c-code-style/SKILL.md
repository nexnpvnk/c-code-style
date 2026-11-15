---
name: c-code-style
description: Write, edit, or review C (and C-compatible header) code according to C coding style rules, then check and run clang-format to enforce formatting. Trigger on "write C code", "edit a .c/.h file", "review C code style", "format C code", "run clang-format", "check clang-format", "apply the C style guide".
---

# C code style

This skill applies whenever you are writing, editing, or reviewing `.c` or `.h`
files in a project that adopts this style guide. The full rule set lives in
[`CODING_RULES.md`](../../../CODING_RULES.md) at the repository root — that
file is the single source of truth. This is a personal fork; see
[Fork-specific changes](../../../CODING_RULES.md#fork-specific-changes) for
the documented differences (this repository's `README.md` is just a
symlink to it, so it renders on the GitHub front page). This skill exists to
make the rules operational: what to self-apply while writing code, and how to
run the formatter afterward.

## 1. Read the rules first

Before writing or editing C code, read `CODING_RULES.md` in this repository
(or wherever this skill was installed alongside it — if it's not there, the
checklist in step 3 below is a self-contained fallback). Do not rely on
memory of "typical" C style — several of this guide's rules are deliberately
non-default (e.g. `(void)` on zero-parameter functions, declarations at the
top of each block, and a 3-character minimum name length).

## 2. What `clang-format` fixes for you

Don't hand-format these — write reasonably and let the formatter normalize
them in step 4: indentation and spacing, brace placement, pointer-asterisk
alignment, function layout, blank-line and parenthesis spacing. Keep return
types on the same line as function names; only empty functions may fit on
one line. Keep `case`/`default` statements on separate lines from their
bodies. Add required trailing commas in C struct initializers yourself.

## 3. What `clang-format` will NOT fix — apply these yourself while writing

These are semantic or naming rules; a formatter cannot infer them. Apply them
as you write, not after:

- Names (variables, parameters, struct/enum members) are lowercase with
  underscores and **at least 3 characters**. Never a bare `__`/`_` prefix.
  `static` module-private functions use a `prv_` prefix.
- Use `stdint.h` types (`uint8_t`, `int32_t`, ...) except `char`/`float`/`double`.
  `stdbool.h` and `bool`/`true`/`false` are allowed for boolean values.
- Ordinary initialization of `global`/`static` variables at declaration is
  allowed, including zero and `NULL`. Use a module `init()` function when
  runtime initialization or reinitialization is needed; account for custom
  embedded memory sections and startup behavior.
- Declare each variable separately, on its own line, including pointers.
  Keep all locals at the top of a block, grouped by type, in order:
  custom structs/enums (and their pointers) → integers (widest unsigned
  first) → floating point.
- `const`-correct pointers and parameters; generic pointers are `void*`, cast
  internally, never cast a `void*` return value.
- Every function with zero parameters uses `(void)`, never empty `()`.
- There is no limit on `return` points. Early returns are allowed when they
  clarify the flow. Consider a return-value variable such as `retval` when
  accumulating status or sharing cleanup; release resources on every path.
- Check a pointer against `NULL` before dereferencing it (unless the
  surrounding code guarantees it can't be `NULL`). After `free(ptr)`,
  immediately set `ptr = NULL`.
- Boolean-treated variables are compared with `if (x)` / `if (!x)`, never
  `== 1` / `== 0`; counter/value variables use `> 0` / `== 0`, never bare
  `if (x)`.
- Doxygen: every function (including `static` ones) gets a `\brief`, a
  `\param[in]`/`\param[out]` per parameter, and a `\return` if it returns
  non-`void`. The doc block goes at the function's **definition** (where the
  body is, typically the `.c` file) — never duplicated at the declaration in
  the header. Pad tags so description text starts at column 22 (see
  `template.c`/`template.h` for the exact alignment). Wrap identifiers,
  constants, and literals referenced in prose in backticks.

## 4. Format the code

After writing or editing any `.c`/`.h` file, run the formatter using this
repository's `.clang-format` (it's found automatically by walking up from the
file's directory — no path needed):

```sh
scripts/check-format.sh <changed-file.c> [<changed-file.h> ...]
```

This script:

1. Checks `clang-format` is on `PATH` and is version `20` or newer (this
   repo's minimum — see `CODING_RULES.md`, General rules).
2. If missing or too old, prints the install command for the current OS
   (`brew install llvm` on macOS, distro package or LLVM apt repo on Linux,
   the LLVM installer on Windows) and exits non-zero — install it, then
   re-run.
3. Otherwise runs `clang-format -i` on the given files.

To verify compliance without writing changes (e.g. before reporting a task
done), use `--check` instead:

```sh
scripts/check-format.sh --check <file.c> [<file.h> ...]
```

Exits `0` if already compliant, non-zero (with the specific violations) if
not.

## 5. Verified behavior

This flow was tested directly in this repository: `check-format.sh` correctly
refuses to run when `clang-format` is absent (prints the right install
command per OS and exits 1), and — once `clang-format` is installed —
correctly detects a deliberately mis-formatted file with `--check`, fixes it
with the default (in-place) mode, and re-verifies clean afterward.
`template.c` and `template.h` in this repository both pass `--check` as-is —
use them as a reference for exact formatting and doxygen alignment.
