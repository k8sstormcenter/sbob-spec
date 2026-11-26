# 2. Pattern and matching grammar

Status: normative.

## Scope

This chapter defines the tokens a behaviour document may use in its entries and how a matcher decides whether an observation is admitted by an entry: file and executable paths, argument vectors, executable names, IP addresses and CIDR blocks, DNS names, ports and protocols, the any-sentinels, and the evaluation of allow sections together with subtract sections. It also defines which entries can never match.

Clause numbers have the form `2.<section>.<n>`. Examples marked `example:<kind>` are part of the conformance material: every statement in them holds for the reference library.

## 2.1 Tokens

| token | code points | name in this specification |
|---|---|---|
| `⋯` | U+22EF | the one-token |
| `⋯⋯` | U+22EF U+22EF | the run-token |
| `*` | U+002A | the star |

**2.1.1** The one-token MUST be the single code point U+22EF. Three full stops (`...`) and U+2026 (`…`) are literal characters and have no wildcard meaning.

**2.1.2** The run-token MUST be exactly two consecutive U+22EF code points.

**2.1.3** A token has a wildcard meaning only in the positions this chapter defines for it. In every other position its characters are literal.

**2.1.4** The grammar has no escape character. A matcher MUST NOT give any character a quoting meaning.

```yaml example:path
pattern: /a/.../b
match: [/a/.../b]
nomatch: [/a/x/b]
```

```yaml example:path
pattern: /data/odd\name
match: [/data/odd\name]
nomatch: [/data/oddname]
```

## 2.2 Paths

Path rules apply to `execs[].path` and `opens[].path`. A path is split into segments at every `/`. A path that starts with `/` has an empty first segment.

**2.2.1** An empty pattern MUST match nothing, and an empty observed path MUST be matched by no pattern.

**2.2.2** A trailing `/` on the pattern or on the observed path MUST be ignored. Any other empty segment (from `//`) is a segment like any other.

**2.2.3** A literal segment MUST match a segment that is byte-for-byte equal to it. Matching is case-sensitive.

**2.2.4** A segment that is exactly the one-token MUST match exactly one segment.

**2.2.5** A segment that is exactly the star and is the last segment of the pattern MUST match one or more segments. It MUST NOT match zero segments, so `/etc/*` does not match the directory `/etc` itself.

**2.2.6** A segment that is exactly the star and is not the last segment MUST match zero or more segments.

**2.2.7** A run of adjacent star segments MUST match exactly what a single star segment in the same position matches.

**2.2.8** The pattern that consists of the star alone MUST match every non-empty observed path, including `/` and relative paths. The pattern `/*` does not match `/` (2.2.5).

**2.2.9** A pattern that starts with `/` MUST match only observed paths that start with `/`.

**2.2.10** A segment that contains the one-token, is not exactly the one-token and contains no run-token is a segment pattern. Each one-token in it MUST match one or more code points other than `/`. Its other characters are literal, and the pattern is anchored at both ends of the segment. A star that is only part of a segment, and the run-token anywhere in a path, are literal (2.1.3).

**2.2.11** A matcher MUST evaluate patterns with several star segments, and segment patterns with several one-tokens, without exponential backtracking.

```yaml example:path
pattern: /etc/⋯
match: [/etc/passwd, /etc/hosts]
nomatch: [/etc, /etc/, /etc/ssh/sshd_config]
```

```yaml example:path
pattern: /etc/*
match: [/etc/passwd, /etc/ssh/sshd_config]
nomatch: [/etc, /etc/, etc/passwd]
```

```yaml example:path
pattern: /a/*/b
match: [/a/b, /a/x/b, /a/x/y/b]
nomatch: [/a/x/c]
```

```yaml example:path
pattern: /a/⋯/b
match: [/a/x/b]
nomatch: [/a/b, /a/x/y/b]
```

```yaml example:path
pattern: /x/*/*
match: [/x/y, /x/y/z]
nomatch: [/x]
```

```yaml example:path
pattern: "*"
match: [/, /etc/passwd, relative/path]
nomatch: [""]
```

```yaml example:path
pattern: /*
match: [/a, /a/b]
nomatch: [/]
```

```yaml example:path
pattern: /var/log/app
match: [/var/log/app, /var/log/app/]
nomatch: [/var/log/App, /var/log//app]
```

```yaml example:path
pattern: /tmp/x⋯
match: [/tmp/xy, /tmp/xé]
nomatch: [/tmp/x, /tmp/x/y]
```

```yaml example:path
pattern: /var/lib/kubelet/pods/⋯/volumes/kubernetes.io~projected/kube-api-access-⋯/token
match: [/var/lib/kubelet/pods/0a1b/volumes/kubernetes.io~projected/kube-api-access-7gn2x/token]
nomatch: [/var/lib/kubelet/pods/0a1b/volumes/kubernetes.io~projected/kube-api-access-/token, /var/lib/kubelet/pods/0a1b/volumes/kubernetes.io~projected/my-kube-api-access-x/token]
```

```yaml example:path
pattern: /sys/fs/cgroup/kubepods.slice/cri-containerd-⋯.scope
match: [/sys/fs/cgroup/kubepods.slice/cri-containerd-0a1b.scope]
nomatch: [/sys/fs/cgroup/kubepods.slice/cri-containerd-.scope, /sys/fs/cgroup/kubepods.slice/cri-containerd-0a1b.scopex, /sys/fs/cgroup/kubepods.slice/docker-0a1b.scope]
```

```yaml example:path
pattern: /a/⋯-⋯
match: [/a/x-y, /a/x-y-z]
nomatch: [/a/x-, /a/-y]
```

```yaml example:path
pattern: /tmp/a⋯⋯
match: []
nomatch: [/tmp/ab]
```

```yaml example:path
pattern: /tmp/*x
match: [/tmp/*x]
nomatch: [/tmp/ax]
```

```yaml example:path
pattern: /tmp/⋯⋯
match: []
nomatch: [/tmp/x, /tmp/x/y]
```

## 2.3 Argument vectors

Argument rules apply to `execs[].args`. The first element is the invoked name: the path the program was executed by, as the executing system call received it (often absolute, for example `/usr/local/bin/redis-server`), or the program's own first argument when the sensor did not observe the executing call itself and read the arguments from the started process. Both forms occur for the same program, and a producer records the form it observed.

**2.3.1** An argument pattern MUST match an observed argument vector position by position, anchored at both ends.

**2.3.2** An empty or absent pattern vector MUST match only an empty observed vector. An exec entry without `args` therefore admits only an execution without arguments.

**2.3.3** A literal argument MUST match an argument that is byte-for-byte equal to it. The star is literal everywhere in an argument vector.

**2.3.4** An argument that is exactly the one-token MUST match exactly one argument, whatever its content, including the empty argument.

**2.3.5** An argument that is exactly the run-token MUST match zero or more consecutive arguments. It MAY occur in any position and more than once.

**2.3.6** An argument that contains the one-token and is not one of the forms of 2.3.4 and 2.3.5 is an embedded pattern. It MUST be split into segments at every `/` and matched against the observed argument split the same way:

- a segment that is exactly the one-token matches exactly one segment;
- a segment that is exactly the run-token matches one or more segments;
- a segment pattern (2.2.10) matches one segment as in 2.2.10;
- every other segment is literal;
- without a run-token segment, the number of segments MUST be equal.

**2.3.7** An embedded pattern with more than one run-token segment MUST match nothing.

**2.3.8** A run-token that is only part of a segment of an argument is literal (2.1.3). A one-token that is only part of a segment makes that segment a segment pattern (2.3.6), so `--config=⋯` admits `--config=a.yaml` and `⋯:⋯` admits `app:1.0`, but neither admits a value with a `/`.

**2.3.9** A matcher MUST evaluate argument patterns with several run-tokens without exponential backtracking.

**2.3.10** The first element MUST be compared as recorded (2.4.3): `/usr/local/bin/redis-server` and `redis-server` are different first elements. An author who wants to admit both forms writes one vector for each, or one argument expression (2.11.13).

**2.3.11** A conforming engine MUST observe, as the first element of an executed vector, the pathname the executing call received, not the first argument the caller supplied: a process started with a different first argument is observed with the executed pathname first. A script started through its interpreter line is observed with the script's pathname as the first element and the interpreter as `path`; its further elements are the script's arguments. A vector read from a started process instead of the executing call (2.3) carries the process's own first argument.

```yaml example:args
pattern: [cargo, build, ⋯⋯]
match: [[cargo, build], [cargo, build, --release, -p, x]]
nomatch: [[cargo, publish]]
```

```yaml example:args
pattern: [cargo, ⋯]
match: [[cargo, build], [cargo, ""]]
nomatch: [[cargo], [cargo, build, --release]]
```

```yaml example:args
pattern: []
match: [[]]
nomatch: [[a]]
```

```yaml example:args
pattern: [⋯⋯, --x]
match: [[a, b, --x], [--x]]
nomatch: [[a, b]]
```

```yaml example:args
pattern: [ls, /plugins/*]
match: [[ls, /plugins/*]]
nomatch: [[ls, /plugins/evil.so]]
```

```yaml example:args
pattern: [node, /srv/⋯/⋯/src/⋯⋯, ⋯⋯]
match: [[node, /srv/o/r/src/x.js], [node, /srv/o/r/src/x/y/z.js, --ci]]
nomatch: [[node, /srv/o/r/src], [node, /tmp/x.js]]
```

```yaml example:args
pattern: [node, /srv/⋯/node_modules/⋯⋯/bin/x.js]
match: [[node, /srv/a/node_modules/@s/p/bin/x.js]]
nomatch: [[node, /srv/a/node_modules/bin/x.js]]
```

```yaml example:args
pattern: [x, a/⋯⋯/b/⋯⋯]
match: []
nomatch: [[x, a/1/b/2]]
```

```yaml example:args
pattern: [--build-arg, TAGS=⋯]
match: [[--build-arg, TAGS=release]]
nomatch: [[--build-arg, TAGS=], [--build-arg, TAGS=a/b]]
```

```yaml example:args
pattern: [--tag, ⋯:⋯]
match: [[--tag, app:1.0], [--tag, a:b:c]]
nomatch: [[--tag, ghcr.io/org/app:1.0], [--tag, app:]]
```

```yaml example:args
pattern: [runc, --pid-file, /run/k8s.io/⋯/⋯.pid]
match: [[runc, --pid-file, /run/k8s.io/0a1b/9f8e.pid]]
nomatch: [[runc, --pid-file, /run/k8s.io/0a1b/.pid]]
```

## 2.4 Executable names

**2.4.1** A recorded name and an invoked name MUST be the same executable name when both are non-empty and their last path elements are equal. The last path element is the part after the last `/` once trailing `/` characters are removed.

**2.4.2** An empty recorded or invoked name MUST NOT be the same executable name as any name.

**2.4.3** Whole-vector matching (2.3) MUST compare the first argument like any other argument. The executable-name rule applies where this specification names it (2.9.4).

**2.4.4** The executable-name rule MUST NOT be applied inside an argument expression (2.11). There the first element of the subject is the invoked name as observed (2.3).

```yaml example:execname
recorded: /bin/sh
same: [sh, /bin/sh, /usr/local/bin/sh]
different: [bash, "", busybox]
```

```yaml example:execname
recorded: busybox
same: [/bin/busybox]
different: [sh, nc]
```

```yaml example:args
pattern: [/bin/sh, -c, ⋯⋯]
match: [[/bin/sh, -c, echo hi]]
nomatch: [[sh, -c, echo hi]]
```

## 2.5 IP addresses and CIDR blocks

Address rules apply to `ipAddresses[]` and to the singular `ipAddress`.

**2.5.1** An entry MUST be one of: an IPv4 or IPv6 literal, a CIDR block (an entry that contains `/`), or the star.

**2.5.2** A literal entry MUST match an observed address equal to it after both are parsed. An IPv4-mapped IPv6 address MUST be treated as the IPv4 address it maps.

**2.5.3** A CIDR entry MUST match every observed address inside the block. Host bits in the entry are ignored. An IPv4-mapped IPv6 block MUST be treated as the corresponding IPv4 block.

**2.5.4** The star MUST match every parseable observed address, IPv4 or IPv6.

**2.5.5** An address with a zone suffix (`%`) is neither a valid entry nor a matchable observation.

**2.5.6** A list MUST match an observation when any entry matches it. An entry that is not one of the forms of 2.5.1 MUST contribute nothing.

**2.5.7** An empty list MUST match nothing, and an empty or unparseable observation MUST be matched by no list.

```yaml example:ip
entries: ["162.0.217.171", "2001:db8::/32"]
match: ["162.0.217.171", "2001:db8::1", "2001:0db8:0000:0000:0000:0000:0000:0001"]
nomatch: ["162.0.217.172", "2001:db9::1", ""]
```

```yaml example:ip
entries: ["10.0.0.0/8"]
match: ["10.1.2.3", "::ffff:10.1.2.3"]
nomatch: ["11.0.0.1", "not-an-address"]
```

```yaml example:ip
entries: ["*"]
match: ["192.0.2.1", "::1"]
nomatch: [""]
```

```yaml example:ip
entries: ["fe80::1%eth0"]
match: []
nomatch: ["fe80::1"]
```

## 2.6 DNS names

Name rules apply to `dnsNames[]` and to the singular `dns`.

**2.6.1** A name MUST be put in canonical form before comparison: lower-cased, one trailing `.` removed, split into labels at every `.`. A name with an empty label is invalid and matches nothing.

**2.6.2** A literal label MUST match an equal label of the canonical form.

**2.6.3** A star that is the first label of an entry with more than one label MUST match exactly one label.

**2.6.4** A star that is the last label of an entry with more than one label MUST match one or more labels. It MUST NOT match zero labels, so the apex is never matched by it.

**2.6.5** A one-token label MUST match exactly one label, in any position.

**2.6.6** An entry that is the star alone MUST match every valid name with at least one label.

**2.6.7** An entry with a label `**` is invalid and MUST match nothing.

**2.6.8** A list MUST match an observation when any entry matches it. An empty list and an empty observation MUST match nothing.

```yaml example:dns
entries: [api.example.com.]
match: [api.example.com., api.example.com, API.Example.COM.]
nomatch: [v1.api.example.com., example.com.]
```

```yaml example:dns
entries: ["*.example.com."]
match: [api.example.com., webhook.example.com.]
nomatch: [v1.api.example.com., example.com., .example.com.]
```

```yaml example:dns
entries: [svc.⋯.kubernetes.io.]
match: [svc.kube-system.kubernetes.io.]
nomatch: [svc.kubernetes.io., svc.a.b.kubernetes.io.]
```

```yaml example:dns
entries: [mycorp.com.*]
match: [mycorp.com.api., mycorp.com.api.v1.]
nomatch: [mycorp.com.]
```

```yaml example:dns
entries: ["*"]
match: [evil.example., localhost]
nomatch: [""]
```

```yaml example:dns
entries: ["*.*"]
match: [a.b.]
nomatch: [a.]
```

```yaml example:dns
entries: ["**.example.com."]
match: []
nomatch: [a.example.com.]
```

## 2.7 Ports and protocols

**2.7.1** A port MUST be identified by its port key: the protocol in upper case, `-`, and the port number in decimal (`TCP-443`).

**2.7.2** In `ingress[]` and `egress[]` entries, an absent or empty `ports` list MUST place no port constraint on the entry.

**2.7.3** A `ports[]` element without a `port` value MUST contribute no port key. An entry whose `ports` list holds only such elements admits no port.

**2.7.4** Port `0` in `ports[]` MUST be the literal port 0. It is not a wildcard.

**2.7.5** In an endpoint string of the form `:<port><path>`, port `0` MUST mean any port.

```yaml example:portkey
cases:
- {protocol: tcp, port: 443, key: TCP-443}
- {protocol: UDP, port: 0, key: UDP-0}
```

```yaml example:ports
ports: []
constraint: none
```

```yaml example:ports
ports: [{protocol: TCP}, {protocol: UDP, port: 53}]
constraint: [UDP-53]
```

```yaml example:ports
ports: [{protocol: TCP, port: 0}]
constraint: [TCP-0]
```

## 2.8 Any-sentinels

**2.8.1** In `capabilities` and `syscalls`, an entry that is the star MUST admit every non-empty observation. Every other entry MUST admit only an observation byte-for-byte equal to it.

**2.8.2** The star is the any-sentinel of `ipAddresses` (2.5.4) and of `dnsNames` (2.6.6), and it MUST be accepted wherever such a list is accepted.

**2.8.3** An empty observation MUST NOT be admitted by any sentinel, and an empty list MUST admit nothing.

```yaml example:any
entries: ["*"]
match: [CAP_SYS_ADMIN, ptrace]
nomatch: [""]
```

```yaml example:any
entries: [read, "*"]
match: [bpf]
nomatch: []
```

```yaml example:any
entries: [CAP_CHOWN]
match: [CAP_CHOWN]
nomatch: [CAP_SYS_ADMIN, cap_chown]
```

```yaml example:any
entries: []
match: []
nomatch: [read]
```

## 2.9 Allow and subtract

A document section lists what is admitted (the allow section). The `subtract` section of the same document lists what is withdrawn from it. Chapter 4 defines how the sections of several fragments are combined; this section defines the evaluation of one combined allow section and one combined subtract section.

**2.9.1** An observation MUST be admitted when an allow entry matches it and no subtract entry denies it. The order in which the entries appear MUST NOT change the outcome. An entry present in both sections is therefore not admitted.

**2.9.2** For a whole execution (path and argument vector), a subtract entry MUST deny an observation when its path matches the observed path (2.2) and its vector matches the observed vector (2.3).

**2.9.3** For an executable path regardless of its arguments, an observation MUST be admitted when an allow entry's path matches the observed path and no subtract entry denies it by 2.9.4.

**2.9.4** For an executable path, a subtract entry without an argument expression (2.11) whose path matches the observed path MUST deny the observation when its vector is empty, or when its vector has exactly two elements, the second is the run-token and the first is empty, is the same executable name as the invoked name (2.4), or matches the invoked name as a path pattern (2.2). A subtract entry with any other vector, or with an argument expression, MUST NOT deny the executable path; it denies only the executions its vector or expression matches (2.9.2, 2.11.9).

**2.9.5** The invoked name of an observed execution MUST be its first argument when that is present and non-empty, and otherwise the last path element of the observed path.

**2.9.6** For a file path, a subtract entry MUST deny an observation when its path matches the observed path (2.2).

**2.9.7** For a DNS name, a subtract entry MUST deny an observation when one of its names matches the observed name (2.6).

**2.9.8** For capabilities and syscalls, a subtract entry MUST deny an observation it admits by 2.8.1.

**2.9.9** For a file path opened by an executable, a subtract open entry with `processes` MUST deny the observation when its path matches the observed path and one of its processes matches the executable (2.2). A subtract open entry without `processes` denies the path for every executable (2.9.6).

```yaml example:subtract
surface: exec
allow: [{path: /usr/bin/getent, args: [/usr/bin/getent, hosts, ⋯]}]
subtract: [{path: /usr/bin/getent, args: [/usr/bin/getent, hosts, scan.example.invalid]}]
admitted: [{path: /usr/bin/getent, args: [/usr/bin/getent, hosts, repo.example.org]}]
denied: [{path: /usr/bin/getent, args: [/usr/bin/getent, hosts, scan.example.invalid]}]
```

```yaml example:subtract
surface: exec_path
allow: [{path: /usr/bin/tar, args: [/usr/bin/tar, ⋯⋯]}]
subtract: [{path: /usr/bin/tar, args: [/usr/bin/tar, czf, /tmp/out.tgz, ⋯⋯]}]
admitted: [{path: /usr/bin/tar, args: [/usr/bin/tar, czf, /tmp/out.tgz, x]}]
denied: []
```

```yaml example:subtract
surface: exec
allow: [{path: /usr/bin/tar, args: [/usr/bin/tar, ⋯⋯]}]
subtract: [{path: /usr/bin/tar, args: [/usr/bin/tar, czf, /tmp/out.tgz, ⋯⋯]}]
admitted: [{path: /usr/bin/tar, args: [/usr/bin/tar, -xzf, a.tgz]}]
denied: [{path: /usr/bin/tar, args: [/usr/bin/tar, czf, /tmp/out.tgz, x]}]
```

```yaml example:subtract
surface: exec_path
allow: [{path: /bin/busybox, args: ["", ⋯⋯]}]
subtract: [{path: /bin/busybox, args: [nc, ⋯⋯]}]
admitted: [{path: /bin/busybox, args: [ls, -l]}]
denied: [{path: /bin/busybox, args: [nc, -l]}, {path: /bin/busybox, args: [/usr/bin/nc, -l]}]
```

```yaml example:subtract
surface: exec_path
allow: [{path: /usr/bin/tar, args: [/usr/bin/tar, ⋯⋯]}]
subtract: [{path: /usr/bin/tar}]
admitted: []
denied: [{path: /usr/bin/tar, args: [/usr/bin/tar, -xzf, a.tgz]}]
```

```yaml example:subtract
surface: open
allow: [{path: /home/runner/*}]
subtract: [{path: /home/runner/.config/user/agent.py}]
admitted: [{path: /home/runner/work/x}]
denied: [{path: /home/runner/.config/user/agent.py}]
```

```yaml example:subtract
surface: domain
allow: [{name: repo.example.org.}, {name: "*.example.invalid."}]
subtract: [{name: scan.example.invalid.}]
admitted: [{name: repo.example.org.}, {name: api.example.invalid.}]
denied: [{name: scan.example.invalid.}]
```

## 2.10 Unmatchable entries and validation

**2.10.1** The instantiation of a path pattern replaces every segment that is exactly the one-token, the run-token or the star, and every occurrence of the one-token or run-token inside a segment, by the one-character segment `x`. The instantiation of an argument replaces the one-token and the run-token the same way and leaves the star alone. A path or argument that contains a wildcard token is unmatchable when it does not match its own instantiation. An exec vector is unmatchable when one of its arguments is, or when it does not match the vector of its instantiated arguments.

**2.10.2** A validator MUST report an empty `execs[].path` or `opens[].path` as invalid, and an unmatchable one as a dead entry. The same applies inside `subtract` and inside every per-container section.

**2.10.3** A validator MUST reject an address entry that is empty or not one of the forms of 2.5.1, including an address with a zone and a CIDR block with an out-of-range prefix length.

**2.10.4** A validator MUST reject a name entry that is empty, consists of a trailing `.` only, has an empty label, has a `**` label, or has a star label in a position other than the first or the last. The entry that is the star alone is valid.

**2.10.5** A validator MUST report an empty element of `opens[].processes` as invalid, and an element whose last path element contains the one-token, the run-token or the star as an entry that admits every executable of its directory.

```yaml example:unmatchable
paths:
  dead: [/var/log/app-⋯⋯.log, /tmp/⋯⋯, /usr/lib/llvm⋯⋯/bin/clang]
  live: [/var/log/⋯, /etc/*, "*", /plain/path, /var/log/app-⋯.log, /usr/lib/llvm-⋯/bin/clang]
args:
  dead: ["--config=⋯⋯", a/⋯⋯/b/⋯⋯]
  live: [⋯, ⋯⋯, /srv/⋯/x, /srv/⋯⋯/x, "--literal=*", "--config=⋯", "⋯:⋯"]
```

```yaml example:validate
dns:
  valid: ["*", "*.example.com.", mycorp.com.*, svc.⋯.cluster.local., "*.*", api.example.com]
  invalid: ["", ".", a..b, a.**.b, a.*.b]
ip:
  valid: ["*", "10.0.0.0/8", "2001:db8::/32", "192.0.2.1", "::1"]
  invalid: ["", "10.0.0.0/40", "300.1.1.1", "fe80::1%eth0", not-an-ip]
```

## 2.11 Argument expressions

An exec entry MAY carry `argsRegex`, a regular expression over the whole argument vector, in place of `args`. Authors write expressions for what the argument patterns of 2.3 cannot state, such as flag values, alternatives and character classes, and a learning producer may write one for an entry it generalized (2.11.11). Where both forms can state a constraint, the pattern form is portable to engines that evaluate no regular expressions (chapter 7). The subject is the argument vector with a boundary character between its elements (2.11.1), and a space in an expression stands for that boundary (2.11.14); otherwise the semantics are those of an RE2 expression in POSIX mode, matched whole.

**2.11.1** The subject of an argument expression MUST be the observed argument vector with its elements joined by the boundary character U+0000. The empty vector is the empty subject. An element never contains U+0000, since arguments of an execution are C strings, so the subject determines the vector: `[a b]` and `[a, b]` have different subjects, and `[a, ""]` has the subject `a` followed by one boundary.

**2.11.2** An argument expression MUST match only when it matches the whole subject. Anchors, alternation and newlines inside the expression or the subject do not change this.

**2.11.3** The syntax MUST be POSIX extended regular-expression syntax as RE2 accepts it in its POSIX mode. Perl classes (`\d`, `\s`, `\w`), word boundaries, groups that start with `(?`, non-greedy operators, back references, Unicode classes and `\A`, `\z`, `\Q` are not part of it; `a*?` is the optional repeat `(a*)?`. Inside an expression `^` and `$` match at line boundaries, `.` does not match a newline and a negated bracket expression does.

**2.11.4** An expression MUST NOT be longer than 16384 bytes, and its repeat counts MUST NOT exceed 1000, nested repeats multiplied. An expression that does not parse on its own, that leaves the syntax of 2.11.3 or that exceeds these bounds is invalid.

**2.11.5** An entry MUST NOT carry both `args` and `argsRegex`. An entry with `argsRegex` MUST be matched by its expression alone; `args` and `argsRequired` are ignored. An empty `argsRegex` is absent.

**2.11.6** A validator MUST report an invalid expression (2.11.4) and an entry that carries both fields (2.11.5).

**2.11.7** An allow entry whose expression is invalid MUST admit no execution. A subtract entry whose expression is invalid MUST deny every execution whose path its path matches (2.9.2) and MUST NOT deny an executable path (2.9.4).

**2.11.8** An allow entry with an expression MUST admit an execution when its path matches the observed path (2.2) and its expression matches the subject. Entries with expressions on a path place no constraint beyond their own: an execution that neither they nor another entry match is not admitted.

**2.11.9** A subtract entry with an expression MUST deny an execution when its path matches the observed path and its expression matches the subject (2.9.2). It MUST NOT deny an executable path (2.9.4).

**2.11.10** For an executable path regardless of its arguments (2.9.3), an allow entry with an expression MUST admit the path for every invoked name; the invoked name is the first element of the subject.

**2.11.11** Merging (chapter 4) and collapsing (chapter 3) MUST carry an expression unchanged, and two entries that differ only in their expression are distinct. A producer that learns from observations MUST write the argument pattern (2.3) for an entry whose arguments it did not generalize. For an entry whose pattern contains a one-token or a run-token it MAY write `argsRegex` instead, rendered from the pattern: literal text quoted, a space inside an argument written `\x20`, a one-token as `[^ ]*`, a run-token as a run of such arguments, elements separated by one space, and the first element as learned. It MUST keep the pattern when the rendered expression would not admit every observation the entry was learned from.

**2.11.12** A matcher MUST evaluate an expression in time linear in the length of the subject.

**2.11.13** The first element of the subject is the invoked name as observed (2.3), bare or with its path. An author who wants an expression to admit both forms can begin it with `(.*/)?redis-server`; a producer writes the first element as learned (2.11.11). The entry's path still decides which executable the entry admits (2.11.8).

**2.11.14** In an expression, a space (U+0020) that is not preceded by a backslash MUST denote the boundary of 2.11.1, inside a bracket expression as well. A space inside an argument MUST be written `\x20`, `\ ` (backslash, space) or with a class such as `[[:space:]]`. A negated bracket expression that excludes the space, such as `[^ ]*`, therefore matches any run inside one argument, spaces included, and never the boundary. Expressions written before version 0.2.2 that use a literal space inside an argument must write it as `\x20`.

```yaml example:argsregex
regex: redis-server --port [0-9]+
match: [[redis-server, --port, "6379"]]
nomatch: [[redis-server, --port, "6379", --daemonize, "yes"], [xredis-server, --port, "6379"], [redis-server, --port, 63a9], [redis-server, --port]]
```

```yaml example:argsregex
regex: (.*/)?redis-server --port [0-9]+
match: [[/usr/local/bin/redis-server, --port, "6379"], [redis-server, --port, "6379"]]
nomatch: [[/usr/local/bin/xredis-server, --port, "6379"], [/usr/local/bin/redis-server, --port, "6379", --daemonize, "yes"]]
```

```yaml example:argsregex
regex: redis-server --port [0-9]+
match: [[redis-server, --port, "6379"]]
nomatch: [[/usr/local/bin/redis-server, --port, "6379"]]
```

```yaml example:argsregex
regex: redis-server --port=[0-9]+( --bind=[0-9.]+)?
match: [[redis-server, --port=6379], [redis-server, --port=6379, --bind=10.0.0.7]]
nomatch: [[redis-server, "--port=6379;id"]]
```

```yaml example:argsregex
regex: a|b
match: [[a], [b]]
nomatch: [[ab], [xa], [bx]]
```

```yaml example:argsregex
regex: "redis-server --port [0-9]+"
match: []
nomatch: [[sh, -c, "id\nredis-server --port 1"]]
```

```yaml example:argsregex
regex: "sh -c [^;]*"
match: [[sh, -c, "id\nwhoami"]]
nomatch: [[sh, -c, "id;whoami"]]
```

```yaml example:argsregex
regex: "a "
match: [[a, ""]]
nomatch: [[a], ["a "]]
```

```yaml example:argsregex
regex: 'a\x20'
match: [["a "]]
nomatch: [[a, ""], [a]]
```

```yaml example:argsregex
regex: /usr/bin/chmod [^ ]* /data
match: [[/usr/bin/chmod, "0700", /data], [/usr/bin/chmod, "a b", /data]]
nomatch: [[/usr/bin/chmod, "0700 /data"], [/usr/bin/chmod, "0700", x, /data]]
```

```yaml example:argsregex
regex: ".*"
match: [[], [x, y]]
nomatch: []
```

```yaml example:argsregex
invalid: ["a)|(b", "\\d+", "(?:a)", "(?i)a", "(a)\\1", "a{1001}", "(a{11}){100}"]
```

```yaml example:subtract
surface: exec
allow: [{path: /usr/local/bin/redis-server, args: [redis-server, ⋯⋯]}]
subtract: [{path: /usr/local/bin/redis-server, argsRegex: "redis-server --port 0"}]
admitted: [{path: /usr/local/bin/redis-server, args: [redis-server, --port, "1"]}]
denied: [{path: /usr/local/bin/redis-server, args: [redis-server, --port, "0"]}]
```

```yaml example:subtract
surface: exec_path
allow: [{path: /usr/local/bin/redis-server, args: [redis-server, ⋯⋯]}]
subtract: [{path: /usr/local/bin/redis-server, argsRegex: "redis-server --port 0"}]
admitted: [{path: /usr/local/bin/redis-server, args: [redis-server, --port, "0"]}]
denied: []
```

```yaml example:subtract
surface: exec
allow: [{path: /usr/local/bin/redis-server, argsRegex: "redis-server --port [0-9]+"}]
subtract: []
admitted: [{path: /usr/local/bin/redis-server, args: [redis-server, --port, "6379"]}]
denied: [{path: /usr/local/bin/redis-server, args: [redis-server, --evil]}, {path: /usr/local/bin/redis-server, args: []}]
```

```yaml example:subtract
surface: exec
allow: [{path: /usr/local/bin/redis-server, args: [redis-server, ⋯⋯]}]
subtract: [{path: /usr/local/bin/redis-server, argsRegex: "("}]
admitted: []
denied: [{path: /usr/local/bin/redis-server, args: [redis-server]}, {path: /usr/local/bin/redis-server, args: [redis-server, --port, "1"]}]
```

## Conformance

A matcher conforms to this chapter when it:

1. gives the outcome stated for every example marked `example:` in this chapter;
2. agrees with the reference library on the conformance corpus of paths, argument vectors, argument expressions, executable names, sentinels and subtract cases whose digest the library publishes as its corpus version;
3. evaluates the adversarial patterns of 2.2.11 and 2.3.9 within polynomial time, and argument expressions in linear time (2.11.12).

A validator conforms when it reports every entry of the `dead` and `invalid` lists in the examples of 2.10 and 2.11 and none of the `live` and `valid` ones.

## Open points

- The executable-name rule (2.4) is applied by the reference library only to the executable-path evaluation of subtract entries (2.9.4). Whether the first element of a vector should be compared by name in whole-vector matching (2.3) is not decided; today it is compared literally (2.4.3).
- Whether a validator rejects unmatchable argument tokens (2.10.1) is not decided. The reference validator checks paths only.
- The reference matcher treats a star label in a middle position of a DNS entry as one label. Such entries are invalid (2.10.4); a matcher's behaviour for them is not specified.
- The meaning of absent (NULL) and explicitly empty (NONE) sections for evaluation, beyond 2.3.2, 2.7.2 and 2.8.3, is not specified. See chapter 0.
- Endpoint strings: the matching of an observed request against `endpoints[].endpoint` beyond 2.7.5 is not specified in this version.

## Changes from v0.0.4

- §5.3 said `⋯⋯` is the only argument wildcard. The one-token matches exactly one argument or one embedded segment (2.3.4, 2.3.6), and the run-token inside an argument matches one or more segments (2.3.6). §4.5 described this correctly.
- §4.5 said a token containing `⋯` is split on `/` with `⋯` as a whole segment. That remains (2.3.6, 2.3.8) and the run-token segment is added.
- The star is the any-sentinel for `capabilities`, `syscalls` and `dnsNames` as well as for `ipAddresses` (2.8). §4.3 did not allow it.
- The executable-name rule is new (2.4).
- §5.6 said an absent `port` means any port. A `ports[]` element without a `port` contributes no port (2.7.3); only an absent or empty `ports` list places no constraint (2.7.2).
- §5.8 did not state case-insensitivity; names are compared in lower case (2.6.1).
- The subtract section and its evaluation (2.9) are new.
- The definition of unmatchable entries (2.10.1) is new.
- Argument expressions (2.11) are new.
- The invoked name is defined as the path the program was executed by, or its own first argument (2.3); it is compared as recorded (2.3.10), also inside an argument expression (2.4.4, 2.11.13).
- In 0.2.2 the subject of an argument expression joins elements with U+0000 instead of a space, and a space in an expression denotes that boundary (2.11.1, 2.11.14), so a one-token rendered as `[^ ]*` can no longer span two arguments. Expressions written before v0.7.2 that use a literal space inside an argument must write it as \x20.
- A learning producer may write an argument expression for an entry it generalized (2.11.11).
