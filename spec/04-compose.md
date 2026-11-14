# 4. Composition of fragments

Status: normative.

## Scope

The document that governs a subject can be written as several fragments: a base written with the software, an admission fragment added where it is deployed, overlays added later. This chapter defines how fragments are ordered, how their sections are combined, how a container's section is taken from a grouped document, how the combined document is projected into the match sets a rule engine evaluates, and the residual of an observed document against a governing one.

Clause numbers have the form `4.<section>.<n>`.

## 4.1 Fragments

A fragment is a document (chapter 1). Its class is the value of the label `sbob/fragment-class`; its bundle is the value of the label `sbob/bundle`; its version is the value of the label `sbob/version`.

| class | role |
|---|---|
| `base` | the behaviour the software needs wherever it runs |
| `admission` | what the place of deployment adds |
| `overlay` | later additions and withdrawals |

**4.1.1** A fragment MUST carry one class from the table above when it is part of a bundle.

## 4.2 Stack order

**4.2.1** Fragments MUST be stacked by class in the order `base`, `admission`, `overlay`. A fragment whose class is not one of these stacks after all of them. Fragments of the same class keep their relative order.

**4.2.2** A composer MUST produce the same combined document whatever order the fragments arrive in. Where two fragments of the same class carry conflicting values for a keyed entry (4.3.6, 4.3.7, 4.3.8), the composer MUST break the tie by a rule that depends only on the content of the fragments.

The reference composer breaks ties by the lower-case hexadecimal SHA-256 of each fragment's JSON encoding, smaller first, and applies the class order of 4.2.1 on top of that order.

## 4.3 Combining sections

Combining takes the fragments in stack order and produces one document. "Later" means later in stack order.

**4.3.1** The combined document MUST have `kind` `RuntimeProfile` and `apiVersion` `profiles.sbob/v1beta1`. It carries no metadata; the metadata of each fragment stays with the fragment.

**4.3.2** The `subject` of the combined document MUST be the `subject` of the last fragment that states one.

**4.3.3** `architectures` MUST be the sorted union of the fragments' architectures.

**4.3.4** `capabilities` and `syscalls` MUST each be the sorted union of the fragments' entries, without duplicates.

**4.3.5** `execs`, `opens` and `endpoints` MUST each be the union of the fragments' entries in order of first appearance, an entry equal to an earlier one being dropped. Two exec entries are equal when `path`, `args`, `envs`, `argsRequired` and unknown fields are equal, an absent and an empty `args` or `envs` counting as equal. Two open entries are equal when `path`, `flags` and unknown fields are equal, an absent and a null `flags` counting as equal and an empty `flags` not. Two endpoint entries are equal when `endpoint`, `methods`, `internal`, `direction`, `headers` and unknown fields are equal, an absent and an empty `methods` counting as equal.

**4.3.6** `ingress` and `egress` entries MUST each be combined by `identifier`: a later entry replaces the whole earlier entry with the same identifier and takes its position; an entry with a new identifier is appended. Entries without an identifier share the empty key, so a later one replaces an earlier one, also within one fragment.

**4.3.7** `rulePolicies` MUST be combined by rule name: a later policy replaces the whole earlier policy for the same rule.

**4.3.8** `imageID` and `imageTag` MUST each be the last non-empty value.

**4.3.9** `seccompProfile` MUST be the last value that is not empty. A value is empty when it is absent, null, or an object whose members are all empty strings, zero, false, null or empty objects. A list, even an empty one, is not empty.

**4.3.10** `identifiedCallStacks` MUST be the union of the fragments' elements in order of first appearance, an element equal as a JSON value to an earlier one being dropped. A value that is not a list MUST be refused.

**4.3.11** The `subtract` section of the combined document MUST be combined from the fragments' `subtract` sections by 4.3.4 to 4.3.6. It MUST be present when at least one fragment carries a `subtract` object, and absent otherwise.

**4.3.12** `matchLabels` MUST be combined per key, a later value replacing an earlier one. `matchExpressions` MUST be the concatenation of the fragments' lists.

**4.3.13** A field unknown to the composer MUST be combined per key, a later value replacing an earlier one.

**4.3.14** Per-container sections MUST be combined per list (`containers`, `initContainers`, `ephemeralContainers`) and per name, each pair of sections by 4.3.4 to 4.3.13, in order of first appearance of the name.

**4.3.15** Combining MUST be associative: combining the fragments in steps gives the same document as combining them at once.

**4.3.16** Combining a document with itself MUST give the same document as combining it alone, except for `matchExpressions`, which 4.3.12 concatenates.

**4.3.17** An observation MUST be admitted by the combined document exactly when the union of the fragments' allow sections admits it and the union of their subtract sections does not deny it (2.9), whatever the order of the fragments. A subtract entry in any fragment withdraws a matching allow entry of any other fragment.

```yaml example:merge
fragments:
- class: overlay
  spec:
    capabilities: [CHOWN]
    execs: [{path: /bin/backup, args: [backup]}]
    egress:
    - {identifier: db, ipAddresses: [10.0.0.5], ports: [{name: TCP-5433, protocol: TCP, port: 5433}]}
- class: base
  spec:
    architectures: [amd64]
    capabilities: [NET_BIND_SERVICE]
    execs: [{path: /bin/server, args: [server]}]
    egress:
    - {identifier: db, ipAddresses: [10.0.0.5], ports: [{name: TCP-5432, protocol: TCP, port: 5432}]}
    - {identifier: cache, ipAddresses: [10.0.0.6]}
    rulePolicies:
      namespace-change: {processAllowed: [runc]}
spec:
  architectures: [amd64]
  capabilities: [CHOWN, NET_BIND_SERVICE]
  execs:
  - {path: /bin/server, args: [server]}
  - {path: /bin/backup, args: [backup]}
  egress:
  - {identifier: db, ipAddresses: [10.0.0.5], ports: [{name: TCP-5433, protocol: TCP, port: 5433}]}
  - {identifier: cache, ipAddresses: [10.0.0.6]}
  rulePolicies:
    namespace-change: {processAllowed: [runc]}
```

```yaml example:merge
fragments:
- class: base
  spec:
    execs: [{path: /usr/bin/curl, args: [curl, ⋯⋯]}]
    subtract:
      execs: [{path: /usr/bin/curl, args: [curl, --insecure, ⋯⋯]}]
- class: overlay
  spec:
    subtract:
      execs: [{path: /usr/bin/curl, args: [curl, -k, ⋯⋯]}]
      syscalls: [ptrace]
spec:
  execs: [{path: /usr/bin/curl, args: [curl, ⋯⋯]}]
  subtract:
    execs:
    - {path: /usr/bin/curl, args: [curl, --insecure, ⋯⋯]}
    - {path: /usr/bin/curl, args: [curl, -k, ⋯⋯]}
    syscalls: [ptrace]
```

```yaml example:merge
fragments:
- class: base
  spec:
    egress:
    - {ipAddresses: [192.0.2.1]}
    - {ipAddresses: [192.0.2.2]}
spec:
  egress:
  - {ipAddresses: [192.0.2.2]}
```

## 4.4 Section of a container

**4.4.1** For a document that is not grouped (1.5.1), the section of every container MUST be the whole `spec`, whatever the container's name and kind.

**4.4.2** For a grouped document, the section of a container MUST be the per-container section with the container's name in the list of the container's kind (`containers` for a regular container, `initContainers` for an init container, `ephemeralContainers` for an ephemeral container). It MUST carry `architectures`, `matchLabels` and `matchExpressions` of `spec`, and nothing else of `spec`: neither its behaviour sections nor its `subtract`.

**4.4.3** When the kind of the container is not known, the lists MUST be searched in the order `containers`, `initContainers`, `ephemeralContainers`, and the first section with the name is the section.

**4.4.4** When a grouped document has no section for a container, the container MUST have no section in that document. A verifier MUST NOT take the section of another container in its place.

```yaml example:section
spec:
  architectures: [amd64]
  egress: [{identifier: api, ipAddresses: [10.0.0.5]}]
  subtract: {syscalls: [ptrace]}
  containers:
  - {name: app, execs: [{path: /bin/app}], subtract: {execs: [{path: /bin/app}]}}
  - {name: shared, execs: [{path: /bin/shared-regular}]}
  initContainers:
  - {name: shared, execs: [{path: /bin/shared-init}]}
cases:
- container: app
  kind: container
  section:
    architectures: [amd64]
    execs: [{path: /bin/app}]
    subtract: {execs: [{path: /bin/app}]}
- container: shared
  kind: initContainer
  section:
    architectures: [amd64]
    execs: [{path: /bin/shared-init}]
- container: shared
  kind: ""
  section:
    architectures: [amd64]
    execs: [{path: /bin/shared-regular}]
- container: sidecar
  kind: ""
  section: null
```

## 4.5 Projection

A rule engine evaluates a section through its projection: one match set per field, and the same for the subtract section.

**4.5.1** For `opens[].path`, `execs[].path` and `endpoints[].endpoint`, an entry that contains the one-token or the star MUST be projected as a pattern and every other entry as an exact value. Patterns MUST be deduplicated.

**4.5.2** `capabilities`, `syscalls`, DNS names and addresses MUST be projected as values, whatever characters they contain; their sentinels and patterns are interpreted when matching (2.5, 2.6, 2.8).

**4.5.3** The projection MUST keep, for every exec path, the list of argument vectors of the entries with that path, and for every open path, the sorted union of the flags of the entries with that path (an empty set for entries without flags).

**4.5.4** For each direction, the projection MUST derive from the entries that do not carry a reserved identifier (1.4.9): the DNS names (`dns`, then `dnsNames`), the addresses (`ipAddress`, then `ipAddresses`), for every entry with addresses its addresses with its port keys, and for every entry with names its names, addresses and port keys.

**4.5.5** For each direction, the projection MUST keep, for every entry with a `podSelector`, its pod selector, its namespace selector and its port keys.

**4.5.6** For each direction, the entries with the identifier `host-entity` or `host-entity-resolved` MUST form the host-peer set of that direction (addresses with port keys). The egress entries with the identifier `host-entity-claimed` MUST form the claimed host set, taken from the allow section only.

**4.5.7** Port keys MUST be computed by 2.7. An entry without `ports` or with an empty `ports` list has no port constraint; its port-key set is absent, not empty.

**4.5.8** `rulePolicies` MUST be carried into the projection unchanged.

**4.5.9** The subtract section MUST be projected by 4.5.1 to 4.5.7 into a separate projection, which is absent when the section has no `subtract`.

**4.5.10** A projection MUST NOT share mutable state with the document it was taken from: changing one never changes the other.

```yaml example:project
spec:
  opens:
  - {path: /etc/passwd, flags: [O_RDONLY]}
  - {path: /etc/passwd, flags: [O_CLOEXEC, O_RDONLY]}
  - {path: /var/log/*}
  egress:
  - {identifier: api, dnsNames: [api.example.com.], ipAddresses: [192.0.2.7], ports: [{protocol: TCP, port: 443}]}
  - {identifier: host-entity, ipAddresses: [192.0.2.1], ports: [{protocol: TCP, port: 10250}]}
opens:
  values: [/etc/passwd]
  patterns: [/var/log/*]
openFlags:
  /etc/passwd: [O_CLOEXEC, O_RDONLY]
  /var/log/*: []
egressDomains: [api.example.com.]
egressAddresses: [192.0.2.7]
egressHost: [{addresses: [192.0.2.1], ports: [TCP-10250]}]
```

## 4.6 Residual

The residual of an observed section against a governing section is what the observed section holds that the governing one does not admit. Tooling uses it to propose additions.

**4.6.1** The residual MUST contain every exec entry of the observed section that no allow entry of the governing section covers and no subtract entry denies. An allow entry covers it when its path matches the observed entry's path (2.2) and its vector matches the observed entry's vector (2.3), the observed strings read as observations. A subtract entry denies it when its path matches and its vector is empty or matches.

**4.6.2** The residual MUST contain every open entry of the observed section whose path no allow open entry matches and no subtract open entry matches. Flags are not compared.

**4.6.3** The residual MUST contain every syscall and every capability of the observed section that is neither listed in the governing section nor in its subtract section.

**4.6.4** The residual MUST NOT share mutable state with either input.

## Conformance

A composer conforms to this chapter when it reproduces the examples marked `example:` in this chapter, meets the laws 4.3.15 to 4.3.17 on any input, and agrees with the reference composer on the conformance vectors for combining, section resolution and projection.

## Open points

- 4.3.6 replaces an entry without an identifier by the next one without an identifier, also within one fragment, so a fragment with two such entries keeps only the last. Producers SHOULD give every network entry an identifier until this is decided.
- The replacement of a fragment by a newer version of itself (same bundle, class and name, higher `sbob/version`) is carried out by the verifier and is not specified in this version; see chapter 5.
- The residual covers execs, opens, syscalls and capabilities. Network sections and endpoints are not part of it in this version.
- The tie-breaking rule of the reference composer (4.2.2) depends on the exact JSON encoding of a fragment. A second implementation that wants identical results under conflicting same-class fragments needs that encoding; it is not specified here.

## Changes from v0.0.4

v0.0.4 had no composition. Everything in this chapter is new.
