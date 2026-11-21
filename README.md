# SBoB specification

The Software Bill of Behaviour specification: the behaviour document, its pattern grammar, matching semantics, composition and trust model, conventions, and the portable rule set.

The current version is 0.2.0: `spec/sbob-v0.2.0.md` is its text in one document, generated from the chapters. `spec/sbob-v0.0.4.md` is the published text of version 0.0.4 (stage Proposed), imported verbatim as the seed and superseded by 0.2.0. Chapters under `spec/` are the normative text and list their changes from the seed; `schema/` holds the behaviour document schema, the bundle manifest schema and the alert stream contract; `conventions/conventions.yaml` is the registry of keys, values, kinds, rule names and rule functions.

This repository holds specification text and schemas only. Conformance is proven against a separate conformance suite; a summary is published here once it is green.
