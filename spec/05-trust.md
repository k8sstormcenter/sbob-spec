# 5. Trust

Status: skeleton. v0.0.4 §8 is marked WIP and carries no text.

## Scope

The bundle manifest (a Merkle root over leaves, each leaf a class, signer, name, content digest and version), signer identity, the trust policy (mode, classes with signers and allowed spec paths, rule classes, binding classes, policy version), verdicts verified, unsigned, tampered and unverified, and the one-way door of policy reload.

## Pinned by

`schema/bundle-manifest.schema.json`, vector set `trust`, `truth/SB` rows.
