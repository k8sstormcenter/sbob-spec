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

**1.4.1** An exec entry MUST have a non-empty `path`.

**1.4.2** An exec entry admits an execution when its `path` matches the executable path and its `args` match the argument vector (2.3). An entry without `args` admits only an execution without arguments (2.3.2).

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
- The subtract section (1.6), status and completion (1.7), the legacy mapping (1.8), the encoding rules (1.9), document names (1.10), binding (1.11) and identity (1.12) are new.
- The fields `syscalls`, `imageTag`, `seccompProfile`, `identifiedCallStacks`, `envs`, `argsRequired`, `internal`, `serviceRefNamespace`, `serviceRefName`, `serviceSelector` and `entity` were not described in v0.0.4.
