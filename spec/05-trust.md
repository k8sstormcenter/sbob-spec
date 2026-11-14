# 5. Trust

Status: **Draft.** This chapter states what is fixed today: the bundle manifest, the keys a signed fragment carries, the verdict names and the inputs of a trust policy. The verification algorithm is not specified yet; see Open points.

## Scope

A governing document can be assembled from fragments (chapter 4) that different parties write and sign. Trust decides which fragments a verifier accepts and reports the outcome as a verdict. This chapter covers the bundle manifest, what a signed fragment carries, the verdicts and their meaning, and the inputs of a trust policy.

Clause numbers have the form `5.<section>.<n>`.

## 5.1 Bundle manifest

A bundle manifest lists the fragments of one bundle. Its schema is `schema/bundle-manifest.schema.json`.

| field | type | meaning |
|---|---|---|
| `root` | 64 lower-case hexadecimal characters | the Merkle root over the leaves, in manifest order |
| `leaves[]` | list of objects | one leaf per fragment |
| `leaves[].class` | `base`, `admission` or `overlay` | the fragment class (4.1) |
| `leaves[].signer` | string | the signer of the fragment |
| `leaves[].name` | string | the fragment's document name |
| `leaves[].contentDigest` | 64 lower-case hexadecimal characters | the digest of the fragment's content |
| `leaves[].version` | integer, 0 or more | the fragment version |

**5.1.1** A bundle manifest MUST be valid against the bundle manifest schema.

**5.1.2** The order of `leaves` is part of the manifest: the root MUST be computed over the leaves in the order they are listed.

**5.1.3** The `class`, `name` and `version` of a leaf MUST equal the class, the document name and the version of the fragment it describes.

## 5.2 Signed fragments

A signed fragment is a document (chapter 1) that carries its trust data in labels and annotations of the registry.

| key | kind | meaning |
|---|---|---|
| `sbob/bundle` | label | the bundle the fragment belongs to |
| `sbob/fragment-class` | label | the fragment class |
| `sbob/version` | label | the monotonic version of the fragment |
| `sbob/generator` | label | the tool that produced the document |
| `sbob/generator-ref` | label | the revision of that tool |
| `sbob/matcher-version` | label | the grammar version the document was written against |
| `sbob/bundle-manifest` | annotation | the bundle manifest the fragment commits to |
| `sbob/content` | annotation | the signed content of the fragment |
| `sbob/signature` | annotation | the signature |
| `sbob/certificate` | annotation | the signer's certificate |
| `sbob/identity` | annotation | the signer identity |
| `sbob/issuer` | annotation | the issuer of the signer identity |
| `sbob/timestamp` | annotation | the signing time |
| `sbob/rekor-bundle` | annotation | the transparency log bundle of the signature |

**5.2.1** A producer of signed fragments MUST write the keys of the table above under their current names, and a verifier SHOULD also read the legacy keys the registry pairs with them.

**5.2.2** The version of a fragment MUST only grow: a later fragment of the same bundle, class and name carries a higher version.

## 5.3 Verdicts

A verifier reports the trust state of the document that governs a subject as one verdict.

| verdict | meaning |
|---|---|
| `verified` | the document carries a signature, and the signature verifies for its content under the policy |
| `unsigned` | the document carries no signature |
| `tampered` | the document carries a signature that does not verify for its content |
| `unverified` | verification could not be completed |

**5.3.1** A verifier MUST report exactly one of the four verdicts for a governing document.

**5.3.2** A verifier MUST report a `tampered` verdict with the rule `signed-profile-tampered` (6.5).

**5.3.3** A verifier that reports that an unsigned authored document governs a subject MUST report it with the rule `unsigned-profile-governs` (6.5).

## 5.4 Trust policy

A trust policy tells a verifier what to accept. Its inputs are:

| input | meaning |
|---|---|
| mode | how strictly the verifier acts on a verdict |
| classes | per fragment class: the signers accepted for it and the document paths (spec sections) a fragment of that class may set |
| rule classes | which signers may supply rule sets |
| binding classes | which signers may supply bindings |
| policy version | the version of the policy |

**5.4.1** A verifier MUST NOT accept a fragment signed by a signer that the policy does not accept for the fragment's class.

**5.4.2** A verifier MUST NOT accept from a fragment a document path that the policy does not allow for the fragment's class.

## Conformance

This chapter is a draft. A verifier can claim conformance to 5.1 and 5.3 only.

## Open points

- The leaf encoding and the hash function of the Merkle tree of 5.1.2.
- The digest algorithm of `contentDigest` and the exact bytes it covers.
- The signature scheme, the form of `sbob/content`, and how the certificate, identity, issuer, timestamp and transparency log bundle are checked.
- The values of the policy mode and what each mode does with each verdict.
- The format of the policy and of its classes, rule classes and binding classes.
- The reload of a policy: the earlier skeleton of this chapter described it as a one-way door; what may and may not change on a reload is not specified.
- The replacement of a fragment by a newer version of itself (5.2.2) and what a verifier does with an older one.
- How `unverified` differs from `tampered` when the verifier cannot reach a resource it needs.

## Changes from v0.0.4

v0.0.4 §8 was marked work in progress and carried no text. Everything in this chapter is new.
