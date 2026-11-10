# 2. Pattern and matching grammar

Status: skeleton with one ruled delta. Normative text is carried in `sbob-v0.0.4.md` §5 until this chapter replaces it.

## Scope

Tokens: `⋯` (one path segment, or one argument, or one embedded segment), `*` (mid-path zero or more segments, trailing one or more segments, literal inside arguments, the any-sentinel for IP addresses, capabilities, syscalls and DNS names), `⋯⋯` (zero or more arguments; inside a token one or more segments). Path, exec pair, argument, IP, CIDR, DNS, port and protocol matching. The basename rule: the recorded and the invoked argv0 are compared by basename. Validation of entries.

## Ruled deltas against v0.0.4 (2026-11-10)

- §5.3 is corrected: `⋯` inside arguments matches exactly one argument or one embedded segment; `⋯⋯` matches zero or more arguments; `⋯⋯` inside a token matches one or more segments. §4.5 was right.
- The bare `*` is the any-sentinel for capabilities, syscalls and dnsNames, accepted by admission and by the verifier alike, consistent with §5.7 for IP addresses.
- Basename comparison of argv0 is part of the contract: the recorded and the invoked name match when their last path elements are equal; an empty name matches nothing; a multiplexer binary is not its applet.
- The reference matcher implements the four rulings above as of 2026-11-10 (corpus sections `execNames` and `sentinels`), and admission accepts a lone `*` in a DNS list.

## Pinned by

Vector sets `match/paths`, `match/args`, `subtract`, and the regex rendering vectors of appendix A.

## Carried from v0.0.4

§5.1, §5.6, §5.7, §5.8 verbatim; §5.3 as corrected above; §5.4 stays TBD until chapter 0 defines NULL versus NONE.
