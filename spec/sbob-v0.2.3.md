<div class="spec-doc" markdown="1">

<dl class="status">
  <dt>Document</dt><dd>Software Bill of Behavior</dd>
  <dt>Version</dt><dd>0.2.3</dd>
  <dt>Stage</dt><dd>Proposed</dd>
</dl>

This file is the text of version 0.2.3 in one document, generated from the chapters under `spec/`. The chapters are the source; change them, not this file.

## Contents

| chapter | stage | source |
|---|---|---|
| 0. Introduction and conformance | Normative | `00-introduction-and-conformance.md` |
| 1. The behaviour document | Normative | `01-document.md` |
| 2. Pattern and matching grammar | Normative | `02-grammar.md` |
| 3. Collapse of learned documents | Normative | `03-collapse.md` |
| 4. Composition of fragments | Normative | `04-compose.md` |
| 5. Trust | Draft | `05-trust.md` |
| 6. Rules | Normative | `06-rules.md` |
| 7. Portability to other engines | Normative | `07-portability.md` |
| 8. Wire-level intent | Draft | `08-wire-intent.md` |
| Appendix A. Regular-expression rendering of patterns | Normative | `appendix-a-regex.md` |

## Changes since 0.0.4

### 0. Introduction and conformance

- §1.3 is 0.2.1, extended to the full BCP 14 key word list.
- §2 named two conformance targets, a document and a verifier. There are five classes now (0.5), each tied to chapters.
- §2 required a document to describe exactly one pod with all container types and to populate `spec.imageID`. Chapter 1 defines what a document requires.
- §2 required a verifier to handle NULL and NONE differently and to report its NULL posture at start-up. In this version both admit nothing (0.4.2); see Open points.
- §2 required drift events keyed by generalized rule names; chapter 6 defines rule names.
- §1.4 moved to chapters 6 and 7.
- Terms (0.3), versions (0.6) and the chapter map (0.7) are new.

### 1. The behaviour document

- One kind, `RuntimeProfile`, with a `subject` field (`container`, `host`) replaces `ContainerProfile`. The API group is `profiles.sbob`. The legacy form stays readable (1.8).
- Labels and annotations use the prefix `sbob/`. `kubescape.io/managed-by` became `sbob/managed-by`, read from the annotation only with the exact value `User` (1.2.3); `sbob.io/spec-version` became `sbob/spec-version`.
- §4.2 said there is no per-container list. Grouped documents with `containers`, `initContainers` and `ephemeralContainers` are part of the document (1.5).
- §2 and §4.1 required `metadata.namespace` and `spec.imageID`. Only `metadata.name`, `kind` and `subject` are required (1.1).
- §4.6 required `flags`; it is optional (1.4.2).
- §4.7 required `ports[]`; it is optional and an absent list places no port constraint (2.7.2).
- §4.4 described `headers` as a map of header name to value pattern; it is a map of header name to a list of values (1.4.7).
- The subtract section (1.6), status and completion (1.7), the legacy mapping (1.8), the encoding rules (1.9), document names (1.10), binding (1.11), identity (1.12) and learned documents (1.13) are new.
- The fields `syscalls`, `imageTag`, `seccompProfile`, `identifiedCallStacks`, `envs`, `argsRequired`, `internal`, `serviceRefNamespace`, `serviceRefName`, `serviceSelector` and `entity` were not described in v0.0.4.

### 2. Pattern and matching grammar

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

### 3. Collapse of learned documents

v0.0.4 had no text on collapse. Everything in this chapter is new.

### 4. Composition of fragments

v0.0.4 had no composition. Everything in this chapter is new.

### 5. Trust

v0.0.4 §8 was marked work in progress and carried no text. Everything in this chapter is new.

### 6. Rules

- §1.4 named 22 generalized rules; the catalogue names every rule of the reference rule set (6.5), new names marked proposed.
- §1.4.1 and §1.4.2 compared four tools in columns; that comparison is replaced by chapter 7.
- §4.8 keyed `rulePolicies` by generalized rule name; the implemented key is the engine identifier (6.7.1).
- The profile functions (6.3), the deny-all projection (6.6), context tags (6.8), bindings (6.9) and event-loss reporting (6.10) are new.
- v0.0.4 said the network stanzas mirror NetworkPolicy; selector evaluation differs from it in the three points after 6.3.16.

### 7. Portability to other engines

- §1.4 compared four tools in unverified columns. Those columns are removed; portability is stated per rule as a class (7.4) and per engine only once proven.
- The fidelity classes (7.1), the portable subset (7.2) and the projection model (7.3) are new.

### Appendix A. Regular-expression rendering of patterns

v0.0.4 had no rendering appendix. Everything here is new.

---

# 0. Introduction and conformance

Status: normative.

## 0.1 Purpose

A Software Bill of Behaviour (SBoB) is a declarative document that states what a piece of software is intended to do at run time: which executables it starts and with which arguments, which files it opens, which system calls and Linux capabilities it uses, which HTTP endpoints it serves or calls, and which network peers it accepts or contacts. A verifier compares what it observes with the document and reports what the document does not admit.

| other artifact | what it covers | relation to an SBoB |
|---|---|---|
| software bill of materials | static composition: packages, versions, licences | complementary |
| vulnerability exploitability statement | whether a known vulnerability is exploitable | complementary |
| seccomp or AppArmor profile | kernel-enforced system call and resource policy | adjacent; a document MAY carry a seccomp profile (chapter 1) |

The specification is independent of any implementation. The library `github.com/k8sstormcenter/sbob-go` is the reference implementation; where this text names one of its functions or constants, the name helps the reader and is not a requirement.

## 0.2 Conformance language

**0.2.1** The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY and OPTIONAL in this specification are to be interpreted as described in BCP 14 (RFC 2119, RFC 8174) when, and only when, they appear in all capitals.

Normative statements are numbered clauses `<chapter>.<section>.<n>`. Text outside a numbered clause, tables included, explains the clauses and adds no requirement unless a clause refers to it. Examples marked `example:<kind>` are conformance material: what they state holds for the reference implementation.

## 0.3 Terms

| term | meaning |
|---|---|
| document | a `RuntimeProfile` object (chapter 1) |
| subject | what a document describes: a container, the containers of one workload, or a host |
| section | one list or map of a document: `execs`, `opens`, `capabilities`, `syscalls`, `endpoints`, `ingress`, `egress`, `rulePolicies` |
| allow section | a section directly under `spec` or under a per-container section |
| subtract section | the `subtract` object and the sections in it (1.6) |
| entry | one element of a section |
| pattern | an entry that uses a token of chapter 2 |
| observation | an event of the subject as a sensor reports it: an execution, an open, a DNS answer, a connection |
| admit | an observation is admitted when the allow sections match it and the subtract sections do not (2.9) |
| authored document | a document marked as written by a person (1.2.3) |
| learned document | a document recorded from observations (1.2.4) |
| fragment | one of several documents that together make up the document of a subject (chapter 4) |
| fragment class | the role of a fragment: `base`, `admission` or `overlay` |
| bundle | a set of fragments published together (chapter 5) |
| collapse | the generalization of a learned document (chapter 3) |
| projection | the per-field match sets a verifier derives from a document (4.5) |
| rule | a named check that a rule engine evaluates on observations (chapter 6) |
| profile function | a function a rule calls to ask whether a document admits an observation (6.3) |
| deny-all projection | a projection under which no profile function admits anything (6.6) |

## 0.4 Absent and explicitly empty sections

An absent section is NULL; a section present with an empty value is NONE.

**0.4.1** A document MUST keep NULL and NONE apart when it is read, stored and written (1.9.2).

**0.4.2** In this version, a NULL and a NONE allow section MUST both admit nothing, with two exceptions: an exec entry without `args` and one with `args: []` both admit only the execution without arguments (2.3.2), and an absent and an empty `ports` list both place no port constraint (2.7.2).

**0.4.3** A NULL and a NONE subtract section MUST both withdraw nothing.

## 0.5 Conformance classes

An implementation claims conformance for one or more classes. Each class names the chapters it has to meet.

| class | what it does | chapters |
|---|---|---|
| document producer | writes documents, by hand, by tooling or by learning | 1; 3 when it collapses |
| matcher | decides whether an entry matches an observation | 2; appendix A when it uses regular expressions |
| composer | combines fragments into one document and projects it | 4 |
| verifier | decides which document governs a subject and checks its trust | 1.2, 1.11, 5 |
| rule engine | evaluates rules on observations against a projection | 6; 7 when it is not the reference engine |

**0.5.1** An implementation that claims a class MUST meet every clause of the chapters the class names, and the conformance section of each of those chapters.

**0.5.2** An implementation MUST state the version of this specification it conforms to.

## 0.6 Versions

**0.6.1** This specification is versioned with semantic versions. The version of this text is `0.2.3`. The registry carries its own `spec_version`: the version of the text in which its keys and values last changed, `0.1.0`.

**0.6.2** A document SHOULD carry the version it conforms to in the annotation `sbob/spec-version` (1.2.5), and MAY carry the grammar version it was authored against in the label `sbob/matcher-version`.

**0.6.3** A container image of a conforming implementation SHOULD carry the OCI labels `sbob.spec-version` (the specification version it implements) and `sbob.library-version` (the version of the reference library it was built against, if any).

**0.6.4** A change of matching or collapse semantics in the reference library is a new minor version at least. The library publishes, as its corpus version, the SHA-256 digest of the conformance corpus it is proven against; two implementations with the same corpus version agree on every case of that corpus.

## 0.7 Chapters

| chapter | content |
|---|---|
| 0 | this introduction, terms, NULL and NONE, conformance classes, versions |
| 1 | the document, its legacy form and its identity |
| 2 | the pattern grammar and matching |
| 3 | collapse of learned documents |
| 4 | composition of fragments and projection |
| 5 | trust (draft) |
| 6 | rules and profile functions |
| 7 | portability to other engines |
| 8 | wire-level intent (draft) |
| A | regular-expression rendering |

The registry `conventions/conventions.yaml` lists every key, value, kind, rule name and profile function this specification uses. The schemas in `schema/` describe the document, the bundle manifest and the alert stream.

## Conformance

An implementation conforms to this specification when it conforms to every class it claims (0.5.1) and states the specification version (0.5.2).

## Open points

- v0.0.4 required a verifier to treat NULL and NONE differently (a NONE section a hard violation on the first observation, a NULL section an implementation-defined posture reported at start-up). The reference implementation evaluates both alike (0.4.2). Whether a verifier posture for NULL sections is reintroduced is not decided.
- How the specification version moves with the library version is not fixed beyond 0.6.4.

## Changes from v0.0.4

- §1.3 is 0.2.1, extended to the full BCP 14 key word list.
- §2 named two conformance targets, a document and a verifier. There are five classes now (0.5), each tied to chapters.
- §2 required a document to describe exactly one pod with all container types and to populate `spec.imageID`. Chapter 1 defines what a document requires.
- §2 required a verifier to handle NULL and NONE differently and to report its NULL posture at start-up. In this version both admit nothing (0.4.2); see Open points.
- §2 required drift events keyed by generalized rule names; chapter 6 defines rule names.
- §1.4 moved to chapters 6 and 7.
- Terms (0.3), versions (0.6) and the chapter map (0.7) are new.

---

# 1. The behaviour document

Status: normative.

## Scope

This chapter defines the behaviour document `RuntimeProfile`: its envelope, its subject, its sections, the per-container sections of a grouped document, the subtract section, rule policies, status and completion, the labels and annotations it carries, the legacy `ContainerProfile` form and the lossless mapping between the two, and the identity of a document: the name of a learned document and the workload identifier.

Clause numbers have the form `1.<section>.<n>`. Field tables list every field of each object; a field not listed is an unknown field (1.9). The registry of keys and values is `conventions/conventions.yaml` in this repository.

## 1.1 Envelope

A document is one JSON or YAML object.

<!-- fields: RuntimeProfile -->

| field | type | description |
|---|---|---|
| `apiVersion` | string | `profiles.sbob/v1beta1` |
| `kind` | string | `RuntimeProfile` |
| `metadata` | object | see 1.2 |
| `subject` | string | what the document describes, see 1.1.3 |
| `spec` | object | the behaviour sections, see 1.3 |
| `status` | object | carried unchanged; no field of it is defined by this specification |

**1.1.1** A document MUST have `kind` `RuntimeProfile` and `apiVersion` `profiles.sbob/` followed by a version. The version of this specification is `v1beta1`.

**1.1.2** A document MUST have a non-empty `metadata.name`.

**1.1.3** A document MUST have a `subject`. Its value MUST be `container` for a document that describes one container or the containers of one workload, and `host` for a document that describes a host. Other values are reserved for later versions and MUST be rejected by a validator of this version.

**1.1.4** A consumer MUST carry `status` unchanged.

## 1.2 Metadata

<!-- fields: Metadata -->

| field | type | description |
|---|---|---|
| `name` | string | the document name, see 1.10 |
| `namespace` | string | namespace of the workload the document belongs to |
| `labels` | map of string to string | keys from the registry `labels` list, or any other key |
| `annotations` | map of string to string | keys from the registry `annotations` list, or any other key |

**1.2.1** Every label and annotation key this specification gives a meaning to MUST be taken from the registry. A producer MUST write the current key (prefix `sbob/`). A consumer SHOULD also accept the legacy key the registry pairs with it and MUST give both the same meaning.

**1.2.2** A key the registry lists under `retired` MUST NOT be given a meaning.

**1.2.3** A document is authored when it carries the annotation `sbob/managed-by` with the exact value `User`. A label of that key, or another spelling or case of the value, MUST NOT make a document authored.

**1.2.4** A document that is not authored is learned. A learned document MUST NOT govern a container or a host: a verifier never takes it as the document it evaluates rules against.

**1.2.5** A producer SHOULD write the annotation `sbob/spec-version` with the version of this specification the document conforms to.

The keys that the chapters of this specification use are:

| key | kind | legacy key | meaning |
|---|---|---|---|
| `sbob/user-defined-profile` | label on a Pod or Node | `kubescape.io/user-defined-profile` | binding value, see 1.11 |
| `sbob/managed-by` | annotation | `kubescape.io/managed-by` | 1.2.3 |
| `sbob/status` | annotation | `kubescape.io/status` | learning status, 1.7 |
| `sbob/completion` | annotation | `kubescape.io/completion` | completion, 1.7 |
| `sbob/profile-role` | label | `kubescape.io/profile-role` | `shadow` or `suggestion` |
| `sbob/spec-version` | annotation | `sbob.io/spec-version` | 1.2.5 |
| `sbob/subject-type` | label | `kubescape.io/sbom-type` | legacy identity of a host document, 1.8 |
| `sbob/instance-template-hash` | label | `kubescape.io/instance-template-hash` | workload template hash of a learned document |

## 1.3 Sections

The behaviour sections sit directly under `spec`. A document with any per-container list is grouped (1.5).

<!-- fields: Spec -->

| field | type | description |
|---|---|---|
| `architectures` | list of string | Linux architecture names the document applies to (`amd64`, `arm64`) |
| `capabilities` | list of string | Linux capabilities (`CAP_NET_BIND_SERVICE`), or the star (2.8) |
| `execs` | list of exec entries | 1.4 |
| `opens` | list of open entries | 1.4 |
| `syscalls` | list of string | system call names, or the star (2.8) |
| `endpoints` | list of endpoint entries | 1.4 |
| `ingress` | list of network entries | peers that open connections to the subject, 1.4 |
| `egress` | list of network entries | peers the subject opens connections to, 1.4 |
| `seccompProfile` | object | a seccomp profile carried with the document |
| `imageID` | string | the image the subject runs, by digest where known |
| `imageTag` | string | the image tag the subject runs |
| `rulePolicies` | map of rule name to policy | 1.4 |
| `identifiedCallStacks` | list of objects | call stacks carried with the document |
| `subtract` | object | 1.6 |
| `matchLabels` | map of string to string | a label selector carried with the document |
| `matchExpressions` | list of selector requirements | a label selector carried with the document |
| `containers` | list of sections | per-container sections for regular containers, 1.5 |
| `initContainers` | list of sections | per-container sections for init containers, 1.5 |
| `ephemeralContainers` | list of sections | per-container sections for ephemeral containers, 1.5 |

**1.3.1** Every section is optional. A producer SHOULD omit a section it has nothing to say about, and SHOULD write an empty list only when it means that nothing of that kind is intended (0.4).

**1.3.2** `capabilities` and `syscalls` entries are names compared byte for byte (2.8.1).

## 1.4 Entries

#### Exec entries

<!-- fields: ExecCall -->

| field | type | description |
|---|---|---|
| `path` | string | path pattern of the executable (2.2) |
| `args` | list of string | argument-vector pattern, the first element is the invoked name (2.3) |
| `envs` | list of string | environment entries recorded with the execution |
| `argsRequired` | boolean | legacy marker of an anchored argument vector, see Open points |
| `argsRegex` | string | regular expression over the whole argument vector, in place of `args` (2.11) |

**1.4.1** An exec entry MUST have a non-empty `path`.

**1.4.2** An exec entry admits an execution when its `path` matches the executable path and its `args` match the argument vector (2.3). An entry without `args` and without `argsRegex` admits only an execution without arguments (2.3.2). An entry with `argsRegex` is matched by its expression (2.11).

**1.4.3** Matching MUST NOT depend on `envs`.

#### Open entries

<!-- fields: OpenCall -->

| field | type | description |
|---|---|---|
| `path` | string | path pattern of the file (2.2) |
| `flags` | list of string | open flags (`O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_CREAT`, `O_CLOEXEC`, ...) |

**1.4.4** An open entry MUST have a non-empty `path`.

**1.4.5** The flags of all entries with the same `path` form one flag set for that path.

#### Endpoint entries

<!-- fields: Endpoint -->

| field | type | description |
|---|---|---|
| `endpoint` | string | `:<port><path>`; port `0` means any port (2.7.5) |
| `methods` | list of string | HTTP methods (`GET`, `POST`) |
| `internal` | boolean | the peer is inside the cluster |
| `direction` | string | `inbound` or `outbound` |
| `headers` | object | header name to list of values |
| `extensions` | object | wire-level intent of the entry, see 1.4.14 |

**1.4.6** `direction` MUST be `inbound` or `outbound` when present.

**1.4.7** `headers` MUST be a JSON object whose values are lists of strings.

#### Network entries

`ingress[]` and `egress[]` carry entries of the same shape.

<!-- fields: NetworkNeighbor -->

| field | type | description |
|---|---|---|
| `identifier` | string | key of the entry, see 1.4.8 |
| `type` | string | `internal` or `external` |
| `dns` | string | one DNS name pattern; legacy singular form of `dnsNames` |
| `dnsNames` | list of string | DNS name patterns (2.6) |
| `ports` | list of ports | port constraint (2.7) |
| `podSelector` | label selector | peer pods by label |
| `namespaceSelector` | label selector | peer namespaces by label |
| `ipAddress` | string | one address entry; legacy singular form of `ipAddresses` |
| `ipAddresses` | list of string | address entries (2.5) |
| `serviceRefNamespace` | string | namespace of a referenced service |
| `serviceRefName` | string | name of a referenced service |
| `serviceSelector` | label selector | a selector of services |
| `entity` | string | a named peer class, see Open points |
| `extensions` | object | wire-level intent of the entry, see 1.4.14 |

<!-- fields: NetworkPort -->

| field | type | description |
|---|---|---|
| `name` | string | conventionally the port key (`TCP-443`) |
| `protocol` | string | `TCP`, `UDP` or `SCTP` |
| `port` | integer or null | the port number |

<!-- fields: LabelSelector -->

| field | type | description |
|---|---|---|
| `matchLabels` | map of string to string | labels a peer must carry |
| `matchExpressions` | list of requirements | set-based requirements |

<!-- fields: LabelSelectorRequirement -->

| field | type | description |
|---|---|---|
| `key` | string | label key |
| `operator` | string | `In`, `NotIn`, `Exists` or `DoesNotExist` |
| `values` | list of string | values for `In` and `NotIn` |

**1.4.8** Within one section, `identifier` is the key of a network entry: entries are combined and replaced by it (4.3.6). Entries without an identifier share the empty key.

**1.4.9** The identifiers `host-entity`, `host-entity-resolved` and `host-entity-claimed` are reserved. Entries carrying them describe the host peer of the subject and are projected apart from the declared peers (4.5.6). A producer MUST NOT use them for any other entry.

**1.4.10** A consumer MUST read both the singular and the plural address field, the singular first, and both the singular and the plural name field, the singular first. A producer SHOULD write only the plural fields.

**1.4.11** Address entries MUST follow 2.5 and name entries MUST follow 2.6.

#### Rule policies

`rulePolicies` maps a rule name (chapter 6) to a policy.

<!-- fields: RulePolicy -->

| field | type | description |
|---|---|---|
| `processAllowed` | list of string | process names (`comm`) that are intended to perform the action |
| `containerAllowed` | boolean | any process of the subject is intended to perform the action |

**1.4.12** `containerAllowed: false` MUST have the same meaning as an absent `containerAllowed`.

**1.4.13** A policy for a rule the engine does not honour MUST have no effect and MUST NOT be treated as a denial.

#### Extensions

Endpoint and network entries, at the top level, in container sections and in `subtract`, MAY carry `extensions`. Its keys describe wire-level intent (protocol, peer, requirements, TLS parameters); their registry is `conventions/conventions.yaml`.

**1.4.14** `extensions` MUST be a JSON object. A consumer that does not evaluate it MUST keep it and write it back unchanged (1.9.1).

**1.4.15** `extensions` MUST NOT change how a consumer that does not evaluate it matches the entry: two entries that differ only in `extensions` describe the same behaviour.

```yaml
spec:
  rulePolicies:
    namespace-change:
      processAllowed: [runc]
    service-account-token-access:
      containerAllowed: true
```

## 1.5 Grouped documents

A grouped document describes several containers of one workload. Each per-container section has a name and the same sections as `spec`, except `architectures`, `matchLabels`, `matchExpressions` and the per-container lists.

<!-- fields: Container -->

| field | type | description |
|---|---|---|
| `name` | string | container name |
| `capabilities` | list of string | as in 1.3 |
| `execs` | list of exec entries | as in 1.3 |
| `opens` | list of open entries | as in 1.3 |
| `syscalls` | list of string | as in 1.3 |
| `endpoints` | list of endpoint entries | as in 1.3 |
| `ingress` | list of network entries | as in 1.3 |
| `egress` | list of network entries | as in 1.3 |
| `seccompProfile` | object | as in 1.3 |
| `imageID` | string | as in 1.3 |
| `imageTag` | string | as in 1.3 |
| `rulePolicies` | map of rule name to policy | as in 1.3 |
| `identifiedCallStacks` | list of objects | as in 1.3 |
| `subtract` | object | as in 1.6, for this container only |

**1.5.1** A document is grouped when at least one of `containers`, `initContainers` and `ephemeralContainers` is non-empty.

**1.5.2** Every per-container section MUST have a non-empty `name`. The same name MAY appear in two lists; it then names two different containers.

**1.5.3** In a grouped document, the behaviour sections directly under `spec` MUST NOT be evaluated for any container. Each container is evaluated against its own section only (4.4).

```yaml
apiVersion: profiles.sbob/v1beta1
kind: RuntimeProfile
subject: container
metadata:
  name: web
  annotations: {sbob/managed-by: User}
spec:
  architectures: [amd64]
  containers:
  - name: app
    execs: [{path: /bin/app, args: [/bin/app]}]
  initContainers:
  - name: setup
    execs: [{path: /bin/setup, args: [/bin/setup]}]
```

## 1.6 The subtract section

<!-- fields: Subtract -->

| field | type | description |
|---|---|---|
| `capabilities` | list of string | as in 1.3 |
| `execs` | list of exec entries | as in 1.3 |
| `opens` | list of open entries | as in 1.3 |
| `syscalls` | list of string | as in 1.3 |
| `endpoints` | list of endpoint entries | as in 1.3 |
| `ingress` | list of network entries | as in 1.3 |
| `egress` | list of network entries | as in 1.3 |

**1.6.1** `subtract` lists entries that are withdrawn from the allow sections of the same section level. Its entries have the shape of the allow entries and MUST obey the same grammar and validation.

**1.6.2** An observation matched by a subtract entry MUST NOT be admitted, whatever the allow sections say and in whichever fragment either entry was written (2.9, 4.3.17).

```yaml
spec:
  execs:
  - {path: /usr/bin/curl, args: [curl, ⋯⋯]}
  subtract:
    execs:
    - {path: /usr/bin/curl, args: [curl, --insecure, ⋯⋯]}
```

## 1.7 Status and completion

A learned document carries its learning state in two annotations.

| annotation | values |
|---|---|
| `sbob/status` | `initializing`, `ready`, `completed`, `failed`, `too-large`, `incomplete`, `missing-runtime` |
| `sbob/completion` | `partial`, `complete` |

**1.7.1** A producer MUST write only the values listed above in these annotations.

## 1.8 The legacy form

The legacy form of a document has `kind` `ContainerProfile` and `apiVersion` `spdx.softwarecomposition.kubescape.io/<version>`, has no `subject` field, and marks a host document with the legacy label `kubescape.io/sbom-type: node`.

**1.8.1** To read a legacy document, a consumer MUST replace the kind by `RuntimeProfile` and the group by `profiles.sbob`, keep the version, and set `subject` to `host` when the label `kubescape.io/sbom-type` has the value `node` or the label `sbob/subject-type` has the value `node` or `host`, and to `container` otherwise.

**1.8.2** To write the legacy form, a producer MUST replace the kind by `ContainerProfile` and the group by `spdx.softwarecomposition.kubescape.io`, keep the version, drop `subject`, and for `subject: host` add the label `kubescape.io/sbom-type: node` when it is not already present.

**1.8.3** Reading a legacy document and writing it back in the legacy form MUST give the original document: every field, every unknown field, every absent, explicitly empty and explicitly null value is kept.

**1.8.4** A converter MUST refuse a document of another kind or another group.

```yaml example:legacy
legacy:
  apiVersion: spdx.softwarecomposition.kubescape.io/v1beta1
  kind: ContainerProfile
  metadata:
    name: node-a
    labels: {kubescape.io/sbom-type: node}
  spec:
    syscalls: [read, openat]
    capabilities: []
subject: host
current:
  apiVersion: profiles.sbob/v1beta1
  kind: RuntimeProfile
  subject: host
  metadata:
    name: node-a
    labels: {kubescape.io/sbom-type: node}
  spec:
    syscalls: [read, openat]
    capabilities: []
```

```yaml example:legacy
legacy:
  apiVersion: spdx.softwarecomposition.kubescape.io/v1beta1
  kind: ContainerProfile
  metadata: {name: web}
  spec:
    execs: [{path: /bin/sh, args: null}]
    futureField: {kept: true}
subject: container
current:
  apiVersion: profiles.sbob/v1beta1
  kind: RuntimeProfile
  subject: container
  metadata: {name: web}
  spec:
    execs: [{path: /bin/sh, args: null}]
    futureField: {kept: true}
```

## 1.9 Encoding

**1.9.1** A consumer MUST keep fields it does not know, at every level, and MUST write them back unchanged.

**1.9.2** A consumer MUST keep the difference between an absent field, a field with an empty value and a field with the value null, and MUST write each back as it was read.

## 1.10 Document names

**1.10.1** An authored document MUST be named by a binding value (1.11), with the container suffix of 1.11.2 where it applies.

**1.10.2** A learned document MUST be named by the instance slug of its workload subject (1.12), unchanged.

## 1.11 Binding

**1.11.1** A Pod or Node binds to authored documents with the label `sbob/user-defined-profile`. Its value is the binding value.

**1.11.2** For a pod with several containers, the document for a container MUST be the one named `<value>-<container name>`, or the section of that container in a grouped document named `<value>` (1.5). For a pod with one container, a document named `<value>` MAY serve that container.

**1.11.3** A container MUST NOT be evaluated against the section of another container.

## 1.12 Identity

### The workload subject

The workload subject of a container is the tuple (API version, namespace, kind, name, alternate name, container kind, container name). The container kind is `container`, `initContainer` or `ephemeralContainer`. The effective name is the alternate name when it is non-empty, else the name.

**1.12.1** The subject string MUST be `apiVersion-<API version>/namespace-<namespace>/kind-<kind>/name-<effective name>/<container kind>Name-<container name>`.

**1.12.2** The subject string without container MUST be the subject string up to and excluding `/<container kind>Name-`.

**1.12.3** The subject hash MUST be the lower-case hexadecimal SHA-256 of the subject string, the container part included.

**1.12.4** A subject is invalid when its API version, namespace, kind or name is empty.

### The instance slug

**1.12.5** The slug of a subject MUST be computed as follows, with the constants named as in the reference library:

1. Let `digests` be `-`, the first 4 characters of the subject hash, `-`, and the last 4 characters of the subject hash.
2. Let `slug` be the kind, `-`, and the effective name.
3. When a container slug is asked for and the container name is non-empty, append `-`, the container name and `digests`.
4. When the length of `slug` in bytes is at least `MaxSlugLength` (253) minus 10, cut it to that many bytes and append `digests`.
5. Lower-case `slug`.

**1.12.6** A slug MUST be refused when it is shorter than 2 or longer than 253 bytes, does not start and end with a lower-case letter or digit, or contains a character other than lower-case letters, digits, `.` and `-`.

```yaml example:slug
apiVersion: v1
namespace: default
kind: Pod
name: web-7d4f8b6c9
container: app
containerKind: container
string: apiVersion-v1/namespace-default/kind-Pod/name-web-7d4f8b6c9/containerName-app
slug: pod-web-7d4f8b6c9-app-44f2-7cb7
slugWithoutContainer: pod-web-7d4f8b6c9
```

```yaml example:slug
apiVersion: apps/v1
namespace: shop
kind: Deployment
name: web
alternateName: web-5c689d88bb
container: app
containerKind: container
string: apiVersion-apps/v1/namespace-shop/kind-Deployment/name-web-5c689d88bb/containerName-app
slug: deployment-web-5c689d88bb-app-398c-66a1
slugWithoutContainer: deployment-web-5c689d88bb
```

### The workload identifier

The workload identifier is a legacy string form of a workload. The registry annotation `sbob/wlid` carries it; the identity fields above replace it.

**1.12.7** The workload identifier MUST be built as `wlid://cluster-<cluster>/namespace-<namespace>`, followed, when the kind is non-empty, by `/` and the kind lower-cased with every `-` removed, followed, when the name is also non-empty, by `-` and the name.

**1.12.8** A parser MUST accept the scheme `wlid://` or `sid://`, MUST strip the `cluster-` and `namespace-` prefixes, MUST split the third element at its first `-` into kind and name, and MUST map a kind that equals, ignoring case, one of the well-known kinds below to that kind's spelling.

The well-known kinds are: `ComponentStatus`, `ConfigMap`, `ControllerRevision`, `CronJob`, `CustomResourceDefinition`, `DaemonSet`, `Deployment`, `Endpoints`, `Event`, `HorizontalPodAutoscaler`, `Ingress`, `Job`, `Lease`, `LimitRange`, `LocalSubjectAccessReview`, `MutatingWebhookConfiguration`, `Namespace`, `NetworkPolicy`, `Node`, `PersistentVolume`, `PersistentVolumeClaim`, `Pod`, `PodDisruptionBudget`, `PodSecurityPolicy`, `PodTemplate`, `PriorityClass`, `ReplicaSet`, `ReplicationController`, `ResourceQuota`, `Role`, `RoleBinding`, `Secret`, `SelfSubjectAccessReview`, `SelfSubjectRulesReview`, `Service`, `ServiceAccount`, `StatefulSet`, `StorageClass`, `SubjectAccessReview`, `TokenReview`, `ValidatingWebhookConfiguration`, `VolumeAttachment`.

**1.12.9** An identifier MUST be refused when it is empty, contains whitespace, or does not have four non-empty parts.

```yaml example:wlid
cases:
- {cluster: prod, namespace: shop, kind: Pod, name: web, wlid: "wlid://cluster-prod/namespace-shop/pod-web"}
- {cluster: prod, namespace: shop, kind: Pod, name: "", wlid: "wlid://cluster-prod/namespace-shop/pod"}
- {cluster: prod, namespace: shop, kind: Pod, name: Web-Frontend, wlid: "wlid://cluster-prod/namespace-shop/pod-Web-Frontend"}
```

## 1.13 Learned documents

A learner observes a running container and writes a learned document for it. These clauses state what it records and under which name; how a learned document is generalized is chapter 3.

**1.13.1** A learner MUST record a successful DNS lookup that no connection follows as an `egress[]` entry whose `dnsNames` hold the looked-up name and whose `ports` list holds one element without a port, so that the entry admits the name (2.6) and no port (2.7.3). Repeated lookups of one name MUST yield one entry. A learner MAY bound the number of such names per container; when it does, it MUST count the names it leaves out.

**1.13.2** A learner that records a peer by pod selector MUST NOT copy into it the labels that change with a release, a revision or a run of the peer: `pod-template-hash`, `pod-template-generation`, `controller-revision-hash`, `statefulset.kubernetes.io/pod-name`, `apps.kubernetes.io/pod-index`, `app.kubernetes.io/version`, `helm.sh/chart`, `chart`, `skaffold.dev/run-id`, `controller-uid`, `batch.kubernetes.io/controller-uid`, `job-name`, `batch.kubernetes.io/job-name` and `service.istio.io/canonical-revision`. The label `version` MUST be kept, because workloads select on it.

**1.13.3** A learner that finds processes by scanning the running processes of a container, rather than by observing their execution, MUST record such a process as an execution only when it was already running before the learner observed executions in that container and its executable differs from its parent's. A process that runs its parent's executable is a fork; its command line, which a process may rewrite, MUST NOT be recorded as an argument vector.

**1.13.4** The subject of a learned document MUST be the workload subject of the container's pod owner (1.12), not of the pod. A restarted container and a new pod of the same owner revision therefore continue one learned document. A new revision of the owner (a new pod-template hash or controller revision) is a new subject.

## Conformance

A document producer conforms to this chapter when every document it writes passes the validation of 1.1, 1.4.1, 1.4.4, 1.4.11 and 1.5.2 and of 2.10, uses only registry keys for meanings this specification defines, and computes names and identifiers as 1.10 and 1.12 state.

A consumer conforms when it reads both forms of 1.8, keeps unknown, absent, empty and null fields (1.9), and reproduces the examples marked `example:` in this chapter.

## Open points

- `argsRequired` is carried for documents written by earlier producers. The reference library offers a matching function in which `argsRequired: false` places no constraint on the arguments, while the evaluation of allow and subtract sections (2.9, chapter 4) ignores the field. Which of the two is normative is not decided.
- `matchLabels` and `matchExpressions` directly under `spec` are carried and combined (4.3) but their use is not specified.
- `seccompProfile` and `identifiedCallStacks` are carried and combined (4.3) but not evaluated by this specification.
- `serviceRefNamespace`, `serviceRefName`, `serviceSelector` and `entity` are carried. Their evaluation is not specified in this version.
- `type` has no default in this version; v0.0.4 said `external`.
- `headers` values: whether they are literal values or patterns is not decided.
- The lifecycle of the status values (when each is written and what a consumer does with it) is not specified in this version.
- The workload-identifier parser of the reference library maps a name that equals a well-known kind to that kind's spelling as well (`pod` becomes `Pod`). This reproduces the earlier implementation and is not intended as a rule.

## Changes from v0.0.4

- One kind, `RuntimeProfile`, with a `subject` field (`container`, `host`) replaces `ContainerProfile`. The API group is `profiles.sbob`. The legacy form stays readable (1.8).
- Labels and annotations use the prefix `sbob/`. `kubescape.io/managed-by` became `sbob/managed-by`, read from the annotation only with the exact value `User` (1.2.3); `sbob.io/spec-version` became `sbob/spec-version`.
- §4.2 said there is no per-container list. Grouped documents with `containers`, `initContainers` and `ephemeralContainers` are part of the document (1.5).
- §2 and §4.1 required `metadata.namespace` and `spec.imageID`. Only `metadata.name`, `kind` and `subject` are required (1.1).
- §4.6 required `flags`; it is optional (1.4.2).
- §4.7 required `ports[]`; it is optional and an absent list places no port constraint (2.7.2).
- §4.4 described `headers` as a map of header name to value pattern; it is a map of header name to a list of values (1.4.7).
- The subtract section (1.6), status and completion (1.7), the legacy mapping (1.8), the encoding rules (1.9), document names (1.10), binding (1.11), identity (1.12) and learned documents (1.13) are new.
- The fields `syscalls`, `imageTag`, `seccompProfile`, `identifiedCallStacks`, `envs`, `argsRequired`, `internal`, `serviceRefNamespace`, `serviceRefName`, `serviceSelector` and `entity` were not described in v0.0.4.

---

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

**2.2.10** A one-token or a star that is only part of a segment, and the run-token anywhere in a path, are literal (2.1.3).

**2.2.11** A matcher MUST evaluate patterns with several star segments without exponential backtracking.

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
match: []
nomatch: [/tmp/xy, /tmp/x]
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
- every other segment is literal;
- without a run-token segment, the number of segments MUST be equal.

**2.3.7** An embedded pattern with more than one run-token segment MUST match nothing.

**2.3.8** A one-token or run-token that is only part of a segment of an argument (for example `--config=⋯` or `⋯:⋯`) is literal (2.1.3).

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
match: []
nomatch: [[--build-arg, TAGS=release]]
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

```yaml example:unmatchable
paths:
  dead: [/var/log/app-⋯.log, /tmp/⋯⋯, /usr/lib/llvm-⋯/bin/clang]
  live: [/var/log/⋯, /etc/*, "*", /plain/path]
args:
  dead: ["--config=⋯", "⋯:⋯", a/⋯⋯/b/⋯⋯]
  live: [⋯, ⋯⋯, /srv/⋯/x, /srv/⋯⋯/x, "--literal=*"]
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

---

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

---

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

**4.3.18** When two endpoint entries are equal except for `extensions` (4.3.5), or a network entry replaces an earlier one with the same identifier (4.3.6), their `extensions` MUST be combined key by key: a key present on one side is kept; a key present on both with values equal as compact JSON is kept once; a key present on both with different values MUST make the composition fail, naming the entry and the key. A composition MUST NOT drop such a key or keep only one of the values.

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

---

# 5. Trust (Draft)

Status: **Draft.** This chapter states what is fixed today: the bundle manifest, the keys a signed fragment carries, the verdict names and the inputs of a trust policy. The verification algorithm is not specified yet; see Open points.

## Scope

A governing document can be assembled from fragments (chapter 4) that different parties write and sign. Trust decides which fragments a verifier accepts and reports the outcome as a verdict. This chapter covers the bundle manifest, what a signed fragment carries, the verdicts and their meaning, and the inputs of a trust policy.

Clause numbers have the form `5.<section>.<n>`.

## 5.1 Bundle manifest

A bundle manifest lists the fragments of one bundle. Its schema is `schema/bundle-manifest.schema.json`.

| field | type | meaning |
|---|---|---|
| `root` | 64 lower-case hexadecimal characters | the Merkle root over the leaves, in manifest order |
| `leaves[]` | list of objects | one leaf per fragment |
| `leaves[].class` | `base`, `admission` or `overlay` | the fragment class (4.1) |
| `leaves[].signer` | string | the signer of the fragment |
| `leaves[].name` | string | the fragment's document name |
| `leaves[].contentDigest` | 64 lower-case hexadecimal characters | the digest of the fragment's content |
| `leaves[].version` | integer, 0 or more | the fragment version |

**5.1.1** A bundle manifest MUST be valid against the bundle manifest schema.

**5.1.2** The order of `leaves` is part of the manifest: the root MUST be computed over the leaves in the order they are listed.

**5.1.3** The `class`, `name` and `version` of a leaf MUST equal the class, the document name and the version of the fragment it describes.

## 5.2 Signed fragments

A signed fragment is a document (chapter 1) that carries its trust data in labels and annotations of the registry.

| key | kind | meaning |
|---|---|---|
| `sbob/bundle` | label | the bundle the fragment belongs to |
| `sbob/fragment-class` | label | the fragment class |
| `sbob/version` | label | the monotonic version of the fragment |
| `sbob/generator` | label | the tool that produced the document |
| `sbob/generator-ref` | label | the revision of that tool |
| `sbob/matcher-version` | label | the grammar version the document was written against |
| `sbob/bundle-manifest` | annotation | the bundle manifest the fragment commits to |
| `sbob/content` | annotation | the signed content of the fragment |
| `sbob/signature` | annotation | the signature |
| `sbob/certificate` | annotation | the signer's certificate |
| `sbob/identity` | annotation | the signer identity |
| `sbob/issuer` | annotation | the issuer of the signer identity |
| `sbob/timestamp` | annotation | the signing time |
| `sbob/rekor-bundle` | annotation | the transparency log bundle of the signature |

**5.2.1** A producer of signed fragments MUST write the keys of the table above under their current names, and a verifier SHOULD also read the legacy keys the registry pairs with them.

**5.2.2** The version of a fragment MUST only grow: a later fragment of the same bundle, class and name carries a higher version.

## 5.3 Verdicts

A verifier reports the trust state of the document that governs a subject as one verdict.

| verdict | meaning |
|---|---|
| `verified` | the document carries a signature, and the signature verifies for its content under the policy |
| `unsigned` | the document carries no signature |
| `tampered` | the document carries a signature that does not verify for its content |
| `unverified` | verification could not be completed |

**5.3.1** A verifier MUST report exactly one of the four verdicts for a governing document.

**5.3.2** A verifier MUST report a `tampered` verdict with the rule `signed-profile-tampered` (6.5).

**5.3.3** A verifier that reports that an unsigned authored document governs a subject MUST report it with the rule `unsigned-profile-governs` (6.5).

## 5.4 Trust policy

A trust policy tells a verifier what to accept. Its inputs are:

| input | meaning |
|---|---|
| mode | how strictly the verifier acts on a verdict |
| classes | per fragment class: the signers accepted for it and the document paths (spec sections) a fragment of that class may set |
| rule classes | which signers may supply rule sets |
| binding classes | which signers may supply bindings |
| policy version | the version of the policy |

**5.4.1** A verifier MUST NOT accept a fragment signed by a signer that the policy does not accept for the fragment's class.

**5.4.2** A verifier MUST NOT accept from a fragment a document path that the policy does not allow for the fragment's class.

## Conformance

This chapter is a draft. A verifier can claim conformance to 5.1 and 5.3 only.

## Open points

- The leaf encoding and the hash function of the Merkle tree of 5.1.2.
- The digest algorithm of `contentDigest` and the exact bytes it covers.
- The signature scheme, the form of `sbob/content`, and how the certificate, identity, issuer, timestamp and transparency log bundle are checked.
- The values of the policy mode and what each mode does with each verdict.
- The format of the policy and of its classes, rule classes and binding classes.
- The reload of a policy: the earlier skeleton of this chapter described it as a one-way door; what may and may not change on a reload is not specified.
- The replacement of a fragment by a newer version of itself (5.2.2) and what a verifier does with an older one.
- How `unverified` differs from `tampered` when the verifier cannot reach a resource it needs.

## Changes from v0.0.4

v0.0.4 §8 was marked work in progress and carried no text. Everything in this chapter is new.

---

# 6. Rules

Status: normative.

## Scope

This chapter defines the rule model: the reference form of a rule in the Common Expression Language (CEL), the event fields a rule sees, the profile functions through which a rule asks whether a document admits an observation, the evaluation of allow and subtract sections behind those functions, the catalogue of generalized rule names, the deny-all projection and its alert-once rule, rule policies, rule tags, and bindings.

Clause numbers have the form `6.<section>.<n>`.

## 6.1 Rule model

A rule set is an object of kind `Rules` in the group `rules.sbob` (legacy: kind `Rules`, `apiVersion` `kubescape.io/v1`). Each rule carries an engine identifier (`id`, of the form `R` and four digits), a display `name`, an `enabled` flag, a `severity`, `tags`, and `expressions`: a list of rule expressions, each an event type with a CEL expression, a `uniqueId` expression and a `message` expression.

**6.1.1** The reference form of a rule MUST be its CEL expressions. A rule fires on an event when the event's type equals the event type of one of its rule expressions and that expression evaluates to true for the event.

**6.1.2** A rule's identity in this specification MUST be its generalized rule name (6.5). An alert MUST carry the engine identifier of the rule that fired; a consumer maps it to the name with the catalogue.

**6.1.3** The value of the `uniqueId` expression for an event is the subject of the alert. Two alerts of one rule for one container with the same subject are repeats of each other.

**6.1.4** A rule that is not enabled MUST NOT fire.

**6.1.5** A rule expression MAY call functions outside the catalogue of 6.3 (helpers of the engine). Such a rule is not portable as written (chapter 7).

## 6.2 Event fields

A rule sees the event as `event` with the fields of its event type. The table lists the fields the reference rule set uses; an engine MAY offer more.

| event type | fields |
|---|---|
| `exec` | `containerId`, `containerName`, `namespace`, `podName`, `comm`, `pcomm`, `pid`, `exepath`, `args`, `cwd`, `upperlayer`, `pupperlayer` |
| `open` | `containerId`, `comm`, `pid`, `path`, `flags`, `flagsRaw` |
| `syscall` | `containerId`, `pid`, `syscallName` |
| `capabilities` | `containerId`, `comm`, `pid`, `capName`, `syscallName` |
| `dns` | `containerId`, `containerName`, `comm`, `name`, `rcode`, `numAnswers`, `addresses` |
| `network` | `containerId`, `containerName`, `comm`, `pktType`, `dstAddr`, `dstPort`, `proto`, `dstNamespace`, `dstPodLabels` |
| `bpf` | `containerId`, `comm`, `cmd` |
| `kmod` | `comm`, `module`, `syscallName` |
| `unshare` | `containerId`, `comm`, `pcomm` |
| `symlink`, `hardlink` | `containerId`, `comm`, `oldPath`, `newPath` |
| `ssh` | `containerId`, `comm`, `dstIp`, `dstPort`, `srcPort` |
| `ptrace`, `randomx` | `comm`, `exepath` |
| `iouring` | `comm`, `opcode`, `flagsRaw` |
| `kubelet_tls` | `comm`, `tlsData`, `tlsDataLen`, `tlsEventType` |
| `k8s-admission` | `Kind`, `Name`, `Namespace`, `UserInfo.Username` |

**6.2.1** `pktType` MUST be `OUTGOING` for a connection the subject opens and `HOST` for a connection to the subject.

**6.2.2** `addresses` of a `dns` event MUST be the addresses of the answer, and `numAnswers` their number.

## 6.3 Profile functions

A profile function answers whether the document that governs the container `containerId` admits an observation. Every function's first argument is the container identifier.

<!-- functions -->

| function | arity | meaning |
|---|---|---|
| `cp.was_executed` | 2 | the executable path is admitted (2.9.3) |
| `cp.was_executed_with_args` | 3 | the execution, path and argument vector, is admitted (2.9.2) |
| `cp.was_path_opened` | 2 | the file path is admitted (2.9.6) |
| `cp.was_path_opened_with_flags` | 3 | the file path is admitted with the given open flags (6.3.4) |
| `cp.was_path_opened_with_prefix` | 2 | an admitted open entry's path starts with the given string (6.3.5) |
| `cp.was_path_opened_with_suffix` | 2 | an admitted open entry's path ends with the given string (6.3.5) |
| `cp.was_syscall_used` | 2 | the system call is admitted (2.8.1, 2.9.8) |
| `cp.was_capability_used` | 2 | the capability is admitted (2.8.1, 2.9.8) |
| `cp.was_endpoint_accessed` | 2 | the endpoint is admitted (6.3.6) |
| `cp.was_endpoint_accessed_with_method` | 3 | the endpoint is admitted for the method (6.3.6) |
| `cp.was_endpoint_accessed_with_methods` | 3 | not specified in this version |
| `cp.was_endpoint_accessed_with_prefix` | 2 | an admitted endpoint entry starts with the given string (6.3.5) |
| `cp.was_endpoint_accessed_with_suffix` | 2 | an admitted endpoint entry ends with the given string (6.3.5) |
| `cp.was_endpoint_accessed_with_direction` | 3 | not specified in this version |
| `cp.was_endpoint_accessed_with_header` | 3 | not specified in this version |
| `cp.was_endpoint_accessed_with_header_value` | 4 | not specified in this version |
| `cp.was_external_endpoint_accessed` | 2 | not specified in this version |
| `cp.was_internal_endpoint_accessed` | 2 | not specified in this version |
| `cp.was_host_accessed` | 2 | an admitted endpoint entry names the host (6.3.7) |
| `cp.is_domain_in_egress` | 2 | the name is admitted by the egress names (2.6, 2.9.7) |
| `cp.is_domain_in_ingress` | 2 | the name is admitted by the ingress names (2.6) |
| `cp.was_address_in_egress` | 2 | the address is admitted by the egress addresses (2.5) |
| `cp.was_address_in_ingress` | 2 | the address is admitted by the ingress addresses (2.5) |
| `cp.was_address_port_protocol_in_egress` | 4 | address, port and protocol are admitted by one egress entry (6.3.8) |
| `cp.was_address_port_protocol_in_ingress` | 4 | address, port and protocol are admitted by one ingress entry (6.3.8) |
| `cp.was_selector_in_egress` | 5 | a peer in the namespace with the pod labels, on the port and protocol, is admitted by an egress selector entry (6.3.9, 6.3.14 to 6.3.16) |
| `cp.was_selector_in_ingress` | 5 | the same for ingress (6.3.9, 6.3.14 to 6.3.16) |
| `cp.is_host_peer_egress` | 4 | address, port and protocol are in the egress host-peer set (6.3.10) |
| `cp.is_host_peer_ingress` | 4 | address, port and protocol are in the ingress host-peer set (6.3.10) |
| `cp.was_address_port_protocol_resolved_from_egress_domain` | 4 | the address was answered for an admitted egress name whose entry admits the port and protocol (6.3.11) |
| `cp.was_address_resolved_from_egress_domain` | 2 | the address was answered for an admitted egress name, whatever the port (6.3.13) |
| `cp.is_dns_answer_in_egress` | 3 | every answer address lies in the address set of the name (6.3.12) |

**6.3.1** An engine that offers a function of the table MUST give it the arity and the meaning of the table. A rule that calls a `cp.` function outside the table, or with another arity, is not applicable to any engine (chapter 7).

**6.3.2** Every function MUST evaluate the allow sections and the subtract section of the container's section together: an observation that a subtract entry denies MUST make the function answer false (2.9).

**6.3.3** Under a deny-all projection every function MUST answer false (6.6).

**6.3.4** `cp.was_path_opened_with_flags` with one flag MUST answer true when the path is admitted, the flag is in the flag set of the path (4.5.3), and no subtract entry whose path matches lists the flag.

**6.3.5** The prefix and suffix functions MUST compare the given string with the entry strings of the section as written, not with observed paths. An allow entry withdrawn by an equal subtract entry MUST NOT count.

**6.3.6** An endpoint MUST be admitted when an allow endpoint entry matches it as a path pattern (2.2) and no subtract endpoint entry matches it. A subtract endpoint entry MUST deny the endpoint for every method, whatever methods it lists.

**6.3.7** `cp.was_host_accessed` MUST answer true when an admitted endpoint entry's URL has the given host, and false when a subtract endpoint entry has it.

**6.3.8** An address, a port and a protocol MUST be admitted by an entry when one of the entry's addresses matches the address (2.5) and the entry's port constraint (2.7) admits the port key of the port and protocol.

**6.3.9** A selector entry MUST admit a peer when its pod selector matches the peer's labels, its namespace selector matches the peer's namespace (an absent namespace selector meaning the subject's own namespace, an empty one every namespace), and its port constraint admits the port key. An entry with an absent or empty pod selector MUST admit no peer. An invalid selector MUST admit nothing and MUST NOT affect other entries.

**6.3.10** The host-peer sets are those of 4.5.6. The egress host-peer set MUST NOT admit a port of the claimed host set.

**6.3.11** `cp.was_address_port_protocol_resolved_from_egress_domain` MUST answer true when the engine has seen, for the container, a DNS answer that carried the address for a name that an egress entry with names admits, and that entry's port constraint admits the port key.

**6.3.12** For `cp.is_dns_answer_in_egress`, the address set of a name MUST be the union of the address lists of the egress entries whose names admit it, after subtract. When one of those entries has no address list, or its address list holds the star, the name is unconstrained and the function MUST answer true. Otherwise the function MUST answer true exactly when every answer address lies in the address set (2.5).

**6.3.13** `cp.was_address_resolved_from_egress_domain` MUST answer true when the engine has seen, for the container, a DNS answer that carried the address for a name that the egress names admit (2.6) and that no subtract entry's names deny. It places no port and no protocol constraint.

**6.3.14** A namespace selector MUST be evaluated against a label set that holds the single label `kubernetes.io/metadata.name` with the name of the peer's namespace as its value. The other labels of the namespace MUST NOT be read. A namespace selector on any other key therefore admits no peer.

**6.3.15** A pod selector MUST be evaluated as a label selector over the labels of the peer pod: every `matchLabels` pair and every `matchExpressions` requirement MUST hold. An entry that names a label whose value changes with a release of the peer stops admitting the peer once that value changes; what a learner records is governed by 1.13.2.

**6.3.16** A peer that the engine cannot resolve to a pod in a namespace MUST NOT be admitted by any selector entry.

The selector rules follow the Kubernetes NetworkPolicy model for the default namespace (6.3.9) and differ from it in three points: an absent or empty pod selector admits no pod, where NetworkPolicy selects every pod; a namespace selector is read against the namespace name only (6.3.14); and an invalid selector admits nothing.

## 6.4 Allow, then subtract

**6.4.1** A rule that reads a document section MUST fire only on observations the section does not admit. Its reference form negates a profile function (`!cp.was_path_opened(event.containerId, event.path)`).

**6.4.2** An entry in a subtract section MUST make every rule that reads the section fire for the observations it denies, in whichever fragment it was written (4.3.17).

## 6.5 Rule catalogue

Every rule has a generalized name. The catalogue is the `rules` list of the registry. A name marked proposed was given to a rule that v0.0.4 did not name.

<!-- rules -->

| name | section | engine ids | events | name status |
|---|---|---|---|---|
| `process-execution` | `execs[].path` | R0001 | exec | adopted |
| `process-arguments` | `execs[].args` | R0040 | exec | adopted |
| `file-access` | `opens[].path` | R0002 | open | adopted |
| `domain-communication` | `egress[].dnsNames` | R0005 | dns | adopted |
| `dns-answer-address` | `egress[].ipAddresses` | R0013 | dns | proposed |
| `egress-traffic` | `egress[]` | R0011 | network | adopted |
| `ingress-traffic` | `ingress[]` | R0012 | network | adopted |
| `syscall-use` | `syscalls` | R0003 | syscall | proposed |
| `capability-use` | `capabilities` | R0004 | capabilities | proposed |
| `binary-not-in-image` | `rulePolicies` | R1001 | exec | adopted |
| `execution-from-memory` | `rulePolicies` | R1005 | exec | adopted |
| `execution-from-shared-memory` | `rulePolicies` | R1000 | exec | adopted |
| `sensitive-file-access` | `rulePolicies` | R0010 | open | adopted |
| `link-over-sensitive-file` | `rulePolicies` | R1010, R1012 | symlink, hardlink | adopted |
| `service-account-token-access` | `rulePolicies` | R0006 | open | adopted |
| `procfs-environment-read` | `rulePolicies` | R0008 | open | adopted |
| `kubernetes-api-access` | `rulePolicies` | R0007 | exec, network | adopted |
| `module-load` | `rulePolicies` | R1002 | kmod | adopted |
| `ebpf-program-load` | `rulePolicies` | R0009 | bpf | adopted |
| `namespace-change` | `rulePolicies` | R1006 | unshare | adopted |
| `process-tracing` | `rulePolicies` | R1015 | ptrace | adopted |
| `dynamic-linker-hook` | `rulePolicies` | R1011 | exec, open | adopted |
| `ssh-connection` | `rulePolicies` | R1003 | ssh | adopted |
| `cgroup-release-agent-write` | `rulePolicies` | none | none | adopted |
| `stdio-to-socket` | `rulePolicies` | none | none | adopted |
| `process-from-mount` | `rulePolicies` | R1004 | exec | proposed |
| `crypto-miner-launched` | `rulePolicies` | R1007 | randomx | proposed |
| `crypto-mining-domain` | `rulePolicies` | R1008 | dns | proposed |
| `crypto-mining-port` | `rulePolicies` | R1009 | network | proposed |
| `signed-profile-tampered` | `trust` | R1016 | none | proposed |
| `unsigned-profile-governs` | `trust` | R1018 | none | proposed |
| `io-uring-operation` | `rulePolicies` | R1030 | iouring | proposed |
| `kubelet-exec-request` | `rulePolicies` | R1031 | none | proposed |
| `dns-exfiltration` | `rulePolicies` | R1032 | dns | proposed |
| `exec-to-pod` | `rulePolicies` | R2000 | none | proposed |

**6.5.1** A rule whose section is a document section reads that section through profile functions and fires on what it does not admit. A rule whose section is `rulePolicies` is a signature rule: it detects an action without a document, and the document supplies only exemptions (6.7). A rule whose section is `trust` reports a trust verdict (chapter 5).

**6.5.2** Each engine identifier MUST belong to one rule name. A rule name MAY have several engine identifiers.

**6.5.3** The rule `dns-answer-address` MUST fire on a successful DNS answer with at least one address for a name that the egress names admit, when `cp.is_dns_answer_in_egress` answers false for the answer's addresses. Its reference form is:

```
event.rcode == 0 && event.numAnswers > 0
  && cp.is_domain_in_egress(event.containerId, event.name)
  && !cp.is_dns_answer_in_egress(event.containerId, event.name, event.addresses)
```

**6.5.4** The rule `egress-traffic` MUST NOT fire for a connection that `cp.was_address_port_protocol_resolved_from_egress_domain` admits. A poisoned DNS answer for an admitted name is detected by `dns-answer-address`, not by `egress-traffic`.

## 6.6 Deny-all projection

A deny-all projection is the projection of an empty document: it admits nothing. When a verifier applies it is decided outside this chapter.

**6.6.1** Under a deny-all projection the rules MUST be evaluated as written, with every profile function answering false (6.3.3). A rule that negates a profile function therefore fires on every event of its type.

**6.6.2** Under a deny-all projection every rule MUST alert exactly once per rule, container and subject (6.1.3). Cooldown and deduplication settings of the engine MUST NOT apply to these alerts and MUST NOT make a subject alert again.

**6.6.3** The subject under a deny-all projection MUST be the value of the rule's `uniqueId` expression with the atomic-writer fold (3.7) applied, taken before any hashing. No other part of the subject is folded. The alert itself MUST keep the literal values of the event.

**6.6.4** File-open rules MUST alert under a deny-all projection like every other rule unless the engine is configured otherwise; the default MUST be to alert.

## 6.7 Rule policies

**6.7.1** The key of a rule policy (1.4) MUST be the engine identifier of the rule it exempts (`R0002`). A consumer maps it to the rule name with the catalogue.

**6.7.2** A rule that honours rule policies MUST NOT alert for an event whose process name (`comm`) is listed in `processAllowed` of the rule's policy, and MUST NOT alert for any event of the container when `containerAllowed` is true.

**6.7.3** A policy for a rule that does not honour policies MUST have no effect (1.4.13).

## 6.8 Rule tags

**6.8.1** A tag of the form `context:<context>` declares a context in which the rule applies. The contexts of this version are `kubernetes` (containers of workloads in a cluster) and `host` (host subjects). When a rule carries context tags, an engine MUST evaluate it only in the contexts they name.

**6.8.2** A tag of the form `portability:<class>` records the rule's fidelity class for engines other than the reference engine (chapter 7). It MUST NOT change how the reference engine evaluates the rule.

## 6.9 Bindings

A binding is an object of kind `RuntimeRuleAlertBinding` in the group `rules.sbob` (legacy: `apiVersion` `kubescape.io/v1`). It carries a `namespaceSelector` and a list `rules` of entries, each naming a rule by its display name in `ruleName`.

**6.9.1** A binding MUST name rules by the display `name` of the rule object.

## 6.10 Event loss

An engine can discard observations before its rules evaluate them, for example when its event queue is full. A discarded observation can hide a violation, so the loss itself has to reach the consumers of the alert stream.

**6.10.1** An engine that discards an observation before every rule has evaluated it MUST count the discarded observation, per reason and per event kind.

**6.10.2** While observations are discarded, an engine MUST report the loss in its alert stream at least once per reporting interval of at most 60 seconds. Each report MUST carry an identifier of the loss episode that is the same in every report of that episode, the start of the episode, the end of the reporting interval, the counts per reason and event kind accumulated since the start of the episode, and the count of the last reporting interval.

**6.10.3** After the first reporting interval without loss, an engine MUST report once that the loss ended, with the start and end of the whole episode and its total.

**6.10.4** A verifier MUST NOT treat an interval covered by an event-loss report as observed in full (8.8.2).

## Conformance

A rule engine conforms to this chapter when it offers the profile functions of 6.3 with their arity and meaning, evaluates allow and subtract together (6.3.2), uses the rule names of 6.5, applies the deny-all alert rule of 6.6 and the policies of 6.7, honours context tags (6.8.1), and reports event loss (6.10).

## Open points

- Functions marked "not specified in this version" are in the registry with their arity; their meaning is not pinned yet.
- `cp.was_path_opened_with_flags` with several flags, and the effect of a subtract entry with flags on `cp.was_path_opened`, are not pinned beyond 6.3.4 and 2.9.6.
- The prefix and suffix functions are pinned only for subtract entries equal to the allow entry (6.3.5).
- Whether a rule without context tags is evaluated in the `host` context is not specified.
- Which rules honour rule policies is recorded per rule by the engine; this version does not list it.
- The reference rule `dns-answer-address` takes the joined answer addresses as its subject, so it alerts once per answer address set rather than once per address.
- The fields `profileDependency`, `isTriggerAlert`, `supportPolicy`, `mitreTactic` and `mitreTechnique` of a rule object are carried; their meaning is not specified in this version.
- The effect of a binding entry that names no rule of the rule set is not specified.
- The event-loss report (6.10) has an engine identifier in the reference engine (R1019) but no generalized rule name in the registry of this version.
- v0.0.4 keyed rule policies by generalized rule name. The reference implementations key them by engine identifier (6.7.1). Whether rule names become accepted keys is not decided.

## Changes from v0.0.4

- §1.4 named 22 generalized rules; the catalogue names every rule of the reference rule set (6.5), new names marked proposed.
- §1.4.1 and §1.4.2 compared four tools in columns; that comparison is replaced by chapter 7.
- §4.8 keyed `rulePolicies` by generalized rule name; the implemented key is the engine identifier (6.7.1).
- The profile functions (6.3), the deny-all projection (6.6), context tags (6.8), bindings (6.9) and event-loss reporting (6.10) are new.
- v0.0.4 said the network stanzas mirror NetworkPolicy; selector evaluation differs from it in the three points after 6.3.16.

---

# 7. Portability to other engines

Status: normative for the classification; the per-surface table records the state of the reference rule set.

## Scope

The reference engine evaluates every rule of chapter 6 in its CEL form. Another engine (a second rule engine, an admission policy engine, a system-call filter) cannot run that form directly. This chapter defines the portable subset of the rule language, the fidelity classes a rule falls into, the projection model by which a second engine serves a document, and the class of every rule of the reference rule set.

Clause numbers have the form `7.<section>.<n>`.

## 7.1 Fidelity classes

| class | meaning for a second engine |
|---|---|
| `native` | the rule uses only constructs every engine with the same event fields has; it can be written as is |
| `generated` | the rule calls profile functions; the engine needs the document compiled into its own data (7.3) |
| `degraded` | the rule uses constructs a second engine may lack (macros, conversions, helpers, arithmetic); it can at best be approximated |
| `not-applicable` | the rule has no portable form: no expression, a constant expression, an unknown or misused profile function, or a parse error |

**7.1.1** The classes MUST be ordered `native`, `generated`, `degraded`, `not-applicable`, from best to worst.

**7.1.2** The class of an expression MUST be the worst class of its findings (7.2), and `native` when it has none. The class of a rule MUST be the worst class of its expressions, and `not-applicable` when it has no expression.

## 7.2 The portable subset

**7.2.1** The following constructs MUST add no finding: the operators `&&`, `||`, `!`, `==`, `!=`, `<`, `<=`, `>`, `>=` and `in`; parentheses; string, number, boolean and null literals; list literals; references to `event` fields; references to variables bound by a macro; and the methods `startsWith`, `endsWith` and `contains` with one argument.

**7.2.2** A call of a profile function of the catalogue (6.3) with its arity MUST add a `generated` finding.

**7.2.3** A call of a `cp.` function that is not in the catalogue, a call with another arity, and a reference to a `cp.` name without a call MUST add a `not-applicable` finding.

**7.2.4** The following constructs MUST add a `degraded` finding: the conditional operator `?:`; the arithmetic operators `+`, `-`, `*`, `/` and `%` (a minus sign before a number literal excepted); field selection on a computed value; indexing; the macros `exists`, `exists_one`, `all`, `map`, `filter` and `has`; the conversions `string`, `int`, `uint`, `double`, `bool`, `bytes`, `dyn`, `type`, `duration` and `timestamp`; any other global function; any other method; the portable methods with another number of arguments; map literals; functions in any other namespace (engine helpers); and references to unknown identifiers.

**7.2.5** An expression that is empty, that does not parse, that nests deeper than 128 levels, or that is the constant `true` or `false` (in any number of parentheses) MUST be classified `not-applicable`.

**7.2.6** String literals MUST be skipped when looking for constructs: text inside a literal never adds a finding.

```yaml example:classify
cases:
- {expression: "event.comm == 'kubectl'", class: native}
- {expression: "event.dstPort in [3333, 45700]", class: native}
- {expression: "event.path.startsWith('/proc/') && event.path.endsWith('/environ')", class: native}
- {expression: "!cp.was_path_opened(event.containerId, event.path)", class: generated}
- {expression: "event.args.exists(a, a == '-c')", class: degraded}
- {expression: "size(event.args) > 3", class: degraded}
- {expression: "event.exepath.matches('/proc/[0-9]+/fd/[0-9]+')", class: degraded}
- {expression: "!cp.was_executed(event.containerId, parse.get_exec_path(event.args, event.comm, event.exepath))", class: degraded}
- {expression: "true", class: not-applicable}
- {expression: "!cp.was_executed(event.containerId)", class: not-applicable}
- {expression: "event.comm == 'cp.unknown(1)'", class: native}
```

## 7.3 Projection model

A second engine does not interpret the CEL text of a rule. It serves a rule from a template of its own and a document from data compiled out of the document.

**7.3.1** A second engine SHOULD evaluate a generated rule against the projection of the governing section (4.5), compiled into the engine's own data, and SHOULD NOT translate the CEL text of the rule.

**7.3.2** A second engine that matches patterns by regular expressions MUST render them by appendix A and MUST check its rendering by A.3.2.

**7.3.3** A second engine that cannot tell containers apart and scopes a document by image SHOULD scope it by the image digest (`imageID`).

**7.3.4** A second engine MUST NOT emit a rule whose class for that engine has not been checked against the reference engine. It MUST report such a rule instead.

**7.3.5** The reference engine MUST evaluate every rule of its rule set in the CEL form, whatever the rule's class.

## 7.4 The reference rule set

The table gives, for every rule of the reference rule set, the class by 7.2 and the findings that decide it. Each rule of the reference rule set carries the same class as its tag `portability:<class>` (6.8.2).

<!-- portability -->

| engine id | rule | class | deciding findings |
|---|---|---|---|
| R0001 | `process-execution` | degraded | non-catalogue function parse.get_exec_path |
| R0040 | `process-arguments` | degraded | non-catalogue function parse.get_exec_path |
| R0002 | `file-access` | generated | calls cp.was_path_opened |
| R0003 | `syscall-use` | generated | calls cp.was_syscall_used |
| R0004 | `capability-use` | generated | calls cp.was_capability_used |
| R0005 | `domain-communication` | generated | calls cp.is_domain_in_egress |
| R0013 | `dns-answer-address` | generated | calls cp.is_domain_in_egress; calls cp.is_dns_answer_in_egress |
| R1032 | `dns-exfiltration` | degraded | method split; macro exists; function size |
| R0006 | `service-account-token-access` | generated | calls cp.was_path_opened_with_suffix |
| R0007 | `kubernetes-api-access` | degraded | non-catalogue function parse.get_exec_path; non-catalogue function k8s.is_api_server_address |
| R0008 | `procfs-environment-read` | generated | calls cp.was_path_opened_with_suffix |
| R0009 | `ebpf-program-load` | degraded | conversion uint |
| R0010 | `sensitive-file-access` | generated | calls cp.was_path_opened |
| R0011 | `egress-traffic` | generated | calls cp.was_address_port_protocol_in_egress; calls cp.was_selector_in_egress; calls cp.is_host_peer_egress; calls cp.was_address_port_protocol_resolved_from_egress_domain |
| R0012 | `ingress-traffic` | generated | calls cp.was_address_port_protocol_in_ingress; calls cp.was_selector_in_ingress; calls cp.is_host_peer_ingress |
| R1000 | `execution-from-shared-memory` | degraded | non-catalogue function parse.get_exec_path |
| R1001 | `binary-not-in-image` | degraded | non-catalogue function parse.get_exec_path |
| R1002 | `module-load` | native | none |
| R1003 | `ssh-connection` | degraded | conversion dyn |
| R1004 | `process-from-mount` | degraded | non-catalogue function parse.get_exec_path; non-catalogue function k8s.get_container_mount_paths; macro exists |
| R1005 | `execution-from-memory` | degraded | method matches |
| R1006 | `namespace-change` | generated | calls cp.was_syscall_used |
| R1007 | `crypto-miner-launched` | not-applicable | constant expression true |
| R1008 | `crypto-mining-domain` | native | none |
| R1009 | `crypto-mining-port` | generated | calls cp.was_address_in_egress |
| R1010 | `link-over-sensitive-file` | generated | calls cp.was_path_opened |
| R1011 | `dynamic-linker-hook` | degraded | non-catalogue function process.get_ld_hook_var; macro has |
| R1012 | `link-over-sensitive-file` | generated | calls cp.was_path_opened |
| R1015 | `process-tracing` | not-applicable | constant expression true |
| R1030 | `io-uring-operation` | degraded | conversion string |
| R1031 | `kubelet-exec-request` | not-applicable | constant expression true |
| R1016 | `signed-profile-tampered` | not-applicable | no expressions |
| R2000 | `exec-to-pod` | native | none |

**7.4.1** The portability tag of every rule of a rule set that carries one MUST equal the rule's class by 7.2.

A rule classed `not-applicable` because its expression is the constant `true` fires on every event of its type: its detection is in the event source, which a second engine needs to provide on its own.

## Conformance

A classifier conforms to this chapter when it gives every example marked `example:` in this chapter the stated class and agrees with the table of 7.4 on the reference rule set. A second engine conforms when it meets 7.3.2 and 7.3.4 and evaluates every rule it emits with the fidelity of its class.

## Open points

- The per-engine fidelity of each rule (a cell per engine and rule) is recorded only once it is proven; this version records the class of each rule, not the cells.
- What each kind of engine cannot observe (process arguments as a vector, DNS answers, the upper layer of an overlay filesystem) is not listed in this version.
- The helper functions of the reference engine (`parse.`, `k8s.`, `process.`) are degraded by 7.2.4. Whether some of them become catalogue functions is not decided.

## Changes from v0.0.4

- §1.4 compared four tools in unverified columns. Those columns are removed; portability is stated per rule as a class (7.4) and per engine only once proven.
- The fidelity classes (7.1), the portable subset (7.2) and the projection model (7.3) are new.

---

# 8. Wire-level intent (Draft)

Status: draft. Not normative until the conformance suite covers it.

## Scope

Chapters 1 to 7 describe what a subject does towards the kernel: executions, files, system calls, capabilities, and
the peers and HTTP endpoints it talks to. This chapter extends the same document so that it also states what is
intended **on the wire**: which application protocol is spoken on an endpoint, with which peer, under which transport
security, and which classes of data may travel in it.

The chapter defines the keys of the `extensions` object that endpoint and network entries carry (1.4.14), the
registry lists those keys draw their values from, and the outcome a verifier reports for them. It adds no section,
no kind and no token to the grammar. A document without `extensions` means exactly what chapters 1 to 7 say.

Intent that spans several connections is sketched in 8.9. It is less mature than the rest.

## 8.1 Design rules

**8.1.1** Every key of `extensions` defined by this chapter is optional, and its absence MUST place no constraint. A
consumer that does not evaluate `extensions` keeps it unchanged and matches the entry as before (1.4.14, 1.4.15).

**8.1.2** An empty list under a key of this chapter MUST mean that nothing of that kind is intended (as 1.3.1).

**8.1.3** A verifier MAY evaluate only some of the keys of this chapter. It MUST publish which ones it evaluates,
per protocol (8.8.5). A key a verifier does not evaluate MUST NOT be reported as satisfied.

**8.1.4** A key of `extensions` that this chapter does not define and the registry does not list MUST be kept and
MUST NOT be given a meaning.

## 8.2 Endpoint entries

Keys of `extensions` on an endpoint entry (1.4):

<!-- fields: EndpointExtensions -->

| key | type | description |
|---|---|---|
| `protocol` | string | application protocol, from the registry list `extensions.protocols` (8.3). Absent means `http` |
| `peer` | string | `identifier` of a network entry of the same section level: `egress` for `direction: outbound`, `ingress` for `inbound` |
| `requires` | list of string | session properties the exchange must have: `tls`, `auth` (8.5) |
| `data` | object | data classes intended in the exchange (8.6) |

<!-- fields: EndpointData -->

| field | type | description |
|---|---|---|
| `request` | list of string | data classes intended in the request |
| `response` | list of string | data classes intended in the response |

**8.2.1** `endpoint` keeps the form `:<port><resource>` and `methods` keeps its type. For `protocol: http` the
resource is the request path and the methods are HTTP methods, as today. For every other protocol the resource and
the verbs are the ones 8.3 lists.

**8.2.2** The resource MUST be matched as a path (2.2). A protocol whose resource has no `/` is written with one
leading `/` (`:5432/shop`, `:9092/orders`).

**8.2.3** `methods` MUST be compared without regard to case. An absent or empty `methods` list places no constraint
on the verb. (Note: this differs from 8.1.2 on purpose, to keep the meaning 1.4 documents have today.)

**8.2.4** An entry with `peer` admits an exchange only when the remote side of the connection is admitted by the
network entry with that identifier (2.5, 2.6, 2.7). A `peer` that names no entry of the section level makes the
entry unmatchable (2.10).

**8.2.5** `direction` is required on an entry whose `extensions` carry `peer`, `requires` or `data`.

## 8.3 Protocols

The registry list `extensions.protocols` names the protocols. Each protocol fixes what the resource and the verb of an exchange are.

| `protocol` | resource | verbs (`methods`) |
|---|---|---|
| `http` | request path, query string removed | HTTP method |
| `grpc` | `/<package>.<Service>/<Method>` | `unary`, `client-stream`, `server-stream`, `bidi` |
| `websocket` | path of the upgrade request | `text`, `binary` |
| `pgsql` | `/<database>` | first keyword of the statement, upper case (`SELECT`, `INSERT`, `COPY`, ...) |
| `mysql` | `/<database>` | as `pgsql` |
| `redis` | `/` | command name, upper case (`GET`, `SET`, `AUTH`, ...) |
| `kafka` | `/<topic>` | `produce`, `fetch` |
| `dns` | `/<query name in canonical form (2.6.1)>` | record type (`A`, `AAAA`, `TXT`, ...) |

**8.3.1** A verifier that sees traffic on a port whose endpoint entry names protocol P, and decodes it as another
protocol, MUST NOT admit it by that entry.

**8.3.2** HTTP/2 is `http`. The version is a session property, not a protocol (8.5).

## 8.4 Network entries

Key of `extensions` on a network entry (1.4; `ingress[]`, `egress[]`):

<!-- fields: NetworkExtensions -->

| field | type | description |
|---|---|---|
| `tls` | object | transport security intended on connections with this peer |

<!-- fields: NetworkTLS -->

| field | type | description |
|---|---|---|
| `minVersion` | string | lowest protocol version: `1.2` or `1.3` |
| `cipherSuites` | list of string | IANA cipher suite names; the star admits any |
| `groups` | list of string | IANA key-exchange group names (`X25519`, `X25519MLKEM768`); the star admits any |
| `alpn` | list of string | application protocol names negotiated |

**8.4.1** A network entry with a `tls` object admits a connection only when the connection is TLS and every field
present in the object admits the negotiated value. Names are compared byte for byte (as 2.8.1).

**8.4.2** `tls: {}` MUST mean: TLS, with no constraint on its parameters.

**8.4.3** A connection is judged by what was **negotiated** (the server's choice), not by what was offered.

#### Peer classes

`entity` on a network entry (1.4, carried, not evaluated by chapters 1 to 7) names a class of peers that the
verifier resolves, in place of addresses.

**8.4.4** An entry with `entity` admits a connection when the verifier places the remote address in that class and
the entry's port constraint admits the port. The registry list `extensions.peer_classes` names them: `host` (the meaning the field
has today), `cluster`, `world`, and jurisdiction classes written as lower-case ISO 3166 codes or `eu`, `eea`.

**8.4.5** A verifier that cannot place an address in a class it is asked about MUST report `inconclusive` with reason
`peer-class-unknown`, never admit and never deny.

```yaml
egress:
- {identifier: partners, type: external, entity: eu, ports: [{name: TCP-443, protocol: TCP, port: 443}]}
```

## 8.5 Session requirements

**8.5.1** `requires: [tls]` on an endpoint entry MUST admit an exchange only on a TLS connection. It is the
endpoint-level form of 8.4.2 and is used when no `peer` is named.

**8.5.2** `requires: [auth]` MUST admit an exchange only when the request carries a credential: an `Authorization`
header, a session cookie, a client certificate, or the protocol's own authentication exchange before the first
statement. Which credential, and its scope, is not specified in this version (Open points).

**8.5.3** `requires: [cleartext]` is valid only in a subtract entry. It matches an exchange that
is not on a TLS connection. It is how "this data only over TLS" is written: the classes are withdrawn wherever the
exchange is in cleartext. An allow entry with `cleartext` MUST be rejected by a validator.

```yaml
# Personal data never travels in cleartext, on any protocol, port or path, in either direction.
spec:
  subtract:
    endpoints:
    - {endpoint: ":0/*", direction: outbound, extensions: {requires: [cleartext], data: {request: ["pii.*"], response: ["pii.*"]}}}
    - {endpoint: ":0/*", direction: inbound,  extensions: {requires: [cleartext], data: {request: ["pii.*"], response: ["pii.*"]}}}
```

## 8.6 Data classes

The registry list `extensions.data_classes` names the classes. Names are dotted, lower case. The first label is the family.

| family | classes |
|---|---|
| `pii` | `pii.email`, `pii.phone`, `pii.name`, `pii.iban`, `pii.card`, `pii.national-id`, `pii.health` |
| `secret` | `secret.bearer`, `secret.jwt`, `secret.private-key`, `secret.cloud-key`, `secret.password`, `secret.token` |
| `marker` | `marker.canary` (a value planted by a test, 8.9) |

**8.6.1** A class entry is a class name, a family followed by `.*`, or the star. `pii.*` admits every class of the
family. The star admits every class.

**8.6.2** An endpoint entry with `data.request` admits an exchange only when every class found in the request is
admitted by the list. The same holds for `data.response` and the response.

**8.6.3** `data: {request: []}` MUST mean that no classified data is intended in the request (8.1.2). An absent
`data`, or an absent `request`, places no constraint.

**8.6.4** A verifier MUST report the class and the location of a finding and MUST NOT report the value.

## 8.7 Subtract

`subtract.endpoints`, `subtract.ingress` and `subtract.egress` take the added fields with the shape of the allow
entries (1.6.1).

**8.7.1** A subtract endpoint entry with `data` matches an exchange when the exchange matches the entry's other
fields **and at least one** class found in it is matched by the list. (An allow entry needs all classes admitted,
8.6.2; a subtract entry needs one to withdraw.)

**8.7.2** A subtract network entry with `tls` matches a connection that is TLS and whose negotiated values are
matched by every field present. A subtract network entry cannot express "not TLS"; that is the allow side
(`tls: {}`) of 8.4.2.

```yaml
# No credential leaves the subject towards an external peer, on any protocol, port or path.
spec:
  subtract:
    endpoints:
    - {endpoint: ":0/*", direction: outbound, internal: false, extensions: {data: {request: ["secret.*"]}}}
```

## 8.8 Verdicts and observability

Chapters 2 and 4 know two outcomes for an observation: admitted or not. Wire properties add a third situation: the
verifier could not see.

**8.8.1** For each entry that carries a field of this chapter, and for each evaluation window, a verifier MUST
report one of:

| status | meaning |
|---|---|
| `compliant` | exchanges were observed, the whole window was observed, and every one was admitted |
| `violated` | at least one exchange was observed that is not admitted, or that a subtract entry matches |
| `not-observed` | the whole window was observed and no exchange concerning the entry took place |
| `inconclusive` | the verifier could not observe what the entry constrains, in all or part of the window |

**8.8.2** A verifier MUST NOT report `compliant` or `not-observed` unless it can show that it observed the whole
window. A `violated` finding stands on partial observation.

**8.8.3** `inconclusive` MUST carry a reason from the registry list `extensions.inconclusive_reasons`: `sensor-gap`,
`result-truncated`, `payload-not-decrypted`, `protocol-not-decoded`, `payload-truncated`, `handshake-not-seen`,
`peer-class-unknown`, `timeout`, `error`.

**8.8.4** `data` on an exchange whose payload the verifier cannot read (encrypted and not decrypted, or cut before
its end) is `inconclusive`, never `compliant`. `tls` and `requires: [tls]` do not need the payload and are judged
from the handshake.

**8.8.5** A verifier MUST publish a capability statement: per protocol, which fields of this chapter it evaluates
and from which evidence.

## 8.9 Intent across connections (sketch)

Three checks cannot be written as properties of one exchange: erasure (a value must be gone after a delete), audit
(an event must be followed by a log record), and incident notification. They need an ordered relation between
observations. They do not belong in the behaviour document, which composes (chapter 4) and binds (1.11) per
container.

Proposed: a second kind in the same group, bound by label selector, evaluated by the same verifier.

```yaml
apiVersion: profiles.sbob/v1beta1
kind: BehaviourRequirement
metadata: {name: erasure, namespace: shop}
spec:
  selector: {matchLabels: {app.kubernetes.io/part-of: shop}}
  sequence:
  - name: create
    do:     {protocol: http, endpoint: ":8080/api/users", methods: [POST], plant: marker.canary}
  - name: delete
    do:     {protocol: http, endpoint: ":8080/api/users/⋯", methods: [DELETE]}
  - name: gone
    within: 60s
    forbid:
    - {protocol: "*", endpoint: ":0/*", data: {request: [marker.canary], response: [marker.canary]}}
```

The entries inside `do`, `forbid` and `require` are written flat here for brevity; they take the keys of 8.2.
`do` steps are performed by the verifier's exerciser; `forbid` and `require` steps are observed. `require` has the
shape of `forbid` and is satisfied by at least one matching observation inside `within`. This section needs a
grammar for step references and time; it is not ready for clauses.

## Open points

- **Resolving jurisdiction classes** (8.4.4): which address-to-country source a verifier uses, how its age is
  reported, and what an anycast or CDN address resolves to.

- **Scope of `auth`** (0038): scheme, audience, and whether the credential is the one issued to this subject.
- **`headers` value patterns** are undecided in 1.4; `requires: [auth]` avoids depending on them.
- **Statement patterns for `pgsql` / `mysql`.** Only the first keyword is a verb here. Table-level intent
  (`SELECT` on `users`) needs a resource deeper than the database and a statement parser.
- **`methods` empty list** keeps today's meaning and so contradicts 8.1.2. Accept, or version the field.
- **Learned documents.** Whether a learner may write the added fields (protocol and peer are learnable; `data`
  classes are learnable only where payloads are readable) and how 3 (collapse) treats resources of non-HTTP protocols.
- **8.9** as a whole.

---

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
| `⋯`, `⋯⋯` or `*` inside a segment, `⋯⋯` as a segment | escaped as literal characters | 2.2.10 |

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
- {pattern: /tmp/x⋯, regex: "^/tmp/x⋯$", match: [], nomatch: [/tmp/xy]}
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
- {token: "--x=⋯", regex: "--x=⋯"}
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

</div>
