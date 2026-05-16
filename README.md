# aidatacollect-updates

Public, signed update catalog for the
[AI Data Collect](https://github.com/bbradford1) platform. Customer
deployments fetch from this repo on an hourly tick to learn which
releases are available.

## What lives here

```
releases.json                              ← top-level catalog
releases.json.sig                          ← Ed25519 signature
builds/<version>/
    packages.json                          ← per-release image manifest
    packages.json.sig
    release-<version>.adcupdate            ← signed offline install bundle
    SHA256SUMS
builds/<version>-hotfix-<32hex>/...        ← password-gated specific builds
docs/                                      ← reserved for the GitHub Pages
                                             changelog site (Slice "reserved
                                             for later" — not wired yet)
```

Every JSON file is paired with a `.sig` whose bytes are the
base64-encoded raw 64-byte Ed25519 signature over the EXACT bytes of
its sibling.

## Trust roots

Public keys whose private counterparts may sign content here:

| key_id | key_version | fingerprint |
|---|---:|---|
| `updates-vendor-prod-2026` | 10 | `SHA256:uBYmtVw6u/WxCJ+D1VeNuS8Xf/6SlHKfZNJtKtjvXLU` |
| `updates-vendor-dev-2026`  | 11 | `SHA256:ccQP5UxQreIOpQTy6+Ti1F51VEAZHrMfVSaFVW0Zvro` |

The matching `.pub` files ship inside every released API image at
`api/services/updates/trusted_keys/`. Customer-side verification
dispatches on `payload.key_version`, NOT on transport metadata.

## How customers consume this

```
GET https://raw.githubusercontent.com/bbradford1/aidatacollect-updates/main/releases.json
GET https://raw.githubusercontent.com/bbradford1/aidatacollect-updates/main/releases.json.sig
```

then, for the selected release tag:

```
GET .../main/builds/<tag>/packages.json
GET .../main/builds/<tag>/packages.json.sig
```

Selection logic (beta channel toggle, compatibility window, deterministic
staged-rollout day offset per `system_id`) lives in the customer-side
`api/services/updates/selector.py`.

## How the catalog gets published

The `aidatacollect-deploy` repo's GitHub Actions workflow
`publish-update-catalog.yml` runs on `release.published`, resolves
digests for the 5 app images + 6 infra images, signs the manifest, and
pushes a commit to `main` here. There is no human write path — the
deploy repo holds the only deploy key for this repo.

## What is NOT here

- Private keys, of any kind. They live age-encrypted in the private
  `bbradford1/aidatacollect-vendor-keys` repo and only ever exist
  decrypted inside the publish workflow's runner.
- The `(version → password)` mapping for specific-build hotfixes. That
  lives in the private vendor-keys repo too.
- Telemetry or per-customer state. The catalog is the same blob for
  every reader; per-customer behavior (beta toggle, license-state gating,
  rollout day offset) is entirely customer-side.

## License

The catalog metadata and signatures in this repo are public artifacts.
The software they describe is licensed separately — see the licensing
state machine in the customer-side API.
