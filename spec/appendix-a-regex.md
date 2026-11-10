# Appendix A. Regular-expression rendering of patterns

Status: skeleton.

## Scope

A normative rendering of every pattern form to an RE2 full-match expression: `⋯` to one segment, mid `*` to zero or more segments, trailing `*` to one or more segments, `⋯⋯` to an argument tail, DNS forms to label groups. The rendering is correct when it agrees with the matcher on every case of the `match` vector sets.

## Pinned by

The differential vectors of `match/regex`.
