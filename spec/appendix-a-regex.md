# Appendix A. Regular-expression rendering of patterns

Status: implemented in the reference matcher (`PathRegexp`, `ArgvRegexp`), proven equal to the matcher on every path and argument case of the corpus. Normative for any engine that evaluates patterns through regular expressions.

## A.1 Paths

A path pattern renders to one RE2 expression anchored at both ends. The subject is the observed path in canonical form: no empty segments and no trailing slash. Adjacent `*` segments are collapsed to one before rendering, as the matcher does.

| pattern element | rendering | meaning |
|---|---|---|
| literal segment | the segment, regex-escaped | byte equality |
| `⋯` as a whole segment | `[^/]+` | exactly one segment |
| `*` as the last segment, after a `/` | `/.+` | one or more segments, never the directory itself |
| `*` as a whole segment before other segments | `(?:[^/]+/)*` in place of the segment and its following `/` | zero or more segments |
| the pattern `*` alone | `.+` | the explicit catch-all, the only form that admits `/` |
| a `⋯` or `*` inside a segment | escaped literally | no wildcard meaning |

Examples: `/etc/*` renders `^/etc/.+$`; `/a/*/b` renders `^/a/(?:[^/]+/)*b$`; `/a/⋯/b` renders `^/a/[^/]+/b$`; `*` renders `^.+$`.

## A.2 Argument vectors

An argument vector renders to one expression over the arguments joined by a separator the engine chooses. The separator is part of the rendering because an engine that only sees a joined command line cannot recover argument boundaries; a separator that can occur inside an argument degrades the match, which is the recorded fidelity of such engines.

| element | rendering (S is the escaped separator, T is `[^S]*`, one argument) |
|---|---|
| literal argument | escaped literally |
| `⋯` as a whole argument | `T` |
| `⋯⋯` as a whole argument, first of several | `(?:T S)*` and the next argument is written without its leading separator |
| `⋯⋯` as a whole argument, not first | `(?:S T)*` |
| `⋯⋯` as the only argument | `(?:T(?:S T)*)?` |
| `⋯` inside a `/`-segmented argument | `[^/S]*` for that segment |
| `⋯⋯` inside a `/`-segmented argument | `[^/S]*(?:/[^/S]*)*`, one or more segments |
| `*` anywhere in an argument | escaped literally |

An empty vector renders `^$` and matches only the argument-less execution.

## A.3 Conformance

A rendering is conformant when, for every path case and every argument case of the corpus, the compiled expression gives the same answer as the reference matcher. The reference rendering is checked this way on every change; an engine's own rendering is checked the same way before its fidelity row is marked native or generated.
