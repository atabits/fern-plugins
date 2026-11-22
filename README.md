# Fern Plugin Registry

Official plugin registry for [Fern](https://github.com/atabits/fern).

- **Live Registry Index (`schema_version: 1`):** <https://atabits.github.io/fern-plugins/registry.json>
- **Registry Status Page:** <https://atabits.github.io/fern-plugins/>

Fern's **Settings → Plugins → Browse** tab reads `registry.json` directly from GitHub Pages with `ETag` (`If-None-Match`) caching and verifies every downloaded plugin archive against its SHA-256 checksum.

---

## Submitting or Updating a Plugin

1. Package your plugin as a `.zip` archive containing `manifest.json` (`api_version: 1`), your entry JavaScript bundle (for example `index.js` or `dist/index.js`), and any local SVG icons or assets.
2. Publish the `.zip` archive on a public `https://` release (such as a GitHub Release in your plugin repository).
3. Fork this repository and add or update a descriptor at `plugins/<your-plugin-id>.json` matching [`schema/plugin-descriptor.schema.json`](schema/plugin-descriptor.schema.json):

```json
{
  "id": "com.example.my-plugin",
  "version": "1.0.0",
  "source_url": "https://github.com/example/fern-my-plugin/releases/download/v1.0.0/plugin.zip",
  "sha256": "<optional-64-char-lowercase-hex>",
  "homepage": "https://github.com/example/fern-my-plugin",
  "min_fern_version": "0.2.0",
  "tags": ["productivity", "sync"]
}
```

### Descriptor Fields

| Field | Required | Description |
| --- | --- | --- |
| `id` | Yes | Plugin ID matching `^[a-z0-9][a-z0-9._-]{1,63}$`, no `..`, identical to filename (`plugins/<id>.json`) and `manifest.json`. |
| `version` | Yes | Non-empty release version string matching `manifest.json` (`version`). |
| `source_url` | Yes | Public `https://` URL to the `.zip` archive. |
| `sha256` | No | Optional 64-character lowercase hex SHA-256 checksum verified during PR checks and mirroring. |
| `homepage` | No | Optional `https://` URL to the plugin homepage or repository (falls back to `manifest.json` `homepage`). |
| `min_fern_version` | No | Optional minimum compatible Fern version (falls back to `manifest.json` `min_fern_version`). |
| `tags` | No | Optional array of unique non-empty tags for catalog discovery. |

---

## Automated Validation & Publishing Pipeline

### On Pull Requests (`validate-pr`)
- Validates every `plugins/<id>.json` descriptor against the ID, version, `source_url`, `homepage`, and `tags` constraints.
- Resolves DNS and blocks non-public / private / loopback / link-local IPs across initial requests and HTTP redirects (SSRF protection).
- Downloads each changed `.zip` and enforces:
  - Compressed size `<= 10 MiB` and unpacked size `<= 32 MiB`
  - No symlinks, unsafe modes, Windows paths, or zip-slip (`..`) path traversals
  - Optional submitter-provided `sha256` match
  - Valid `manifest.json` (`api_version: 1`, matching `id` and `version`, existing `entry`, known permissions / valid `network` host patterns, valid sidebar SVG icons `<= 256 KiB`, valid `settings` specs).

### On Merge to `main` (`publish-registry`)
- Downloads verified archives once and mirrors each `<id>@<version>` archive as an immutable `plugin.zip` asset under GitHub Release tag `<id>-<version>`.
- Generates `registry.json` (`schema_version: 1`) pointing `download_url` at the mirrored GitHub Release asset with computed `sha256` and `size_bytes`.
- Deploys `registry.json` and `index.html` to GitHub Pages.

---

## License

The registry infrastructure, workflow, and documentation in this repository are licensed under the [MIT License](LICENSE). Individual plugins listed in the registry remain licensed under the terms specified by their respective authors.
