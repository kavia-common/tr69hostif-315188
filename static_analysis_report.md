# Static analysis summary (partial) — tr69hostif-315188

## Tooling used
- GCC 13.3.0 (`-fsyntax-only` with `-Wall -Wextra -Wpedantic -Wshadow -Wformat=2 -Wconversion -Wsign-conversion -Wstrict-prototypes -Wmissing-prototypes -Wold-style-definition ...`)

## Coverage limitations / blockers
- `clang`, `clang-tidy`, and `cppcheck` are not installed in the environment.
- Autotools (`autoreconf`, generated `configure`) is not available and the repo does not include pre-generated `configure`/`Makefile`/`cfg/config.h`. This prevents a full compile across all translation units with warnings enabled.
- Some sources depend on external headers (example: `glib.h`) that are not available via default include paths; full analysis requires installing dev packages and using pkg-config include flags.

## Findings (selected/high-signal)

### 1) `src/backgroundrun.c:28`
- **Severity/category:** warning / C-Style (prototypes)
- **Message:** function declaration isn’t a prototype (`void usage()`)
- **Suggested fix:** change to `static void usage(void)`; avoid old-style definitions.

### 2) `src/unittest/stubs/paramMgr.c:1`
- **Severity/category:** error / Build-Dependencies
- **Message:** `fatal error: glib.h: No such file or directory`
- **Suggested fix:** install GLib dev headers and compile with `pkg-config --cflags glib-2.0` (or ensure build system provides `GLIB_CFLAGS`).

### 3) `src/unittest/stubs/secure_wrapper.c`
**Portability / non-standard constructs**
- **`~line 40`**: named variadic macros (`args...`) flagged under `-Wpedantic`.
  - Fix: use C99 `...` and `__VA_ARGS__`.
- **`~line 51`**: GNU statement-expression `({ ... })` used by macro `FAIL` flagged under `-Wpedantic`.
  - Fix: use `do { ... } while (0)` macros or inline functions.

**Maintainability**
- **`~line 86+`**: repeated `-Wshadow` occurrences: local `task` variables shadow a global `task`.
  - Fix: rename one side; avoid global mutable state in test stubs where possible.

**API hygiene**
- **`~line 806, 977, 992, 1038...`**: `-Wmissing-prototypes` / “no previous prototype” warnings (e.g., `v_secure_system`, `v_secure_popen`).
  - Fix: add prototypes in headers or mark these functions `static` if internal.

**Correctness**
- **`~line 410`**: `-Wconversion` int-to-char conversion may change value.
  - Fix: validate bounds and use explicit casts with correct signedness.

## Recommended next steps (for full-project coverage)
1. Install missing tooling:
   - `clang`, `clang-tidy`, `cppcheck`, `bear` (or `compiledb`) and autotools (`autoconf`, `automake`, `libtool`).
2. Generate a compilation database:
   - Use autotools build + `bear -- make` to produce `compile_commands.json`.
3. Run:
   - `clang-tidy -p . <files>` and `cppcheck --enable=warning,style,performance,portability --inconclusive ...`
4. Re-run GCC/Clang builds with `-Wall -Wextra` and capture full warning logs.

> This report intentionally makes **no code changes**; it only enumerates issues and recommended fixes.
