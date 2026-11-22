# Fern Plugin Registry

Official plugin registry for [Fern](https://github.com/atabits/fern).

- **Live Registry Index (`schema_version: 1`):** <https://atabits.github.io/fern-plugins/registry.json>
- **Descriptor JSON Schema:** <https://atabits.github.io/fern-plugins/plugin-descriptor.schema.json>
- **Registry Status Page:** <https://atabits.github.io/fern-plugins/>

Fern's **Settings → Plugins → Browse** tab reads `registry.json` directly from GitHub Pages with `ETag` (`If-None-Match`) caching and verifies every downloaded plugin archive against its SHA-256 checksum.

---

## Submitting or Updating a Plugin

1. Package your plugin as a `.zip` archive containing `manifest.json` (`api_version: 1`), your entry JavaScript bundle (for example `index.js` or `dist/index.js`, `<= 2 MiB` per JS file), and any local SVG icons or assets.
2. Publish the `.zip` archive on a public `https://` release (such as a GitHub Release in your plugin repository) and compute its lowercase SHA-256 checksum.
3. Fork this repository and add or update a descriptor at `plugins/<your-plugin-id>.json` matching [`schema/plugin-descriptor.schema.json`](schema/plugin-descriptor.schema.json):

```json
{
  "id": "com.example.my-plugin",
  "version": "1.0.0",
  "source_url": "https://github.com/example/fern-my-plugin/releases/download/v1.0.0/plugin.zip",
  "sha256": "<required-64-char-lowercase-hex>",
  "homepage": "https://github.com/example/fern-my-plugin",
  "min_fern_version": "0.2.0",
  "tags": ["productivity", "sync"]
}
```

### Descriptor Fields

| Field | Required | Description |
| --- | --- | --- |
| `id` | Yes | Plugin ID matching `^[a-z0-9][a-z0-9._-]{1,63}$`, no `..`, identical to filename (`plugins/<id>.json`) and `manifest.json`. |
| `version` | Yes | Release version matching `^[0-9A-Za-z][0-9A-Za-z._+-]{0,63}$` without `..`, identical to `manifest.json` (`version`). Always bump `version` when publishing a new archive. |
| `source_url` | Yes | Public `https://` URL to the `.zip` archive. |
| `sha256` | Yes | Required 64-character lowercase hex SHA-256 checksum verified during PR checks and mirroring (prevents PR-to-merge archive swaps). |
| `homepage` | No | Optional `https://` URL to the plugin homepage or repository (falls back to `manifest.json` `homepage`). |
| `min_fern_version` | No | Optional minimum compatible Fern version (falls back to `manifest.json` `min_fern_version`). |
| `tags` | No | Optional array of unique non-empty tags for catalog discovery. |

---

## Automated Validation & Publishing Pipeline

### On Pull Requests (`validate-pr`)
- Validates every `plugins/<id>.json` descriptor against the schema (no unknown fields, no `null` values, valid `id`/`version`, `https://` URLs, required `sha256`, unique non-empty `tags`) and rejects stray or nested files under `plugins/`.
- Resolves DNS once per hop, blocks non-public / private / loopback / link-local / IPv6-transition IPs across initial requests and HTTPS redirects, and connects to the pinned resolved IP (DNS-rebinding SSRF protection).
- Rejects reused `<id>-<version>` release tags whose existing mirrored archive has a different SHA-256.
- Downloads each changed `.zip` and enforces:
  - Compressed size `<= 10 MiB`, unpacked size `<= 32 MiB`, `<= 4096` archive entries, and `<= 2 MiB` per `.js`/`.mjs` module
  - No symlinks, unsafe modes, Windows paths, or zip-slip (`..`) path traversals
  - Exact match against the descriptor's required `sha256`
  - Valid `manifest.json` (`api_version: 1`, matching `id` and `version`, non-null string metadata, valid `https://` homepage, existing `entry` `<= 2 MiB`, known permissions / valid `network` host patterns, unique `contributions.sidebar` + `contributions.pages` view IDs with valid sanitized SVG icons `<= 256 KiB`, valid `commands`, `search_providers`, `themes`, `locales`, `importers`, and `settings` labels/types/options).

### On Merge to `main` (`publish-registry`)
- Re-validates all descriptors, verifies `sha256` on both newly mirrored and already-mirrored releases, and mirrors each new `<id>@<version>` archive as an immutable `plugin.zip` asset under GitHub Release tag `<id>-<version>`.
- Generates `registry.json` (`schema_version: 1`) with non-null strings/arrays and URL-encoded `download_url` pointing at the mirrored GitHub Release asset with verified `sha256` and `size_bytes`.
- Deploys `registry.json`, `index.html`, and `plugin-descriptor.schema.json` to GitHub Pages.

---

## License

The registry infrastructure, workflow, and documentation in this repository are licensed under the [MIT License](LICENSE). Individual plugins listed in the registry remain licensed under the terms specified by their respective authors.
