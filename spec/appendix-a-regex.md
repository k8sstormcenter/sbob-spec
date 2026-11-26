# Appendix A. Regular-expression rendering of patterns

Status: normative for any engine that evaluates patterns of chapter 2 through regular expressions.

## Scope

Some engines cannot run the matcher of chapter 2 and evaluate regular expressions instead. This appendix defines how a path pattern and an argument-vector pattern are rendered to one RE2 expression, and when a rendering conforms. The reference library implements it as `PathRegexp`, `ArgvRegexp` and `ArgTokenRegexp` in its `match` package.

Clause numbers have the form `A.<section>.<n>`.

## A.1 Paths

The subject of a path expression is the observed path in canonical form: no empty segment and no trailing `/`.

| pattern element | rendering | meaning |
|---|---|---|
| the empty pattern | `^$` | matches nothing a canonical subject can be |
| the pattern `*` alone | `^.+$` | the catch-all (2.2.8) |
| literal segment | the segment with every regular-expression metacharacter escaped | byte equality |
| a segment that is exactly `⋯` | `[^/]+` | exactly one segment |
| `*` as the last segment, after a `/` | `/.+` | one or more segments (2.2.5) |
| `*` as a segment that is not the last | `(?:[^/]+/)*` in place of the segment and its following `/` | zero or more segments (2.2.6) |
| a run of adjacent `*` segments | rendered as one `*` | 2.2.7 |
| a segment pattern (2.2.10) | its literal parts escaped, each `⋯` rendered as `[^/]+` | one or more code points per `⋯` (2.2.10) |
| `⋯⋯` or `*` inside a segment, `⋯⋯` as a segment | escaped as literal characters | 2.2.10 |

**A.1.1** A path pattern MUST render to one expression anchored with `^` and `$`.

**A.1.2** The rendering MUST drop a trailing `/` of the pattern before rendering its segments (2.2.2).

**A.1.3** The rendering MUST collapse runs of adjacent star segments to one before rendering (2.2.7).

**A.1.4** Each element MUST be rendered as the table above states, and the elements MUST be joined with `/` except after a non-last star segment, whose rendering carries its own `/`.

```yaml example:regex-path
cases:
- {pattern: "", regex: "^$", match: [], nomatch: [/]}
- {pattern: "*", regex: "^.+$", match: [/, /etc/passwd], nomatch: []}
- {pattern: /etc/*, regex: "^/etc/.+$", match: [/etc/passwd, /etc/ssh/sshd_config], nomatch: [/etc]}
- {pattern: /a/*/b, regex: "^/a/(?:[^/]+/)*b$", match: [/a/b, /a/x/y/b], nomatch: [/a/x/c]}
- {pattern: /a/*/*/b, regex: "^/a/(?:[^/]+/)*b$", match: [/a/b, /a/x/b], nomatch: [/a/x/c]}
- {pattern: /a/⋯/b, regex: "^/a/[^/]+/b$", match: [/a/x/b], nomatch: [/a/b, /a/x/y/b]}
- {pattern: /x/*/*, regex: "^/x/.+$", match: [/x/y, /x/y/z], nomatch: [/x]}
- {pattern: /a/*/b/*/c, regex: "^/a/(?:[^/]+/)*b/(?:[^/]+/)*c$", match: [/a/b/c, /a/1/b/2/3/c], nomatch: [/a/b/x]}
- {pattern: "*/x", regex: "^(?:[^/]+/)*x$", match: [x, a/b/x], nomatch: [a/y]}
- {pattern: /var/log/, regex: "^/var/log$", match: [/var/log], nomatch: [/var/log/x]}
- {pattern: /opt/x.y, regex: "^/opt/x\\.y$", match: [/opt/x.y], nomatch: [/opt/xzy]}
- {pattern: /tmp/x⋯, regex: "^/tmp/x[^/]+$", match: [/tmp/xy], nomatch: [/tmp/x, /tmp/x/y]}
- {pattern: /run/⋯.pid, regex: "^/run/[^/]+\\.pid$", match: [/run/a.pid], nomatch: [/run/.pid]}
- {pattern: /tmp/⋯⋯, regex: "^/tmp/⋯⋯$", match: [], nomatch: [/tmp/x]}
```

## A.2 Argument vectors

An argument vector renders to one expression over the observed arguments joined by a separator S that the engine chooses. In the table, S is the escaped separator and T is `[^S]*`, any run of characters without the separator. With the empty separator, T is `.*`.

| element | rendering |
|---|---|
| the empty vector | `^$` |
| literal argument | the argument with every metacharacter escaped, the star included |
| an argument that is exactly `⋯` | `T` |
| an argument that is exactly `⋯⋯` and is the only argument | `(?:T(?:S T)*)?` |
| an argument that is exactly `⋯⋯`, first of several | `(?:T S)*`, and the next argument is written without its leading separator |
| an argument that is exactly `⋯⋯`, not first | `(?:S T)*` |
| an embedded pattern (2.3.6) | its `/` segments rendered one by one and joined with `/` |
| inside an embedded pattern, a segment that is exactly `⋯` | `[^/S]*` |
| inside an embedded pattern, a segment that is exactly `⋯⋯` | `[^/S]*(?:/[^/S]*)*` |
| inside an embedded pattern, a segment pattern (2.3.6) | its literal parts escaped, each `⋯` rendered as `[^/S]+` |
| inside an embedded pattern, any other segment | escaped literally |

**A.2.1** An argument pattern MUST render to one expression anchored with `^` and `$`, with S between consecutive arguments except where the table says otherwise.

**A.2.2** The empty vector MUST render `^$`, which matches only the argument-less execution (2.3.2).

**A.2.3** The star MUST be escaped in every argument (2.3.3).

**A.2.4** Each argument MUST be rendered as the table above states.

**A.2.5** An engine that sees only a joined command line SHOULD choose a separator that cannot occur inside an argument. A separator that can occur inside an argument makes argument boundaries ambiguous; the rendering then admits observations the matcher does not, and the engine's fidelity for argument rules is degraded (chapter 7).

```yaml example:regex-args
separator: " "
cases:
- {pattern: [], regex: "^$", match: [[]], nomatch: [[a]]}
- {pattern: [⋯⋯], regex: "^(?:[^ ]*(?: [^ ]*)*)?$", match: [[], [a], [a, b]], nomatch: []}
- {pattern: [cargo, build, ⋯⋯], regex: "^cargo build(?: [^ ]*)*$", match: [[cargo, build], [cargo, build, --release]], nomatch: [[cargo, publish]]}
- {pattern: [⋯⋯, --x], regex: "^(?:[^ ]* )*--x$", match: [[--x], [a, b, --x]], nomatch: [[a, b]]}
- {pattern: [a, ⋯⋯, b], regex: "^a(?: [^ ]*)* b$", match: [[a, b], [a, x, y, b]], nomatch: [[a, x]]}
- {pattern: [cargo, ⋯], regex: "^cargo [^ ]*$", match: [[cargo, build]], nomatch: [[cargo, build, --release]]}
- {pattern: [ls, /plugins/*], regex: "^ls /plugins/\\*$", match: [[ls, /plugins/*]], nomatch: [[ls, /plugins/evil.so]]}
- {pattern: [node, /srv/⋯/⋯/src/⋯⋯, ⋯⋯], regex: "^node /srv/[^/ ]*/[^/ ]*/src/[^/ ]*(?:/[^/ ]*)*(?: [^ ]*)*$", match: [[node, /srv/o/r/src/x.js], [node, /srv/o/r/src/x/y.js, --ci]], nomatch: [[node, /srv/o/r/src], [node, /tmp/x.js]]}
- {pattern: [a.b, x+y], regex: "^a\\.b x\\+y$", match: [[a.b, x+y]], nomatch: [[axb, x+y]]}
```

```yaml example:regex-token
separator: " "
cases:
- {token: ⋯, regex: "[^ ]*"}
- {token: lit.eral, regex: "lit\\.eral"}
- {token: /srv/⋯/x, regex: "/srv/[^/ ]*/x"}
- {token: /srv/⋯⋯/x, regex: "/srv/[^/ ]*(?:/[^/ ]*)*/x"}
- {token: a*b, regex: "a\\*b"}
- {token: "--x=⋯", regex: "--x=[^/ ]+"}
- {token: "--x=⋯⋯", regex: "--x=⋯⋯"}
```

```yaml example:regex-token
separator: ""
cases:
- {token: ⋯, regex: ".*"}
- {token: /srv/⋯/x, regex: "/srv/[^/]*/x"}
```

## A.3 Conformance

**A.3.1** A rendering conforms when, for every path case and every argument case of the conformance corpus whose subject is in canonical form, the compiled expression gives the same answer as the reference matcher.

**A.3.2** An engine MUST check its own rendering this way before its fidelity for a pattern rule is recorded as native or generated (chapter 7).

The reference rendering is checked this way on every change of the reference library, and every example marked `example:` in this appendix holds for it.

## Open points

- A segment that is exactly `⋯` matches an empty segment in the matcher (2.2.4) and renders to `[^/]+`, which does not. The two agree only on subjects in canonical form; a rendering for non-canonical subjects is not specified.
- An embedded pattern with more than one `⋯⋯` segment matches nothing (2.3.7), but its rendering matches. Such tokens are unmatchable (2.10.1); whether an engine must refuse to render them is not decided.
- An embedded `⋯⋯` segment renders to an expression that also admits an empty segment; the matcher admits one or more segments of any content. They agree on the corpus.

## Changes from v0.0.4

v0.0.4 had no rendering appendix. Everything here is new.
