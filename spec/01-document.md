# 1. The behaviour document

Status: skeleton. Normative text is carried in `sbob-v0.0.4.md` §3 and §4 until this chapter replaces it.

## Scope

The envelope (kind, metadata, identity), the sections execs, opens, syscalls, capabilities, endpoints, ingress, egress, rulePolicies, architectures, imageID and imageTag, the subtract section with the same shape as the allow sections, grouped documents with per-container sections (containers, initContainers, ephemeralContainers), and the status and completion model.

Planned changes recorded here and ruled on 2026-11-10: one kind `RuntimeProfile` with a `subject` field (`container`, `host`, open for more) replaces `ContainerProfile`; the API group becomes `profiles.sbob`; labels and annotations use the prefix `sbob/`; the document name of a learned profile keeps the instance slug, an authored document is named by its binding label value.

## Pinned by

`schema/behaviour-document.schema.json` (generated once from the implementation types, now the source), vector sets `truth/PB` (binding and per-container section rows) and `truth/LL`.

## Carried from v0.0.4

§3 and §4.1 to §4.7 verbatim. §4.8 moves to chapter 06.
