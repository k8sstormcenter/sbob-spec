# 8. Wire-level intent

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
