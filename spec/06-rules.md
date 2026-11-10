# 6. Rules

Status: skeleton. Normative text is carried in `sbob-v0.0.4.md` §1.4 and §4.8 until this chapter replaces it.

## Scope

The rule catalogue keyed by generalized rule name, each with its canonical CEL form and the engine rule identifier alongside; the profile functions a rule may call, with arity; allow-then-subtract evaluation; deny-all projection semantics (alerts exactly once per rule, container and subject); rulePolicies exemptions and the rule that an unhonoured policy entry never denies.

## Pinned by

`conventions/conventions.yaml` (rule names, identifiers, functions), vector set `rules`, `truth/PB` rows on deny-all.

## Carried from v0.0.4

§1.4.1, §1.4.2, §1.4.3 and §4.8 verbatim. The Tool 2 to Tool 4 columns are replaced by chapter 7 once proven.
