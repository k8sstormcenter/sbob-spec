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
| `cp.was_path_opened_by` | 3 | the file path is admitted for the given executable (6.3.17) |
| `cp.was_path_opened_with_prefix` | 2 | an admitted open entry's path starts with the given string (6.3.5) |
| `cp.was_path_opened_with_suffix` | 2 | an admitted open entry's path ends with the given string (6.3.5) |
| `cp.was_path_opened_with_suffix_by` | 3 | an admitted open entry's path ends with the given string and the entry admits the given executable (6.3.19) |
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

**6.3.17** `cp.was_path_opened_by` MUST answer true when an allow open entry admits the path and the executable (1.4.16) and no subtract open entry denies them (2.9.9).

**6.3.18** The other open functions of the table MUST NOT read `processes`: an allow open entry counts for them whatever its processes, and a subtract open entry with `processes` MUST NOT deny for them.

**6.3.19** `cp.was_path_opened_with_suffix_by` MUST answer true when an allow open entry whose path, as written, ends with the given string admits the executable (1.4.16), and no subtract open entry whose path ends with the string denies it for that executable (2.9.9).

**6.3.20** For `cp.is_dns_answer_in_egress`, a subtract egress entry withdraws its addresses from the answers of a name only when it has no ports. A subtract egress entry with ports MUST NOT deny an answer, since a DNS answer carries no port. An entry whose ports list holds no port counts as an entry without ports.

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
