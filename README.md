# cyberdeck-release

Update manifests for **Neon Nexus**. Served over GitHub Pages at
**https://cyberdeck.chapmanmedia.com/**, which is the address compiled into
every shipped binary.

This repository holds *manifests only*. The payloads — the `.app`, the `.exe`,
the tarballs, the soundtrack — are **Releases assets** on this same repository.
They are deliberately not committed: git keeps every version of every file
forever, and the soundtrack alone is ~37MB per release, which would turn a
clone into a download.

## Layout

| Path | What |
|---|---|
| `latest.json` | The update descriptor the app fetches. One file, always this name. |
| `CNAME` | The Pages custom domain. Do not delete — removing it breaks the endpoint for every installed copy. |
| `.nojekyll` | Stops Pages running Jekyll over the files. |

## The descriptor

`latest.json` is an envelope:

```json
{
  "manifest": "<the manifest, as raw JSON text>",
  "sig": "<base64 Ed25519 signature over exactly those bytes>"
}
```

The signature covers the manifest **string as transmitted**, not a re-serialised
object — so there is no canonicalisation rule for the signer and the verifier to
disagree about. The client verifies before it parses, and refuses anything that
does not verify against a public key compiled into the binary.

The manifest itself:

```json
{
  "version": "0.2.0",
  "commit": "abc1234",
  "files":  { "gui/assets/music/manifest.json": "https://github.com/matt2000/cyberdeck-release/releases/download/v0.2.0/manifest.json" },
  "sha512": { "gui/assets/music/manifest.json": "..." },
  "bundle": { "url": "https://github.com/matt2000/cyberdeck-release/releases/download/v0.2.0/NeonNexus.app.zip", "sha512": "..." }
}
```

- `files` — written relative to the app. Read by every platform.
- `bundle` — a whole replacement `NeonNexus.app`. Read by macOS only, and
  published **only when the binary actually changed**; an asset-only release
  omits it and costs kilobytes instead of the whole application.

## The current state

`latest.json` currently carries an empty manifest. That is the well-defined
"nothing to update" state: the client refuses an empty manifest and changes
nothing. It is here so the endpoint can be tested before the first real
release exists.

## Publishing a release

See `docs/releasing.md` in the main repository. In short: build, sign with
`tools/sign_manifest.nim`, upload the payloads as Releases assets, then commit
the new `latest.json` here.

## Security

The host is not trusted. Integrity comes from the signature, which is why this
repository can be public and served from a CDN without that being a risk. What
*must* stay secret is the signing key — it never enters this repository, the
main repository, or CI.

Two public keys are compiled into the client: a primary, and a rollover used
only to recover from the primary leaking. Both must exist before the first
public release; neither can be added to binaries that have already shipped.
