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

**0.6.1** This specification is versioned with semantic versions. The version of this text is `0.2.0`. The registry carries its own `spec_version`: the version of the text in which its keys and values last changed, `0.1.0`.

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
