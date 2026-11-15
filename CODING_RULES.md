# Recommended C style and coding rules

This document describes the C code style maintained in this personal fork, based on [Tilen MAJERLE's C style guide](https://github.com/MaJerle/c-code-style). It is the recommended guideline for all contributions and can be integrated with AI coding agents and enforced automatically using clang-format.

> **Fork notice:** This is a personal fork maintained by [nexnpvnk](https://github.com/nexnpvnk/c-code-style), with custom rules and formatter settings. The [fork-specific changes](#fork-specific-changes) below link directly to each modified rule. Original authorship and license notices are retained.

> To adopt these rules in your own project, copy **this file** (`CODING_RULES.md`) into your repository — not this repository's `README.md`, which is only a symlink to this file so it renders on the GitHub front page. Copying `CODING_RULES.md` won't overwrite your project's own `README.md`.

## Table of Contents

- [Recommended C style and coding rules](#recommended-c-style-and-coding-rules)
  - [Table of Contents](#table-of-contents)
  - [Fork-specific changes](#fork-specific-changes)
  - [The single most important rule](#the-single-most-important-rule)
  - [Integration with VSCode](#integration-with-vscode)
  - [Conventions used](#conventions-used)
  - [General rules](#general-rules)
  - [Comments](#comments)
  - [Functions](#functions)
  - [Variables](#variables)
  - [Structures, enumerations, typedefs](#structures-enumerations-typedefs)
  - [Compound statements](#compound-statements)
    - [Switch statement](#switch-statement)
  - [Macros and preprocessor directives](#macros-and-preprocessor-directives)
  - [Documentation](#documentation)
  - [Header/source files](#headersource-files)
  - [Clang format integration](#clang-format-integration)
    - [Formatting scope and wrapping](#formatting-scope-and-wrapping)
    - [Continuous integration (GitHub Actions)](#continuous-integration-github-actions)

## Fork-specific changes

The following intentional changes distinguish this fork from the upstream style:

| Change | Rule |
| --- | --- |
| Declare each variable separately, on its own line; keep locals at the beginning of each block and preserve their type order | [Variable declarations](#fork-variable-declarations) |
| Allow `stdbool.h`, `bool`, `true`, and `false` | [Boolean values](#fork-boolean-values) |
| Remove the limit on `return` points; retain the recommendation to consider `retval` for accumulated status or shared cleanup | [Return points](#fork-return-points) |
| Allow ordinary initialization of global and static variables at declaration | [Static and global initialization](#fork-static-initialization) |
| Keep the return type and any storage-class or qualifier keywords on the same line as the function name | [Function signatures](#fork-function-signatures) |
| Keep `case`/`default` labels on separate lines from their bodies | [Switch cases](#fork-switch-cases) |
| Expand every non-empty function body across multiple lines; allow empty functions on one line | [Short functions](#fork-short-functions) |
| Reformat only project-authored files; preserve the formatting of imported and adapted code | [Formatting scope](#fork-formatting-scope) |
| Require LLVM 23 or newer; use 120 columns, four-space indentation, and one argument or parameter per line in wrapped calls and signatures | [Line wrapping](#fork-line-wrapping) |
| Keep logical operators at line ends and avoid splitting string literals | [Line wrapping](#fork-line-wrapping) |

These changes are reflected in `.clang-format`, `AGENTS.md`, the Claude Code skill, and the reference templates where applicable. Rules about other integer types, documentation placement, macros, and memory management retain their existing requirements.

## The single most important rule

Let's start with a quote from a GNOME developer.

> The single most important rule when writing code is this: *check the surrounding code and try to imitate it*.
>
> As a maintainer it is dismaying to receive a patch that is obviously in a different coding style to the surrounding code. This is disrespectful, like someone tromping into a spotlessly-clean house with muddy shoes.
>
> So, whatever this document recommends, if there is already existing code and you are patching it, keep its current style consistent even if it is not your favorite style.

## Integration with VSCode

VSCode can use the `clang-format` tool (part of the LLVM package) to automatically format code during development, but `clang-format` is not limited to VSCode. It is a standalone formatting tool that can be used from any editor or from the command line.

When a file is saved, VSCode can invoke `clang-format` and format the file. The formatting rules are provided by a `.clang-format` file. If `clang-format` cannot find this file in the current directory, it searches upward until it finds one. If no `.clang-format` file is found, clang-format uses its default rules.

This repository always contains an up-to-date `.clang-format` file that matches the rules described here. You can place this file in your repository root or in a shared project root for multiple repositories.

Some configuration options should be enabled:
![VSCode configuration](images/vscode-settings.png)

> Note: this repository ships a Claude Code skill (`.claude/skills/c-code-style/`) and an `AGENTS.md` that check `clang-format` is available and run it on relevant files using the repository `.clang-format` rules — see the skill's `SKILL.md` for details.

## Conventions used

The keywords *MUST*, *MUST NOT*, *REQUIRED*, *SHALL*, *SHALL NOT*, *SHOULD*, *SHOULD NOT*, *RECOMMENDED*, *NOT RECOMMENDED*, *MAY*, and *OPTIONAL* in this document are interpreted as described in BCP 14 [RFC2119] [RFC8174].

## General rules

These are the most obvious and important general rules. Please review them carefully before continuing.

- `clang-format` SHOULD be used with the `.clang-format` file included in this repository (version `23.x` or later is required). Use the same LLVM version for `clang-format` and `clangd` when both are installed.
- Use the `C11` standard
- Do not use tabs; use spaces instead
- Use `4` spaces per indent level
- Use `1` space between a keyword and its opening parenthesis
```c
/* OK */
if (condition)
while (condition)
for (init; condition; step)
do {} while (condition)

/* Wrong */
if(condition)
while(condition)
for(init;condition;step)
do {} while(condition)
```

- Do not use space between function name and opening bracket
```c
int32_t sum_result = sum(4, 3);              /* OK */
int32_t sum_result = sum (4, 3);             /* Wrong */
```

- Never use `__` or `_` prefix for variables/functions/macros/types. This is reserved for C language itself
    - Prefer `prv_` name prefix for strictly module-private (static) functions
    - Prefer `libname_int_` or `libnamei_` prefix for library internal functions, that should not be used by the user application while they MUST be used across different library internal modules
- Use only lowercase characters for variables/functions/types with optional underscore `_` char
- Variable names SHOULD be at least `3` characters long
- Opening curly bracket is always at the same line as keyword (`for`, `while`, `do`, `switch`, `if`, ...)
```c
size_t idx;
for (idx = 0; idx < 5; ++idx) {           /* OK */
}
for (idx = 0; idx < 5; ++idx){            /* Wrong */
}
for (idx = 0; idx < 5; ++idx)             /* Wrong */
{
}
```

- Use single space before and after comparison and assignment operators
```c
int32_t var_value;
var_value = 3 + 4;              /* OK */
for (var_value = 0; var_value < 5; ++var_value) /* OK */
var_value=3+4;                  /* Wrong */
var_value = 3+4;                /* Wrong */
for (var_value=0;var_value<5;++var_value)       /* Wrong */
```

- Use single space after every comma
```c
func_name(5, 4);        /* OK */
func_name(4,3);         /* Wrong */
```

<a id="fork-static-initialization"></a>

- Ordinary initialization of `global` and `static` variables at declaration is allowed, including zero and `NULL`. Use a dedicated `init` function when runtime initialization or reinitialization is needed.
```c
static int32_t global_var_a;            /* OK, implicit zero initialization */
static int32_t global_var_b = 4;        /* OK, explicit startup value */
static int32_t global_var_c = 0;        /* OK, explicit zero initialization */
static const void* global_data = NULL; /* OK, explicit null pointer initialization */
```
> For custom embedded memory sections or nonstandard startup code, ensure that the startup process and linker script provide the intended initial values. If a section is intentionally not initialized, document and implement its initialization separately.

<a id="fork-variable-declarations"></a>

- Declare each variable separately, on its own line, even when several variables have the same type. Keep local declarations and their initializers at the beginning of the block, before the first executable statement.
```c
/* OK */
void my_func(void) {
    char var_char1;
    char var_char2;
}

/* Wrong, multiple variables in one declaration */
void my_func(void) {
    char var_char1, var_char2;
}
```

- Declare local variables in order
    1. Custom structures and enumerations
    2. Integer types, wider unsigned type first
    3. Single/Double floating point
```c
int my_func(void) {
    /* 1 */
    my_struct_t my_struct;     /* First custom structures */
    my_struct_ptr_t* my_struct_ptr; /* Pointers too */

    /* 2 */
    uint32_t var_u32;
    int32_t var_i32;
    uint16_t var_u16;
    int16_t var_i16;
    char var_char;
    /* ... */

    /* 3 */
    double var_dbl;
    float var_flt;
}
```

- Always declare local variables at the beginning of the block, before first executable statement
- Always add trailing comma in the last element of structure (or its children) initialization (this helps clang-format to properly format structures). Unless structure is very simple and short
```c
typedef struct {
    int val_a;
    int val_b;
} str_t;

str_t item = {
    .val_a = 1,
    .val_b = 2,   /* Comma here */
};

/* Examples of "complex" structure, with or with missing several trailing commas, after clang-format runs the formatting */
static const my_struct_t my_var_1 = {
    .type = TYPE1,
    .type_data =
        {
            .type1 =
                {
                    .par1 = 0,
                    .par2 = 1, /* Trailing comma here */
                }, /* Trailing comma here */
        },  /* Trailing comma here */
};

static const my_struct_t my_var_2 = {.type = TYPE2,
                                     .type_data = {
                                         .type2 =
                                             {
                                                 .par1 = 0,
                                                 .par2 = 1,
                                             },
                                     }};    /* Missing comma here */
static const my_struct_t my_var_3 = {.type = TYPE3,
                                     .type_data = {.type3 = {
                                                       .par1 = 0,
                                                       .par2 = 1,
                                                   }}}; /* Missing 2 commas here */

/* No trailing commas - good only for small and simple structures */
static const my_struct_t my_var_4 = {.type = TYPE4, .type_data = {.type4 = {.par1 = 0, .par2 = 1}}};
```

- Declare counter variables in `for` loops with a descriptive name
```c
/* OK */
for (size_t idx = 0; idx < 10; ++idx)

/* OK, if you need the counter variable later */
size_t idx;
for (idx = 0; idx < 10; ++idx) {
    if (...) {
        break;
    }
}
if (idx == 10) {

}

/* Wrong */
size_t idx;
for (idx = 0; idx < 10; ++idx) ...
```

- A single-variable declaration may initialize the variable with a function call. Keep all declarations and initializers at the beginning of the block and declare each variable separately.
```c
/* Wrong, multiple variables in one declaration */
void calculate(void) {
    int32_t var_value, result = sum(1, 2);
}

/* OK, one declaration per variable, all at the top of the block */
void calculate(void) {
    int32_t var_value;
    int32_t result = sum(1, 2);
    uint8_t status = 3;
    uint8_t flags = 4;

    var_value = result;
}
```

- Except for `char`, `float`, `double`, or `bool`, always prefer types declared in `stdint.h`, e.g. `uint8_t` for `unsigned 8-bit`.

<a id="fork-boolean-values"></a>

- The `stdbool.h` header and its `bool`, `true`, and `false` names are allowed for boolean values. Integer boolean flags using `1` and `0` remain allowed.
```c
#include <stdbool.h>

/* OK, boolean type and named values */
bool is_ready = false;
bool status = true;

/* Also allowed, integer boolean flag */
uint8_t status_flag = 0;
```

- Never compare against `true`, eg. `if (check_func() == 1)`, use `if (check_func()) { ... }`
- Always compare pointers against `NULL` value
```c
void* ptr;

/* ... */

/* OK, compare against NULL */
if (ptr == NULL || ptr != NULL) {

}

/* Wrong */
if (ptr || !ptr) {

}
```

- Always check a pointer against `NULL` before dereferencing it, unless the surrounding code guarantees, by construction, that it cannot be `NULL` at that point
```c
/* OK */
void process(int32_t* ptr) {
    if (ptr == NULL) {
        return; /* Guard against NULL before use */
    }
    *ptr = 10; /* Safe to dereference */
}

/* Wrong, ptr is dereferenced without a NULL check */
void process(int32_t* ptr) {
    *ptr = 10;
}
```

- Prefer the use of *pre-increment (and decrement respectively)* instead of *post-increment (and decrement respectively)* whenever possible. Depending on the semantics, *post-increment* is REQUIRED.
```c
int32_t counter_value = 0;
...
counter_value++;            /*  */
++counter_value;            /* Preferred */

for (size_t idx = 0; idx < 10; ++idx) {}  /* OK */
```

- Always use `size_t` for length or size variables
- Always use `const` for pointer when function should not modify memory pointed to by `pointer`
- Always use `const` for function parameters or variables if they should not be modified
```c

/* When the pointer itself can be modified, but the data it points to cannot */
void my_func(const void* data) {

}

/* When neither the pointer nor the data it points to can be modified */
void my_func(const void* const data) {

}

/* Not REQUIRED, but advised */
void my_func(const size_t length) {

}

/* When the pointer cannot be reassigned, but the data it points to can be modified */
void my_func(void* const data) {

}
```

- When function may accept pointer of any type, always use `void *`, do not use `uint8_t *`
    - Function MUST take care of proper casting in the implementation
```c
/*
 * To send data, function should not modify memory pointed to by `data` variable
 * thus `const` keyword is important
 *
 * To send generic data (or to write them to file)
 * any type may be passed for data,
 * thus use `void *`
 */
/* OK example */
void send_data(const void* data, size_t len) { /* OK */
    /* Do not cast `void *` or `const void *` */
    const uint8_t* d_ptr = data; /* Function handles proper type for internal usage */
}

void send_data(const void* data, int len) {    /* Wrong, do not use int */
}
```

- Always use brackets with `sizeof` operator
- Never use *Variable Length Array* (VLA). Use dynamic memory allocation instead with standard C `malloc` and `free` functions or if library/project provides custom memory allocation, use its implementation
    - Take a look at [LwMEM](https://github.com/MaJerle/lwmem), custom memory management library
- After a pointer is passed to `free`, immediately set it to `NULL` to avoid dangling-pointer and double-free bugs
```c
/* OK */
#include <stdlib.h>
void my_func(size_t size) {
    int32_t* arr;
    arr = malloc(sizeof(*arr) * size); /* OK, Allocate memory */
    arr = malloc(sizeof *arr * size);  /* Wrong, brackets for sizeof operator are missing */
    if (arr == NULL) {
        /* FAIL, no memory */
    }

    free(arr);  /* Free memory after usage */
    arr = NULL; /* OK, avoid dangling pointer after free */
}

/* Wrong */
void my_func(size_t size) {
    int32_t arr[size];  /* Wrong, do not use VLA */
}
```

- Always compare variable against zero, except if it is treated as `boolean` type
- Never compare `boolean-treated` variables against zero or one. Use NOT (`!`) instead
```c
size_t length = 5;  /* Counter variable */
bool is_ok = false; /* Boolean variable, with `stdbool.h` included */
if (length)         /* Wrong, length is not treated as boolean */
if (length > 0)     /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */
if (length == 0)    /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */

if (is_ok)          /* OK, variable is treated as boolean */
if (!is_ok)         /* OK, -||- */
if (is_ok == 1)     /* Wrong, never compare boolean variable against 1! */
if (is_ok == 0)     /* Wrong, use ! for negative check */
```

- Always use `/* comment */` for comments, even for *single-line* comment
- Always include check for `C++` with `extern` keyword in header file
- Every function MUST include *doxygen-enabled* comment, even if function is `static`
- Use English names/text for functions, variables, comments
- Use *lowercase* characters for variables
- Use *underscore* if variable contains multiple names, eg. `force_redraw`. Do not use `forceRedraw`
- Never cast function returning `void *`, eg. `uint8_t* ptr = (uint8_t *)func_returning_void_ptr();` as `void *` is safely promoted to any other pointer type
    - Use `uint8_t* ptr = func_returning_void_ptr();` instead
- Always use `<` and `>` for C Standard Library include files, eg. `#include <stdlib.h>`
- Always use `""` for custom libraries, eg. `#include "my_library.h"`
- When casting to pointer type, always align asterisk to type, eg. `uint8_t* val = (uint8_t*)var_width_diff_type`
- Always respect code style already used in project or library

## Comments

- Comments starting with `//` are not allowed. Always use `/* comment */`, even for single-line comment
```c
//This is comment (wrong)
/* This is comment (ok) */
```

- For multi-line comments use `space+asterisk` for every line
```c
/*
 * This is multi-line comments,
 * written in 2 lines (ok)
 */

/**
 * Wrong, use double-asterisk only for doxygen documentation
 */

/*
* Single line comment without space before asterisk (wrong)
*/

/*
 * Single line comment in multi-line configuration (wrong)
 */

/* Single line comment (ok) */
```

- Use `12` indents (`12 * 4` spaces) offset when commenting. If statement is larger than `12` indents, make comment `4-spaces` aligned (examples below) to next available indent
```c
void my_func(void) {
    char val_a;
    char val_b;

    val_a = call_func_returning_char_a(val_a);          /* This is comment with 12*4 spaces indent from beginning of line */
    val_b = call_func_returning_char_a_but_func_name_is_very_long(val_a);   /* This is comment, aligned to 4-spaces indent */
}
```

## Functions

- Every function that may be accessed from outside its module MUST include a function prototype (declaration).
- Function declarations and definitions that accept no parameters MUST explicitly use `(void)`, never empty parentheses `()`
```c
/* OK */
int32_t get_status(void);

/* Wrong, in C this declares a function accepting an unspecified number of parameters, not zero parameters */
int32_t get_status();
```
- Function names MUST be lowercase, optionally separated with underscore `_` characters
```c
/* OK */
void my_func(void);
void myfunc(void);

/* Wrong */
void MYFunc(void);
void myFunc(void);
```

- When a function returns a pointer, align the asterisk with the return type
```c
/* OK */
const char* my_func(void);
my_struct_t* my_func(int32_t val_a, int32_t val_b);

/* Wrong */
const char *my_func(void);
my_struct_t * my_func(void);
```
- Align similar function prototypes for better readability
```c
/* OK */
void        set(int32_t val);
my_type_t   get(void);
my_ptr_t*   get_ptr(void);

/* Wrong */
void set(int32_t val);
const char * get(void);
```

<a id="fork-function-signatures"></a>

- Function declarations and definitions MUST keep the return type and any storage-class or qualifier keywords on the same line as the function name. Wrap long parameter lists rather than placing the return type on a separate line.
```c
/* OK */
int32_t foo(void) {
    return 0;
}

/* OK */
static const char* get_string(void) {
    return "Hello world!\r\n";
}

/* Wrong */
int32_t
foo(void) {
    return 0;
}
```

<a id="fork-short-functions"></a>

- Non-empty function bodies MUST span multiple lines, even if they contain only one statement. Empty functions MAY be written on one line.
```c
/* OK */
int32_t get_default(void) {
    return 0;
}

/* OK, empty function */
void my_hook(void) {}

/* Wrong, non-empty function on one line */
int32_t get_default(void) { return 0; }
```

<a id="fork-return-points"></a>

- There is no limit on the number of `return` points. Early returns, including returns in the middle of a function, are allowed when they make the control flow clearer. Consider a return-value variable such as `retval` when accumulating status or sharing cleanup. Ensure that every return path releases any resources owned by the function.
```c
/* OK, single early-return for parameter check, single return at the end */
int32_t my_func(const void* data, size_t len) {
    if (data == NULL || len == 0) {
        return -1; /* Early return, parameter check */
    }

    /* ... function body ... */

    return 0; /* Single return at the end */
}

/* OK, an additional early return makes the error path explicit */
int32_t my_func(const void* data, size_t len) {
    if (data == NULL || len == 0) {
        return -1;
    }

    /* ... */
    if (some_condition) {
        return -2; /* OK, return directly when appropriate */
    }
    /* ... */

    return 0;
}

/* Also recommended when useful: accumulate status in `retval` */
int32_t my_func(const void* data, size_t len) {
    int32_t retval = 0;

    if (data == NULL || len == 0) {
        return -1;
    }

    /* ... */
    if (some_condition) {
        retval = -2; /* OK, store the result instead of returning here */
    }

    if (retval == 0) {
        /* Do something else */
        retval = some_function(some_parameter);
    }
    if (retval == 0) {
        /* Do something third */
    }
    /* ... */
    return retval;
}
```

## Variables

- Make variable name all lowercase with optional underscore `_` character
```c
/* OK */
int32_t val;
int32_t my_var;
int32_t myvar;

/* Wrong */
int32_t VAL;
int32_t myVar;
int32_t MYVar;
```

- Group local variables together by `type`, preserving the order defined in [General rules](#general-rules), but declare each variable separately on its own line
```c
/* OK */
void foo(void) {
    int32_t val_a;
    int32_t val_b;
    char chr_a;
    char chr_b;
}

/* Wrong, multiple variables in one declaration */
void foo(void) {
    int32_t val_a, val_b;
}
```

- Do not declare variable after first executable statement
```c
void foo(void) {
    int32_t val_a;
    val_a = bar();
    int32_t val_b;      /* Wrong, there is already executable statement */
}
```

- You may declare new variables inside next indent level
```c
int32_t val_a;
int32_t val_b;
val_a = foo();
if (val_a) {
    int32_t val_c;         /* OK, declaration at the beginning of the nested block */
    int32_t val_d;
    val_c = foo();
    int32_t val_e;      /* Wrong, there was already executable statement inside block */
}
```

- Declare pointer variables with asterisk aligned to type
```c
/* OK */
char* txt;

/* Wrong */
char *txt;
char * txt;
```

- Declare each pointer variable separately and keep the asterisk aligned to its type
```c
/* OK */
char* cur;
char* nxt;

/* Wrong, multiple pointer variables in one declaration */
char *cur, *nxt;
```

## Structures, enumerations, typedefs

- Structure or enumeration name MUST be lowercase with optional underscore `_` character between words
- Structure or enumeration may contain `typedef` keyword
- All structure members MUST be lowercase
- All enumeration members SHOULD be uppercase
- Structure/enumeration MUST follow doxygen documentation syntax

When structure is declared, it may use one of `3` different options:

1. When structure is declared with *name only*, it *MUST not* contain `_t` suffix after its name.
```c
struct struct_name {
    char* txt;
    char val;
};
```
2. When structure is declared with *typedef only*, it *has to* contain `_t` suffix after its name.
```c
typedef struct {
    char* txt;
    char val;
} struct_name_t;
```
3. When structure is declared with *name and typedef*, it *MUST NOT* contain `_t` for basic name and it *MUST* contain `_t` suffix after its name for typedef part.
```c
typedef struct struct_name {    /* No _t */
    char* txt;
    char val_a;
    char val_b;
} struct_name_t;    /* _t */
```

Examples of bad declarations and their suggested corrections
```c
/* val_a and val_b MUST be separated to 2 lines */
/* Name of structure with typedef MUST include _t suffix */
typedef struct {
    int32_t val_a, val_b;
} point;

/* Corrected version */
typedef struct {
    int32_t val_a;
    int32_t val_b;
} point_t;

/* Wrong name, it MUST not include _t suffix */
struct coord_t {
    int32_t val_a;
    int32_t val_b;
};

/* Wrong parameters, MUST be all uppercase */
typedef enum {
    MY_ENUM_TESTA,
    my_enum_testb,
} my_enum_t;
```

- When initializing structure on declaration, use `C99` initialization style
```c
/* OK */
point_t point = {
    .val_a = 4,
    .val_b = 5,
};

/* Wrong */
point_t point = {1, 2};
```

- When new typedef is introduced for function handles, use `_fn` suffix
```c
/* Function accepts 2 parameters and returns uint8_t */
/* Name of typedef has `_fn` suffix */
typedef uint8_t (*my_func_typedef_fn)(uint8_t par_a, const char* par_b);
```

## Compound statements

- Every compound statement MUST include opening and closing curly bracket, even if it includes only `1` nested statement
- Every compound statement MUST include single indent; when nesting statements, include `1` indent size for each nest
```c
/* OK */
if (cnd) {
    do_a();
} else {
    do_b();
}

/* Wrong */
if (cnd)
    do_a();
else
    do_b();

/* Wrong */
if (cnd) do_a();
else do_b();
```

- In case of `if` or `if-else-if` statement, `else` MUST be in the same line as closing bracket of first statement
```c
/* OK */
if (cnd_a) {

} else if (cnd_b) {

} else {

}

/* Wrong */
if (cnd_a) {

}
else {

}

/* Wrong */
if (cnd_a) {

}
else
{

}
```

- In case of `do-while` statement, `while` part MUST be in the same line as closing bracket of `do` part
```c
/* OK */
do {
    int32_t val;
    val = do_a();
    do_b(val);
} while (check());

/* Wrong */
do
{
/* ... */
} while (check());

/* Wrong */
do {
/* ... */
}
while (check());
```

- Indentation is REQUIRED for every opening bracket
```c
if (cnd_a) {
    do_a();
} else {
    do_b();
    if (cnd_b) {
        do_c();
    }
}
```

- Compound statement MUST include curly brackets, even in the case of a single statement. Examples below show bad practices
```c
if (cnd_a) do_b();
else do_c();

if (cnd_a) do_a(); else do_b();
```

- Empty `while`, `do-while` or `for` loops MUST include brackets
```c
/* OK */
while (is_register_bit_set()) {}

/* Wrong */
while (is_register_bit_set());
while (is_register_bit_set()) { }
while (is_register_bit_set()) {
}
```

- If `while` (or `for`, `do-while`, etc) is empty (it can be the case in embedded programming), use empty single-line brackets
```c
/* Wait for bit to be set in embedded hardware unit */
volatile uint32_t* addr = HW_PERIPH_REGISTER_ADDR;

/* Wait bit 13 to be ready */
while (*addr & (1 << 13)) {}        /* OK, empty loop contains no spaces inside curly brackets */
while (*addr & (1 << 13)) { }       /* Wrong */
while (*addr & (1 << 13)) {         /* Wrong */

}
while (*addr & (1 << 13));          /* Wrong, curly brackets are missing. Can lead to compiler warnings or unintentional bugs */
```
- Always prefer using loops in this order: `for`, `do-while`, `while`
- Avoid incrementing variables inside loop block if possible, see examples

```c
/* Not recommended */
int32_t cnt = 0;
while (cnt < 10) {
    .
    ..
    ...
    ++cnt;
}

/* Better */
for (size_t cnt = 0; cnt < 10; ++cnt) {

}

/* Better, if inc may not happen in every cycle */
for (size_t cnt = 0; cnt < 10; ) {
    if (...) {
        ++cnt;
    }
}
```

- Inline `if` statement MAY be used only for assignment or function call operations
```c
/* OK */
int val = condition ? if_yes : if_no; /* Assignment */
func_call(condition ? if_yes : if_no); /* Function call */
switch (condition ? if_yes : if_no) {...}   /* OK */

/* Wrong, this code is not well maintenable */
condition ? call_to_function_a() : call_to_function_b();

/* Rework to have better program flow */
if (condition) {
    call_to_function_a();
} else {
    call_to_function_b();
}
```

### Switch statement

<a id="fork-switch-cases"></a>

- `case` and `default` labels MUST be on separate lines from their bodies, even when the body contains only one statement. Never collapse a case into a single line. An opening block brace MAY remain on the label line as shown below.
```c
/* OK */
switch (value) {
    case 0:
        break;
    default:
        handle_value();
        break;
}

/* Wrong, case and default bodies share their label lines */
switch (value) {
    case 0: break;
    default: handle_value(); break;
}
```

- Add *single indent* for every `case` statement
- Use additional *single indent* for `break` statement in each `case` or `default` statement
```c
/* OK, every case has single indent */
/* OK, every break has additional indent */
switch (check()) {
    case 0:
        do_a();
        break;
    case 1:
        do_b();
        break;
    default:
        break;
}

/* Wrong, case indent missing */
switch (check()) {
case 0:
    do_a();
    break;
case 1:
    do_b();
    break;
default:
    break;
}

/* Wrong */
switch (check()) {
    case 0:
        do_a();
    break;      /* Wrong, break MUST have indent as it is under case */
    case 1:
    do_b();     /* Wrong, indent under case is missing */
    break;
    default:
        break;
}
```

- Always include `default` statement
```c
/* OK */
switch (var) {
    case 0:
        do_job();
        break;
    default:
        break;
}

/* Wrong, default is missing */
switch (var) {
    case 0:
        do_job();
        break;
}
```

- If local variables are REQUIRED, use curly brackets and put `break` statement inside.
    - Put opening curly bracket in the same line as `case` statement
```c
switch (sel) {
    /* OK */
    case 0: {
        int32_t val_a;
        int32_t val_b;
        char chr;
        val_a = 5;
        /* ... */
        break;
    }

    /* Wrong */
    case 1:
    {
        int32_t val;
        break;
    }

    /* Wrong, break shall be inside */
    case 2: {
        int32_t val;
    }
    break;
}
```

## Macros and preprocessor directives

- Always use macros instead of literal constants, especially for numbers
- All macros MUST be fully uppercase, with optional underscore `_` character, except if they are clearly marked as function which may be in the future replaced with regular function syntax
```c
/* OK */
#define SQUARE(val)         ((val) * (val))

/* Wrong */
#define square(val)           ((val) * (val))
```

- Always protect input parameters with parentheses
```c
/* OK */
#define MIN(val_a, val_b)           ((val_a) < (val_b) ? (val_a) : (val_b))

/* Wrong */
#define MIN(val_a, val_b)           val_a < val_b ? val_a : val_b
```

- Always protect final macro evaluation with parenthesis
```c
/* Wrong */
#define MIN(val_a, val_b)           (val_a) < (val_b) ? (val_a) : (val_b)
#define SUM(val_a, val_b)           (val_a) + (val_b)

/* Imagine result of this equation using wrong SUM implementation */
int32_t expected_result = 5 * SUM(3, 4);  /* Expected result is 5 * 7 = 35 */
int32_t actual_result = 5 * (3) + (4);    /* It is evaluated to this, final result = 19 which is not what we expect */

/* Correct implementation */
#define MIN(val_a, val_b)           ((val_a) < (val_b) ? (val_a) : (val_b))
#define SUM(val_a, val_b)           ((val_a) + (val_b))
```

- When macro uses multiple statements, protect these using `do {} while (0)` statement
```c
typedef struct {
    int32_t pos_x;
    int32_t pos_y;
} point_t;
point_t pnt;                  /* Define new point */

/* Wrong implementation */

/* Define macro to set point */
#define SET_POINT(ptr, val_a, val_b)  (ptr)->pos_x = (val_a); (ptr)->pos_y = (val_b)    /* 2 statements. Last one should not implement semicolon */

SET_POINT(&pnt, 3, 4);        /* Set point to position 3, 4. This evaluates to... */
(&pnt)->pos_x = (3); (&pnt)->pos_y = (4); /* ... to this. In this example this is not a problem. */

/* Consider this ugly code, however it is valid by C standard (not recommended) */
if (cnd_a)                      /* If cnd_a is true */
    if (cnd_b)                  /* If cnd_b is true */
        SET_POINT(&pnt, 3, 4);/* Set point to val_a = 3, val_b = 4 */
    else
        SET_POINT(&pnt, 5, 6);/* Set point to val_a = 5, val_b = 6 */

/* Evaluates to code below. Do you see the problem? */
if (cnd_a)
    if (cnd_b)
        (&pnt)->pos_x = (3); (&pnt)->pos_y = (4);
    else
        (&pnt)->pos_x = (5); (&pnt)->pos_y = (6);

/* Or if we rewrite it a little */
if (cnd_a)
    if (cnd_b)
        (&pnt)->pos_x = (3);
        (&pnt)->pos_y = (4);
    else
        (&pnt)->pos_x = (5);
        (&pnt)->pos_y = (6);

/*
 * Ask yourself a question: To which `if` statement does the `else` keyword belong?
 *
 * Based on first part of code, answer is straight-forward. To inner `if` statement when we check `cnd_b` condition
 * Actual answer: Compilation error as `else` belongs nowhere
 */

/* Better and correct implementation of macro */
#define SET_POINT(ptr, val_a, val_b)  do { (ptr)->pos_x = (val_a); (ptr)->pos_y = (val_b); } while (0)    /* 2 statements. No semicolon after while loop */
/* Or even better */
#define SET_POINT(ptr, val_a, val_b)  do {     \   /* Backslash indicates statement continues in new line */
    (ptr)->pos_x = (val_a);                    \
    (ptr)->pos_y = (val_b);                    \
} while (0)                                     /* 2 statements. No semicolon after while loop */

/* Now original code evaluates to */
if (cnd_a)
    if (cnd_b)
        do { (&pnt)->pos_x = (3); (&pnt)->pos_y = (4); } while (0);
    else
        do { (&pnt)->pos_x = (5); (&pnt)->pos_y = (6); } while (0);

/* Every part of `if` or `else` contains only `1` inner statement (do-while), hence this is valid evaluation */

/* To make code perfect, use brackets for every if-ifelse-else statements */
if (cnd_a) {                    /* If cnd_a is true */
    if (cnd_b) {                /* If cnd_b is true */
        SET_POINT(&pnt, 3, 4);/* Set point to val_a = 3, val_b = 4 */
    } else {
        SET_POINT(&pnt, 5, 6);/* Set point to val_a = 5, val_b = 6 */
    }
}
```

- Avoid using `#ifdef` or `#ifndef`. Use `defined()` or `!defined()` instead
```c
#ifdef XYZ
/* do something */
#endif /* XYZ */
```

- Always document `if/elif/else/endif` statements
```c
/* OK */
#if defined(XYZ)
/* Do if XYZ defined */
#else /* defined(XYZ) */
/* Do if XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
/* Do if XYZ defined */
#else
/* Do if XYZ not defined */
#endif
```

- Do not indent sub statements inside `#if` statement
```c
/* OK */
#if defined(XYZ)
#if defined(ABC)
/* do when ABC defined */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* Do when XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
    #if defined(ABC)
        /* do when ABC defined */
    #endif /* defined(ABC) */
#else /* defined(XYZ) */
    /* Do when XYZ not defined */
#endif /* !defined(XYZ) */
```

## Documentation

Documented code allows doxygen to parse and generate html/pdf/latex output, thus it is very important to do it properly at an early stage of the project.

- Use doxygen-enabled documentation style for `variables`, `functions` and `structures/enumerations`
- Always use `\` for doxygen, do not use `@`
- Pad every doxygen tag with spaces so the description text starts at column `22` of the line, matching `template.c`/`template.h`
```c
/**
 * \brief           Holds pointer to first entry in linked list
 *                  This text is aligned to the same column as the \brief description above
 */
static
type_t* list;
```

- Every structure/enumeration member MUST include documentation
- Align start of comments between different structure members to the same column
```c
/**
 * \brief           This is point struct
 * \note            This structure is used to calculate all point
 *                      related stuff
 */
typedef struct {
    int32_t pos_x;                               /*!< Point X coordinate */
    int32_t pos_y;                               /*!< Point Y coordinate */
    int32_t size;                                /*!< Point size.
                                                     Since comment is very big,
                                                     you may go to next line */
} point_t;

/**
 * \brief           Point color enumeration
 */
typedef enum {
    COLOR_RED,                                  /*!< Red color */
    COLOR_GREEN,                                /*!< Green color */
    COLOR_BLUE,                                 /*!< Blue color */
} point_color_t;
```

- Documentation for a function MUST be written at its *definition* (where the function body is implemented, typically the `.c` file), and MUST NOT be duplicated at its *declaration* (the prototype without a body, typically in the header file)
- Function MUST include `brief` and all parameters documentation
- Every parameter MUST be noted if it is `in` or `out` for *input* and *output* respectively
- Function MUST include `return` parameter if it returns something. This does not apply for `void` functions
- Function can include other doxygen keywords, such as `note` or `warning`
- Use colon `:` between parameter name and its description
```c
/**
 * \brief           Sum `2` numbers
 * \param[in]       par_a: First number
 * \param[in]       par_b: Second number
 * \return          Sum of input values
 */
int32_t sum(int32_t par_a, int32_t par_b) {
    return par_a + par_b;
}

/**
 * \brief           Sum `2` numbers and write it to pointer
 * \note            This function does not return value, it stores it to pointer instead
 * \param[in]       par_a: First number
 * \param[in]       par_b: Second number
 * \param[out]      result: Output variable used to save result
 */
void void_sum(int32_t par_a, int32_t par_b, int32_t* result) {
    *result = par_a + par_b;
}
```

- If function returns member of enumeration, use `ref` keyword to specify which one
```c
/**
 * \brief           My enumeration
 */
typedef enum {
    MY_ERR,                                     /*!< Error value */
    MY_OK                                       /*!< OK value */
} my_enum_t;

/**
 * \brief           Check some value
 * \return          \ref MY_OK on success, member of \ref my_enum_t otherwise
 */
my_enum_t check_value(void) {
    return MY_OK;
}
```

- Wrap constants, literal values, and code identifiers referenced in documentation text in backticks, eg. `` `NULL` ``
```c
/**
 * \brief           Get data from input array
 * \param[in]       inp: Input data
 * \return          Pointer to output data on success, `NULL` otherwise
 */
const void* get_data(const void* inp) {
    return inp;
}
```

- Documentation for macros MUST include `hideinitializer` doxygen command
```c
/**
 * \brief           Get minimal value between `val_a` and `val_b`
 * \param[in]       val_a: First value
 * \param[in]       val_b: Second value
 * \return          Minimal value between `val_a` and `val_b`
 * \hideinitializer
 */
#define MIN(val_a, val_b)       ((val_a) < (val_b) ? (val_a) : (val_b))
```

## Header/source files

- Leave single empty line at the end of file
- Every file MUST include doxygen annotation for `file` and `brief` description followed by empty line (when using doxygen)
```c
/**
 * \file            template.h
 * \brief           Template include file
 */
                    /* Here is empty line */
```

- Every file (*header* or *source*) MUST include license (opening comment includes single asterisk as this MUST be ignored by doxygen)
- Use the same license as already used by project/library
```c
/**
 * \file            template.h
 * \brief           Template include file
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */
```

- Header files MUST include header guards using `#ifndef`
- Header files MUST include a `C++` compatibility check
- Include external headers outside the `C++` check
- Include standard C headers before application-specific headers
- Header files should include only the headers required to compile correctly; `.c` files should include additional headers if needed
- Header files MUST expose only module public variables, types, and functions
- Use `extern` for global module variables in header files and define them in source files later
```
/* file.h ... */
#ifndef ...

extern int32_t my_variable; /* This is a global variable declaration in a header */

#endif

/* file.c ... */
int32_t my_variable;        /* Actually defined in source */
```
- Never include `.c` files in another `.c` file
- `.c` file should first include corresponding `.h` file, later others, unless otherwise explicitly necessary
- Do not include module private declarations in header file

- Header file example (no license for sake of an example)
```c
/* License comes here */
#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/* Include headers */

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/* File content here */

#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif /* TEMPLATE_HDR_H */
```

## Clang format integration

This repository includes an up-to-date `.clang-format` file for `clang-format`. The tool can be integrated with most modern IDEs, including VSCode, or run directly from the command line.

### Formatting scope and wrapping

<a id="fork-formatting-scope"></a>

Reformat only files authored from scratch for the adopting project. Files copied
or adapted from libraries, vendors, SDKs, code generators, or external templates
retain their original formatting, even outside submodules and after local edits.
Reformat those files only when the project owner explicitly authorizes them.
The reference templates maintained in this style repository are explicitly in
scope for its own formatting checks.

Before formatting a whole project, identify file provenance and select an
explicit list of project-authored files. Git tracking and location outside a
submodule do not establish authorship. Preserve the adopting project's workflow,
approval requirements, and build/test restrictions.

<a id="fork-line-wrapping"></a>

- Use a 120-column limit and four spaces for indentation and continuations.
- Keep calls and signatures on one line when they fit. Otherwise, put each
  argument or parameter on its own line, indent by four spaces, and put the
  closing parenthesis on a separate line at the call or declaration's indentation.
  Keep the return type on the same line as the function name.
- Keep short, simple structure initializers compact. When such an initializer
  wraps after its first field, align the next field with the first field rather
  than adding another four-space indent. The existing trailing-comma exception
  for short, simple structures still applies; add required C structure trailing
  commas yourself because `clang-format` does not insert them.
- Keep `&&` and `||` at line ends. Prefer one check per line when a condition
  needs wrapping. Other binary operators also stay at line ends when wrapped.
- Keep string literals intact. Do not split diagnostic text into word fragments
  to meet the column limit. Adjacent literals used to lay out multiline output
  remain supported.
- Prefer a break after assignment when it lets the right-hand expression remain
  readable. Expressions that fit on one line stay on one line.

For example, a wrapped signature and a wrapped call use the same bracket layout:

```c
int32_t process_measurements(
    const measurement_configuration_t* configuration,
    const measurement_sample_t* samples,
    size_t sample_count,
    measurement_result_t* result
);

print_status(
    "State     : %s\n"
    "Frequency : %lu Hz\n"
    "Volume    : %u %%\n",
    status.enabled ? "on" : "off",
    (unsigned long)status.frequency_hz,
    (unsigned)status.volume_percent
);
```

A compact table entry keeps its fields aligned:

```c
    {OPT_FREQUENCY,
     WSH_SHELL_OPT_STR(WSH_SHELL_OPT_ACCESS_WRITE, "-f", "--frequency", "PWM frequency in Hz (20000..50000)")},
```

LLVM 23 does not enforce a completely flat layout for every nested condition or
every two-operand logical expression. It may retain checks together or add
operand alignment and continuation indentation, including inside `_Static_assert`.
Accept the native formatter result for these cases. Do not add postprocessing,
format-disable markers, extra parentheses, or semantic rewrites solely to force
a different cosmetic layout.

To format a single project-authored file manually:
```sh
clang-format -i path/to/file.c
```

To format an explicitly selected set of project-authored files:
```sh
clang-format -i app/src/control.c app/include/control.h
```

Recommended installation methods:
- macOS: `brew install llvm` and add `$(brew --prefix llvm)/bin` to `PATH`
- Linux: install the `clang-format` package from your distribution, or install LLVM from the official LLVM repository
- Windows: install LLVM from the official LLVM installer and ensure `clang-format.exe` is on `PATH`

If the tool is not available, the repository formatting rules can still be reviewed manually, but automated formatting will not work until `clang-format` is installed.

> If one uses Visual Studio Code (vscode), the `clang-format` tool ships with the IDE.

https://code.visualstudio.com/docs/cpp/cpp-ide#_code-formatting

### Continuous integration (GitHub Actions)

Running `clang-format --dry-run --Werror` as a required PR check is standard
practice (LLVM itself does this): it's a fast, objective gate that catches
formatting drift regardless of whether a contributor has their editor or a
pre-commit hook set up, and it removes "please fix the formatting"
review comments entirely.

This repository includes `.github/workflows/clang-format.yml`, which:
1. Installs `clang-format` 23 from LLVM's official apt repository instead of
   relying on the runner's default version.
2. Runs `scripts/check-format.sh --check` against the maintained reference
   templates, `template.c` and `template.h`, failing the job if either is not
   compliant. Adopting projects must replace that explicit list with their own
   project-authored files and preserve their vendor and submodule exclusions.

By itself, a failing check does **not** block a merge — GitHub still allows
merging past a red check unless you tell it not to. To make it a real gate:
1. Push the workflow to your default branch, then open (or push to) one PR
   so the check runs at least once — GitHub only lets you select checks in
   branch protection after they've appeared in the repository's check
   history.
2. Go to **Settings → Branches → Branch protection rules** and add (or edit)
   a rule for your default branch.
3. Enable **"Require status checks to pass before merging"**, then search
   for and select `clang-format / Check clang-format compliance`.
4. (Optional) Also enable "Require branches to be up to date before
   merging" so PRs are re-checked against the latest base branch.
5. Save. PRs with formatting violations can no longer be merged until fixed.
