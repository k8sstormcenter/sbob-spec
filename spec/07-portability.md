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
