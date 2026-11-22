## Plugin Submission Summary

<!-- Describe what your plugin does or what changed in this version, including any newly requested permissions. -->

- **Plugin ID:** ``
- **Version:** ``
- **Repository / Source:** 

## Checklist

- [ ] Added or updated descriptor at `plugins/<plugin-id>.json` (filename matches both `id` and `manifest.json`)
- [ ] Followed [`schema/plugin-descriptor.schema.json`](../schema/plugin-descriptor.schema.json) (including required lowercase 64-char `sha256`)
- [ ] `source_url` points to a public `https://` `.zip` release asset (`<= 10 MiB` compressed, `<= 32 MiB` unpacked, JS files `<= 2 MiB`, no symlinks)
- [ ] `manifest.json` targets `api_version: 1` and requests only the minimal required permissions
- [ ] Tested locally in Fern via **Settings → Plugins → Install from ZIP…**
- [ ] (If updating) Bumped `version` and summarized purpose and any new permissions above
