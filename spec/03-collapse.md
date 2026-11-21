# 3. Collapse of learned documents

Status: normative for what a collapsed document means; the reference algorithm is normative only for producers that claim reference collapse (3.1.4).

## Scope

A learned document records what a subject did: literal paths, literal argument vectors, literal endpoints. Collapse generalizes such a document into patterns of chapter 2 so that it admits the same behaviour with different identifiers: process and thread numbers, generated counters, per-request directories, per-script interpreter calls. This chapter defines what any collapse must preserve, the settings and thresholds, the reference algorithm for opens, execs and endpoints, the procfs and counter folding, the atomic-writer fold, and idempotence.

Clause numbers have the form `3.<section>.<n>`.

## 3.1 What is normative

**3.1.1** A collapsed document is a document. Its meaning MUST be given by chapters 1 and 2 alone; a consumer MUST NOT need to know whether or how it was collapsed.

**3.1.2** A collapse MUST be sound: every observation that the input document admits MUST be admitted by the output document.

**3.1.3** A collapse MUST NOT write an unmatchable entry (2.10.1).

**3.1.4** When to collapse, how often, and with which settings is left to the producer. A producer that claims reference collapse MUST produce, for the reference settings, the same entries as the reference algorithm of 3.3 to 3.6, compared as sets of entries.

## 3.2 Settings

The reference settings are the constants of the reference library.

| setting | constant | value | applies to |
|---|---|---|---|
| open threshold | `OpenDynamicThreshold` | 50 | distinct children of a path node |
| exec threshold | `ExecDynamicThreshold` | 50 | distinct values of an argument position, distinct argv0 values, entries per binary |
| endpoint threshold | `EndpointDynamicThreshold` | 100 | distinct children of an endpoint path node |
| address group threshold | `NetworkIPGroupThreshold` | 50 | entries of a network group that differ only by address |
| CIDR floor | `NetworkCIDRFloorBits` | 24 | the shortest prefix an aggregated block may have |
| CIDR split cap | `NetworkMaxCIDRSplitBits` | 12 | how far one block is split into floor-width blocks |

The reference per-prefix thresholds are:

| prefix | threshold |
|---|---|
| `/etc` | 100 |
| `/etc/apache2` | 50 |
| `/opt` | 50 |
| `/var/run` | 50 |
| `/app` | 50 |

**3.2.1** The threshold for a path MUST be the threshold of the per-prefix setting whose prefix is the longest one that ends at a segment boundary of the path, and the global threshold when no prefix applies. On equal length the first listed setting wins. A trailing `/` on a prefix MUST be ignored; the prefix `/` and the empty prefix apply to every absolute path.

**3.2.2** A non-positive global exec threshold MUST mean the reference value. Per-prefix settings with a threshold below 1 MUST be ignored for execs.

## 3.3 Opens

The reference algorithm keeps one tree of path segments per collapse run. Every segment of every input path is a node.

**3.3.1** Every input path MUST be cleaned (repeated `/`, `.` and `..` segments resolved) before it is analysed. An empty path MUST stay empty.

**3.3.2** When a node has more distinct children than the threshold for the node's path, its children MUST be merged into one `⋯` child whose subtree is the union of theirs. A node that has a `⋯` child routes every later child through it.

**3.3.3** When the threshold for a node's path is 1, the first new child of that node MUST make the node's children one `*` child.

**3.3.4** A `*` child MUST absorb every later child of its node, and the output path MUST end at the `*`.

**3.3.5** In every output path, a run of two or more adjacent `⋯` segments MUST be replaced by one `*` segment.

**3.3.6** Under `/proc/`, at any depth, a segment of digits only and a `⋯` segment MUST be treated as one `⋯` child on first sight, whatever the threshold. Named procfs entries (`self`, `net`, `sys`, ...) MUST stay literal in the tree unless the threshold merges them; consolidation (3.3.10) still drops them when a pattern covers them.

**3.3.7** Anywhere, a segment of at least four digits that starts with `0` (a generated counter) MUST be treated as a `⋯` child on first sight. Other numbers MUST stay literal.

**3.3.8** An input path that carries a `*` segment MUST make the children of that node one `*` child that absorbs the existing children, a `⋯` child included, so the output is at least as broad as the input.

**3.3.9** The flags of input entries that collapse to the same path MUST be united.

**3.3.10** After the tree pass, the open set MUST be consolidated: a literal entry that a pattern entry matches (2.2) MUST be dropped and its flags united into the covering pattern with the longest path string (the lexically smaller on equal length); a pattern entry whose every match is matched by a broader pattern entry MUST be dropped and its flags united into the broader one. No stored open set carries an entry that another entry of the same set covers.

**3.3.11** The reference algorithm accepts a set of pinned paths that are kept as they are and never folded. Its use is an open point.

```yaml example:collapse-opens
threshold: 2
input: [/data/a/info, /data/b/info, /data/c/info]
output: [/data/⋯/info]
```

```yaml example:collapse-opens
threshold: 2
input: [/data/a/info, /data/b/info]
output: [/data/a/info, /data/b/info]
```

```yaml example:collapse-opens
threshold: 2
input: [/a/x/1/f, /a/y/2/f, /a/z/3/f]
output: [/a/*/f]
```

```yaml example:collapse-opens
threshold: 50
input: [/proc/123/status, /proc/1/task/77/stat]
output: [/proc/⋯/status, /proc/⋯/task/⋯/stat]
```

```yaml example:collapse-opens
threshold: 50
input: [/tmp/cache/proxy/0000000001, /tmp/cache/007, /tmp/cache/0]
output: [/tmp/cache/proxy/⋯, /tmp/cache/007, /tmp/cache/0]
```

```yaml example:collapse-opens
threshold: 50
input: [/etc/*, /etc/passwd, /etc/ssl/certs/a.pem]
output: [/etc/*]
```

## 3.4 Execs

**3.4.1** Entries with `argsRequired: true` and entries without arguments MUST pass through unchanged; equal ones MUST be merged. Direct invocations of a runtime (3.4.9) MUST NOT go through 3.4.4 to 3.4.7; they follow 3.4.11, and an entry that another entry of the same path covers MUST be absorbed (3.4.6).

**3.4.2** The other entries MUST be grouped by `path`, and the threshold of each group is the threshold for its path (3.2.1).

**3.4.3** Within a group, entries with the same argument vector MUST be merged and their `envs` united. Entries that another entry of the group covers MUST be absorbed before any generalization, so that a delta a stored document already admits never widens it.

**3.4.4** When a group has more distinct first arguments (argv0) than the threshold, every literal argv0 MUST be replaced by its shape: argv0 values are split at `/` into segments, values with the same number of segments share one segment tree, and a segment position with more distinct values than the threshold becomes `⋯`. A shape that does not match its original argv0 MUST be replaced by `⋯`. When the shapes are still more than the threshold, every argv0 MUST become `⋯`. An argv0 that already contains `⋯` MUST be kept. For a path that names a runtime (3.4.9), a shape MUST only replace an argv0 when its last segment is literal and does not itself name a runtime; otherwise the argv0 MUST be kept literal, and the replacement by `⋯` MUST NOT happen.

**3.4.5** The remaining vectors without a run-token MUST be put in argument trees, one per pair of argv0 and vector length, one level per position. A node with more distinct children than the threshold, or with a `⋯` child and any other child, MUST have its children merged into one `⋯` child whose subtree is the union of theirs. Vectors of different lengths MUST NOT be merged.

**3.4.6** The group MUST then be consolidated: an entry that another entry of the group covers MUST be dropped and its `envs` united into the covering entry. An entry covers another when its vector matches the other's vector read as an observation (2.3). An entry that contains a run-token MUST only be merged with an equal entry.

**3.4.7** When a consolidated group still has more entries than the threshold, it MUST be replaced by one entry `[argv0, ⋯⋯]` per distinct argv0, carrying the united `envs` of that argv0.

**3.4.8** The output MUST be ordered by path, then by argument vector, then entries without `argsRequired` before those with it, then by `envs`.

**3.4.9** A runtime is named by the last path element of `path`: `sh`, `bash`, `dash`, `ash`, `zsh`, `ksh`, `mksh`, `node`, `nodejs`, `java`, `awk`, `gawk`, `mawk`, `nawk`, or `python`, `perl`, `ruby`, `php` or `lua` followed by nothing or by a version made of digits and dots (`python3.12`, `perl5.36.0`). For `busybox` the runtime is the applet `sh`, `ash` or `awk`, named by the first element or, when the first element is `busybox`, by the second. The wrappers `env`, `nohup`, `nice`, `timeout`, `stdbuf` and `xargs` run the runtime named after their own options and, for `env`, assignments. An entry is a direct invocation when its path names a runtime and the last path element of its first element names the same runtime, or contains a token. Any other first element on a runtime path is a script started through its interpreter line (2.3.11), and its arguments belong to the script.

**3.4.10** On a direct invocation the main operand is the argument that selects the code that runs: the program text of `sh -c` (the first argument after the options when an option cluster contains `c`), `python -c`, `perl -e`/`-E`, `ruby -e`, `node -e`/`--eval`/`-p`/`--print`, `php -r`/`-R`/`-B`/`-E`, `lua -e` and the `awk` program, the value of `python -m`, `java -jar`, `java -m`/`--module`, `awk -f` and `php -f`, the java main class, or else the script, the first argument after the options. Options that load code keep their value with them (`java -cp`, `-classpath`, `--class-path`, `-p`, `--module-path`, `-javaagent:`, `-agentpath:`, `-agentlib:`, `-XX:OnOutOfMemoryError=`, `-XX:OnError=`; `node -r`/`--require`/`--import`/`--loader`/`--experimental-loader`; `perl -M`/`-m`/`-I`; `ruby -r`/`-I`; `php -d auto_prepend_file=`/`auto_append_file=`; `lua -l`); they come before the main operand.

**3.4.11** On a direct invocation every argument up to and including the main operand MUST stay literal: a collapse step MUST NOT put a token or a run-token there, and a producer MUST NOT write one. A whole-argument `⋯` matches any argument (2.3), so a token among the options can stand for an injected option that loads code. Arguments after the main operand belong to the program and MAY be generalized; entries with the same literal prefix and length share an argument tree for those arguments, and when the prefix still has more entries than the threshold they MUST be replaced by the prefix followed by `⋯⋯`. An entry with no main operand MUST stay literal. Arguments of a script started through its interpreter line follow 3.4.4 to 3.4.7.

```yaml example:collapse-execs
threshold: 2
path: /usr/bin/grep
input: [[grep, -e, a], [grep, -e, b], [grep, -e, c]]
output: [[grep, -e, ⋯]]
```

```yaml example:collapse-execs
threshold: 2
path: /usr/bin/grep
input: [[grep, -e, a], [grep, -x, b], [grep, -i, c], [grep, a], [grep, b], [grep, c]]
output: [[grep, ⋯], [grep, ⋯, ⋯]]
```

```yaml example:collapse-execs
threshold: 2
path: /bin/sh
input: [[sh, -c, a], [sh, -c, b], [sh, -c, c]]
output: [[sh, -c, a], [sh, -c, b], [sh, -c, c]]
```

```yaml example:collapse-execs
threshold: 2
path: /usr/bin/python3.12
input: [[python3, -W, ignore, -c, a], [python3, -W, ignore, -c, b], [python3, -W, ignore, -c, c]]
output: [[python3, -W, ignore, -c, a], [python3, -W, ignore, -c, b], [python3, -W, ignore, -c, c]]
```

```yaml example:collapse-execs
threshold: 2
path: /opt/java/bin/java
input: [[/opt/java/bin/java, -Xmx1g, -cp, /app/lib, app.Main, a], [/opt/java/bin/java, -Xmx2g, -cp, /app/lib, app.Main, b], [/opt/java/bin/java, -Xmx3g, -cp, /app/lib, app.Main, c]]
output: [[/opt/java/bin/java, -Xmx1g, -cp, /app/lib, app.Main, a], [/opt/java/bin/java, -Xmx2g, -cp, /app/lib, app.Main, b], [/opt/java/bin/java, -Xmx3g, -cp, /app/lib, app.Main, c]]
```

```yaml example:collapse-execs
threshold: 2
path: /opt/java/bin/java
input: [[/opt/java/bin/java, -cp, /app/lib, app.Main, a], [/opt/java/bin/java, -cp, /app/lib, app.Main, b], [/opt/java/bin/java, -cp, /app/lib, app.Main, c]]
output: [[/opt/java/bin/java, -cp, /app/lib, app.Main, ⋯]]
```

```yaml example:collapse-execs
threshold: 2
path: /usr/bin/perl
input: [[/usr/bin/pg_isready, -h, a], [/usr/bin/pg_isready, -h, b], [/usr/bin/pg_isready, -h, c]]
output: [[/usr/bin/pg_isready, -h, ⋯]]
```

```yaml example:collapse-execs
threshold: 1
path: /bin/sh
input: [[/tmp/t.1/run.sh], [/tmp/t.2/run.sh]]
output: [[/tmp/⋯/run.sh]]
```

```yaml example:collapse-execs
threshold: 50
path: /bin/sh
input: [[sh, ⋯⋯], [sh, -c, x]]
output: [[sh, ⋯⋯]]
```

## 3.5 Endpoints

**3.5.1** An endpoint MUST be read as a URL, with `http://` assumed when it has no scheme and a bare IPv6 host put in brackets, and written as `:<port><path>`. Its path MUST be analysed by 3.3.2 to 3.3.5 in a separate tree per port, with the endpoint threshold.

**3.5.2** Entries that collapse to the same endpoint, direction and `internal` value MUST be merged: methods united, header values united per header name.

**3.5.3** An entry with a specific port MUST be merged into an entry with port `0`, the same path, the same direction and the same `internal` value. Entries with different ports MUST NOT be merged otherwise.

## 3.6 Network entries

The reference library defines the settings of address aggregation (3.2) but not the aggregation itself. The settings mean: a group of network entries that differ only by address is aggregated into CIDR blocks when it has more entries than the address group threshold; an aggregated block has a prefix length of at least the CIDR floor; a block broader than the floor is split into blocks of the floor's width only when that gives at most 2 to the power of the CIDR split cap blocks, and is kept as it is otherwise. See Open points.

## 3.7 The atomic-writer fold

A projected volume is written through a directory whose name changes on every update: two full stops, a timestamp, a full stop and digits (`..2026_01_31_12_00_00.123`). The atomic-writer fold makes paths through such directories stable.

**3.7.1** A segment is an atomic-writer segment when it is two full stops, then four digits, `_`, two digits, `_`, two digits, `_`, two digits, `_`, two digits, `_`, two digits, then a full stop, then one or more digits, and nothing else.

**3.7.2** The atomic-writer fold of a string MUST replace every atomic-writer segment after the first `/`-separated element of the string by `⋯` and MUST change nothing else.

```yaml example:fold
cases:
- {in: "app_/run/secrets/kubernetes.io/serviceaccount/..2026_11_11_01_20_25.1234567/token", out: "app_/run/secrets/kubernetes.io/serviceaccount/⋯/token"}
- {in: "app_/run/secrets/kubernetes.io/serviceaccount/..data/token", out: "app_/run/secrets/kubernetes.io/serviceaccount/..data/token"}
- {in: "sh_/tmp/..2026_11_11/token", out: "sh_/tmp/..2026_11_11/token"}
- {in: "sh_/tmp/..2026_11_11_02_08_39./token", out: "sh_/tmp/..2026_11_11_02_08_39./token"}
- {in: "sh_/tmp/a..2026_11_11_02_08_39.42/token", out: "sh_/tmp/a..2026_11_11_02_08_39.42/token"}
- {in: "cat_/a/..2026_08_03_16_47_11.1/b/..2026_08_03_16_47_12.2/c", out: "cat_/a/⋯/b/⋯/c"}
- {in: "curl_10.0.0.1_9443", out: "curl_10.0.0.1_9443"}
```

## 3.8 Idempotence

**3.8.1** Collapsing the output of an exec collapse again with the same settings MUST give the same output.

**3.8.2** The procfs and counter folding of 3.3.6 and 3.3.7 MUST be idempotent: a path they produced is left unchanged by them.

## Conformance

A producer that claims reference collapse conforms when it reproduces the examples marked `example:` in this chapter and the reference collapse vectors (inputs and expected outputs over several learning windows, and a real open set of a package directory), and when its outputs pass 3.1.2 and 3.1.3. Every producer that collapses MUST meet 3.1.1 to 3.1.3.

## Open points

- The skeleton of this chapter stated that a document authored by a person is stored as written and never collapsed again. The reference library does not decide this; it is the storing component's rule and is not pinned in this version.
- The pinned paths of 3.3.11 were the files listed in the image's software bill of materials. Whether such files are exempt from folding is decided together with the bill-of-materials question.
- The aggregation of learned addresses into CIDR blocks (3.6) is implemented by the storing component, not the reference library; its algorithm is not specified here.
- The atomic-writer fold (3.7) is used for the deny-all subject (6.6). Whether learned open paths should be folded by it is not decided.
- Idempotence of the open collapse as a whole is not pinned; 3.8 states what is.
- With a global threshold of 1, the reference open collapse turns every path into `*`.

## Changes from v0.0.4

v0.0.4 had no text on collapse. Everything in this chapter is new.
